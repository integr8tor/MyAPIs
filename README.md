# LBF_DEMO_OrderAPI - Demo Order API

## Overview

This project contains the **Demo Order API** for IBM API Connect v12 with DataPower v6 gateway support. The API allows users to view and track their shipping orders and deliveries with **Client ID and Client Secret** authentication enforced at the Product/Plan level. The API demonstrates advanced policy capabilities including data transformation and redaction.

## Assets Included

This sample contains the following assets:

### API Definitions

- **OrderAPI.yaml** - Main API asset (DemoOrderAPI) with kind: API format
- **OrderAPI-spec.yaml** - OpenAPI 3.0.3 specification with full path definition and security schemes defined (but not enforced at API level)

### Policies

- **OrderAPI-Assembly.yaml** - DataPower Assembly policy with a complete policy flow:
  - **Invoke Policy** (v2.0.0) - Calls the backend order service
  - **Parse Policy** (v2.1.0) - Parses JSON response from backend
  - **Map Policy** (v2.1.0) - Transforms data fields from snake_case to camelCase
  - **Redact Policy** (v2.0.0) - Redacts sensitive tracking number information
  - Compatibility settings for IBM API Connect v12

### CORS Configuration

- **OrderAPI-Cors.yaml** - CORS configuration allowing GET requests with both X-IBM-Client-Id and X-IBM-Client-Secret headers

### Products & Plans

- **OrderAPI-Product.yaml** - Product asset (DemoOrderAPI_Product) for DataPower APIGateway
- **OrderAPI-Plan.yaml** - Plan asset (DemoOrderAPI_Plan) with quota reference and gateway extensions
- **OrderAPI-Quota.yaml** - Rate limiting quota (100 requests per minute)

## IBM API Connect v12 Compatibility

This API has been updated for IBM API Connect v12 with the following changes:

### Key Updates for v12

1. **Map Policy Version**: Uses map policy version 2.1.0 which is compatible with OpenAPI 3.0.3 APIs in API Connect v12.

2. **Compatibility Settings**: Added `suppress-limitation-errors: true` in the x-ibm-configuration to suppress OpenAPI 3.0.3 limitation warnings.

3. **Security Architecture**: Security is enforced at the Product/Plan level rather than in the OpenAPI spec, allowing for easier testing in API Manager without requiring subscriptions.

4. **Complete Policy Flow**: Demonstrates a full assembly with invoke, parse, map, and redact policies for a comprehensive API management example.

## Policy Flow Details

### 1. Invoke Policy
- Calls the backend order service
- Constructs URL with order number parameter
- Stores response in `backendResponse` variable

### 2. Parse Policy
- Parses the JSON response from backend
- Makes data accessible for transformation
- Outputs to `parsedOrder` variable

### 3. Map Policy
- Transforms backend field names to API-friendly names:
  - `order_number` → `orderNumber`
  - `created_at` → `orderDate`
  - `shipped_at` → `shippedDate`
  - `status` → `orderStatus`
  - `delivery_method` → `shippingMethod`
  - `tracking_reference` → `trackingNumber`
  - `tracking_status` → `trackingStatus`

### 4. Redact Policy
- Redacts sensitive tracking number information
- Replaces actual tracking number with `***REDACTED***`
- Demonstrates data privacy and security best practices

## Getting Started

1. Import all YAML files from this folder into IBM API Connect v12 in this order:
   - OrderAPI-Quota.yaml
   - OrderAPI-Cors.yaml
   - OrderAPI-Assembly.yaml
   - OrderAPI-spec.yaml
   - OrderAPI.yaml
   - OrderAPI-Plan.yaml
   - OrderAPI-Product.yaml
2. Publish to a catalog containing DataPower APIGateway
3. **For Testing in API Manager**:
   - The API can be tested directly in API Manager without requiring subscriptions
   - Security schemes are defined but not enforced at the API level
   - This allows immediate testing and validation
4. **For Production Use**:
   - Create a Consumer organization
   - Create an Application
   - Subscribe the Application to the API (this generates both Client ID and Client Secret)
   - Security is enforced through the Product/Plan configuration
