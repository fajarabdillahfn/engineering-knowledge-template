# Flow: InquiryV2

The end-to-end process of receiving an inquiry request from `api-gateway`, resolving the vendor, calling the vendor, and returning the bill details to the caller.

## Trigger

`api-gateway` issues a gRPC `InquiryV2` call to this service. The request includes a product code, an account number, an amount (for prepaid), a month (for BPJS), a partner ID, a client ID, and an optional `DirectConnection` flag.

## Preconditions

- The caller (api-gateway) has authenticated the user.
- The product code is registered in the routing table (`unified_inquiry_info` view or equivalent).
- The product is active.
- The product has at least one switch, and the switch has a vendor.
- The vendor is active.

## Steps

1. `inquiryHandler.InquiryV2` receives the gRPC request. It validates that the request is not nil, then parses `ClientID` (uint16) and `DirectConnection` (bool). Parse failures return a generic failure response without calling the service.
2. The handler builds the internal `models.InquiryRequest` from the gRPC request and sets the context to carry `directConnection` and a metadata header `v2.0` (used by downstream services).
3. The handler calls `inquiryService.Inquiry(ctx, request)`.
4. `inquiryService.Inquiry` calls `inquiryRegistry.GetUnifiedInquiryInfo(ProductCode)` to resolve the product, switch, and vendor in a single database round-trip. A `sql.ErrNoRows` returns `models.InquiryProductNotFound`. Any other error returns `models.InquiryFail`.
5. The service checks the product status. If inactive, returns `models.InquiryInactiveProduct`.
6. The service checks that the product has a category. If the category ID is 0, returns `models.InquiryCategoryNotPresent`.
7. The service checks the switch. If the switch ID is 0, returns `models.InquirySwitchNotPresent`.
8. The service checks the vendor status. If inactive, returns `models.InquiryVendorInactive`.
9. The service sets `REQUEST_VENDOR_CODE_KEY` and `REQUEST_CATEGORY_CODE_KEY` on the context.
10. The service applies product-specific rules via `validateProductSpecificRules`:
    - **`CTAYBP` (BPJS Ketenagakerjaan)**: requires `Month` between 1 and 12.
    - **`CTAYLS` (Listrik) with `VENDOR_JL` or `VENDOR_JLB`**: postpaid requires exactly 12 digits; prepaid requires 11-12 digits.
11. If the vendor is KAI, the service checks KAI enrollment. In non-production environments, specific test account numbers bypass this check.
12. The service dispatches based on the product type:
    - **`PREPAID`**: prepends "0" to the account number for certain category codes (`CTAYPD`, `CTAYPL`, `CTAYEM`, `CTAYTK`) — see `decisions/ADR-0003-account-number-prefix.md`. Calls `prepaidService.CreatePrepaidInquiry`.
    - **`POSTPAID`**: calls `postpaidService.CreatePostpaidInquiry`.
    - **`VOUCHER`**: not implemented. Returns an error.
13. The domain service (`prepaid` or `postpaid`) selects the vendor adapter (e.g., `jkpln`) and calls its `DoInquiry` method.
14. The vendor adapter builds a vendor-specific gRPC request and calls the vendor's gRPC service. The gRPC client applies the shared retry policy (3 retries on `UNAVAILABLE`, exponential backoff).
15. The vendor adapter parses the vendor's gRPC response into the internal `models.InquiryResponse`.
16. The handler marshals the response to JSON, runs it through `inquiryFormatter.Response` to produce the OpenAPI-format response, and maps it back to the gRPC proto.
17. If the result has a non-empty `RequestID`, the handler fires a background goroutine to write the OpenAPI response to the inquiry log. The goroutine uses a `ValueOnlyContext` to preserve trace IDs but ignore the request's cancellation and deadline.
18. The handler returns the gRPC response to `api-gateway`.

## Outcome

On success, the client receives a `InquiryV2Response` with `Success=true`, the customer's bill details, the amount, the validity, and a `RequestID`. On failure, the client receives `Success=false` with a specific `ResponseCode` and a `Message` in English and Indonesian.

## Failure Handling

- **Nil request or malformed input**: returns a generic failure response with `ResponseCode=models.InquiryFail`.
- **Product not found**: `ResponseCode=models.InquiryProductNotFound`.
- **Product inactive**: `ResponseCode=models.InquiryInactiveProduct`.
- **Category not mapped**: `ResponseCode=models.InquiryCategoryNotPresent`.
- **Switch not present**: `ResponseCode=models.InquirySwitchNotPresent`.
- **Vendor inactive**: `ResponseCode=models.InquiryVendorInactive`.
- **Product-specific validation failure (BPJS month, PLN IDPEL length)**: returns the specific response code from the validation error.
- **KAI enrollment check failure**: `ResponseCode=models.InquiryFail`.
- **VOUCHER product type**: returns an error indicating the type is not implemented.
- **Vendor gRPC returns `UNAVAILABLE`**: retried up to 3 times by the gRPC client. If still unavailable, `ResponseCode=models.InquiryFail`.
- **Vendor gRPC returns a non-`UNAVAILABLE` error**: not retried. The error is propagated; the specific response code is vendor-defined.
- **Vendor returns a structured error response**: not retried. The vendor's response code is mapped to a controlroom response code by the vendor adapter.
- **Context cancellation**: logged as a warning, treated as a failure with the response code from the partial result if available, otherwise `models.InquiryFail`.
- **Panic in the background update goroutine**: recovered and logged. Does not affect the response to the caller.

## Relationships

- `references` `../services/controlroom-inquiry.md` (the service that executes the flow).
- `references` `../grpc/action.proto` (the gRPC contract that defines the inbound request and response).
- `references` `../decisions/ADR-0001-grpc-for-vendor-comms.md` (the choice of gRPC for the vendor call).
- `references` `../decisions/ADR-0002-per-vendor-interface.md` (the choice of per-vendor interfaces).
- `references` `../decisions/ADR-0003-account-number-prefix.md` (the account number normalization rule).
- `references` `../common/grpc_client.go` (the gRPC retry policy).
- `relates_to` `../troubleshooting/vendor-unavailable.md` (the most common operational failure mode of this flow).
- `relates_to` the upstream `api-gateway` service and the parallel `controlroom-payment` service.