5. Pass both authentication headers when calling from external clients:
   - X-IBM-Client-Id: your-client-id
   - X-IBM-Client-Secret: your-client-secret

## API Endpoint

- **API Name**: DemoOrderAPI
- **Path**: `/order/{orderNumber}`
- **Method**: GET
- **Description**: Retrieve order details by order number
- **Security**: Client ID + Client Secret (enforced at Product/Plan level)

## Example Request

```
GET /order/1234
Headers:
  X-IBM-Client-Id: your-client-id
  X-IBM-Client-Secret: your-client-secret
```

## Response Schema

The API returns transformed and redacted order data:

```json
{
  "orderNumber": "ORD00989792",
  "orderDate": "2024-01-01T10:00:00Z",
  "shippedDate": "2024-01-02T14:30:00Z",
  "orderStatus": "SHIPPED",
  "shippingMethod": "Express",
  "trackingNumber": "***REDACTED***",
  "trackingStatus": {
    // Tracking details from shipping provider
  }
}
```

**Note**: The `trackingNumber` field is redacted for privacy and security purposes.

## Backend Service

- **Order System**: https://sample-api.us-east-a.apiconnect.automation.ibm.com/orders/order/{orderNumber}

The backend returns data in snake_case format which is transformed to camelCase by the map policy.

## Security Features

- **Dual Authentication**: Requires both Client ID and Client Secret (enforced at Product/Plan level)
- **Test-Friendly**: API can be tested in API Manager without subscriptions
- **Production-Ready**: Security is enforced for actual API consumers through Product/Plan configuration
- **Data Redaction**: Sensitive tracking information is automatically redacted in responses
- **CORS Support**: Both authentication headers are allowed in CORS configuration

## Testing

### In API Manager (No Subscription Required)
1. Open the API in API Manager
2. Use the built-in test tool
3. Enter an order number (e.g., "1234")
4. Click "Invoke" - no authentication required for testing
5. Observe the transformed and redacted response

### From External Clients (Subscription Required)
1. Create an application and subscribe to the DemoOrderAPI_Product
2. Use the generated Client ID and Client Secret
3. Include both headers in your API calls
4. The response will include redacted tracking information

## Troubleshooting

### Publishing Errors
- **"Map policy version not supported"**: Resolved by using map policy version 2.1.0 which is compatible with OpenAPI 3.0.3
- **"Limitation errors"**: Resolved by adding `suppress-limitation-errors: true` in the compatibility configuration

### Runtime Errors
- **401 Unauthorized**: If testing from external clients, ensure you have a valid subscription and are passing both Client ID and Client Secret headers
- **500 Internal Server Error**: Check that the backend service is accessible and the order number parameter is being passed correctly

## Demonstration Features

This API demonstrates the following IBM API Connect v12 capabilities:

1. **Backend Integration** - Invoke policy calling external REST service
2. **Data Parsing** - Parse policy for JSON response handling
3. **Data Transformation** - Map policy for field name transformation
4. **Data Security** - Redact policy for sensitive information masking
5. **API Security** - Client ID/Secret authentication at Product level
6. **Rate Limiting** - Quota policy for request throttling
7. **CORS Support** - Cross-origin resource sharing configuration
8. **Test Mode** - Easy testing without subscription requirements

## Notes

- This is a **Demo API** for demonstration purposes
- Rate limiting is set to 100 requests per minute per subscription
- Namespace: `LBF_DEMO_OrderAPI`
- All policy references must use this namespace
- Version: 1.0
- **Compatible with IBM API Connect v12 and DataPower Gateway v6**
- Demonstrates best practices for data transformation and privacy

## Version History

### v1.0 (Current)
- Updated for IBM API Connect v12 compatibility
- Implemented map policy v2.1.0 for data transformation (compatible with OpenAPI 3.0.3)
- Added parse policy for JSON response handling
- Added redact policy for tracking number privacy
- Added compatibility settings for v12
- Security enforcement moved to Product/Plan level for easier testing
- Optimized for both testing and production use
- Complete demonstration of invoke, parse, map, and redact policies

# Made with Bob