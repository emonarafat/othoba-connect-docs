# 🚀 API Azan Wholesale - Product & Stock Synchronization

> **Production-Ready Technical Reference for EuroAsia Wholesale Integration**

<div style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); padding: 20px; border-radius: 8px; color: white; margin-bottom: 30px;">

**📌 Version:** 2.1 | **📅 Released:** February 2026 | **Updated:** January 2024 | **Status:** ✅ Production Ready

```
🔗 API Endpoint:  https://connect.othoba.com/api/azan-wholesale
🔒 Protocol:      REST (HTTPS/TLS 1.2+)
📦 Format:        JSON (application/json)
🔑 Auth:          API Key (X-API-Key or Secret-Key Header)
⚡ Framework:     ASP.NET Core 6 (.NET 6)
🎯 Response:      Typed ApiResponse + RFC 7807 ProblemDetails
🔧 Status Codes:  200 (Success), 400 (Validation), 401 (Auth), 500 (Error)
```

</div>

## 📋 Quick Overview

The **API for Azan Wholesale** provides a modern, secure REST interface for real-time synchronization of:

- ✅ **Product Information** - Names, pricing, supplier details
- ✅ **Inventory Levels** - Stock quantities, availability
- ✅ **Error Reporting** - Detailed validation feedback with structured error responses

### Key Highlights

| 🎯 Feature | 📝 Details |
|-----------|-----------|
| **Performance** | 100-200ms average response time |
| **Scalability** | Stateless, distributed architecture |
| **Security** | API Key + HTTPS/TLS 1.2+ |
| **Validation** | Comprehensive input validation |
| **Reliability** | Exponential backoff support |
| **Rate Limit** | 1,000 requests/hour |
| **Response Format** | Strongly-typed responses with ProblemDetails |

---

## 📚 Table of Contents

- [🔐 Authentication & Security](#-authentication--security)
- [📦 API Models & Schemas](#-api-models--schemas)
- [📤 Response Formats](#-response-formats)
- [🔌 Endpoints Reference](#-endpoints-reference)
- [⚙️ HTTP Status Codes](#-http-status-codes)
- [💻 Implementation Examples](#-implementation-examples)
- [⚠️ Error Handling](#-error-handling)
- [📊 Performance & Limits](#-performance--limits)

---

## 🔐 Authentication & Security

### 🔑 API Key Authentication

> All requests require authentication via API Key transmitted in the HTTP request header. **Supports both `X-API-Key` and `Secret-Key` headers.**

```
X-API-Key: {api-key}
```

or

```
Secret-Key: {api-key}
```

**Authentication Specifications:**

| 🎯 Requirement | 📋 Specification |
|---|---|
| 🔒 **Protocol** | **HTTPS/TLS 1.2+** (mandatory) |
| 📝 **Header Names** | `X-API-Key` **or** `Secret-Key` (case-insensitive) |
| 🔑 **Key Format** | Alphanumeric string (provided by Othoba) |
| 📤 **Transmission** | HTTP Header only (never in URL/body) |
| ✅ **Validation** | Performed on every single request |
| 🔄 **Fallback** | Checks `X-API-Key` first, then `Secret-Key` if not found |

### 🛡️ Security Best Practices

| 🔐 Practice | 💡 Implementation |
|---|---|
| 🔒 **Storage** | Use secure environment variables (never hardcode) |
| 📋 **Logging** | Never log complete keys; mask all but last 4 chars |
| 🔄 **Rotation** | Rotate API keys quarterly minimum |
| 🚫 **Access Control** | Restrict to specific IP ranges when possible |
| 👁️ **Monitoring** | Alert on unusual patterns, repeated failures |
| 🚨 **Revocation** | Immediately revoke compromised keys |

---

## 📦 API Models & Schemas

### 📱 ProductUpdateRequest

> **Purpose:** Synchronize product details to the Othoba platform in real-time.

**JSON Schema:**

```json
{
  "type": "object",
  "properties": {
    "product_id": { "type": "string", "description": "Unique product identifier" },
    "supplier": { "type": "string", "description": "Supplier/vendor name" },
    "name": { "type": "string", "description": "Product name/title" },
    "sku": { "type": "string", "description": "Stock Keeping Unit" },
    "mrp_price": { "type": "number", "description": "Maximum Retail Price" },
    "wholesale_price": { "type": "number", "description": "Wholesale price" }
  },
  "required": ["product_id", "supplier", "name", "sku", "mrp_price", "wholesale_price"]
}
```

**Field Reference:**

| 🏷️ Field | 📝 Type | ✅ Required | 📏 Constraints | 📖 Description |
|---|---|---|---|---|
| `product_id` | String | ✓ | Non-empty, alphanumeric | Unique product identifier in Othoba |
| `supplier` | String | ✓ | 1-500 chars | Supplier/vendor name |
| `name` | String | ✓ | 1-1000 chars | Product title/description |
| `sku` | String | ✓ | 1-500 chars, alphanumeric | Stock Keeping Unit code |
| `mrp_price` | Decimal | ✓ | ≥ 0, max 2 decimals | Maximum Retail Price |
| `wholesale_price` | Decimal | ✓ | ≥ 0, max 2 decimals | Wholesale/cost price |

**Example Payload:**

```json
{
  "product_id": "123",
  "supplier": "Azan Wholesale",
  "name": "Samsung 65-Inch 4K Smart TV",
  "sku": "TV-SAMSUNG-65-2024",
  "mrp_price": 89999.99,
  "wholesale_price": 75000.00
}
```

---

### 📦 StockUpdateRequest

> **Purpose:** Update inventory/stock levels for existing products in real-time.

**JSON Schema:**

```json
{
  "type": "object",
  "properties": {
    "product_id": { "type": "string" },
    "sku": { "type": "string" },
    "supplier": { "type": "string" },
    "stock": { "type": "integer" }
  },
  "required": ["product_id", "sku", "supplier", "stock"]
}
```

**Field Reference:**

| 🏷️ Field | 📝 Type | ✅ Required | 📏 Constraints | 📖 Description |
|---|---|---|---|---|
| `product_id` | String | ✓ | Non-empty, alphanumeric | Product identifier (must exist) |
| `sku` | String | ✓ | 1-500 chars, alphanumeric | Stock Keeping Unit |
| `supplier` | String | ✓ | 1-500 chars | Supplier identifier |
| `stock` | Integer | ✓ | ≥ 0, ≤ 2,147,483,647 | Available quantity (units) |

**Example Payload:**

```json
{
  "product_id": "123",
  "sku": "TV-SAMSUNG-65-2024",
  "supplier": "Azan Wholesale",
  "stock": 150
}
```

---

## 📤 Response Formats

### ✅ Success Response Format

**HTTP 200 OK - Strongly Typed Response:**

```json
{
  "success": true,
  "message": "Product updated successfully",
  "errors": null
}
```

**Response Structure:**

| 🏷️ Field | 📝 Type | 📖 Description |
|---|---|---|
| `success` | Boolean | Always `true` for successful requests |
| `message` | String | Human-readable success message |
| `errors` | Object \| Null | Always `null` for success responses |

**Success Message Examples:**

| 🎯 Endpoint | 📝 Message |
|---|---|
| POST `/update-product` | "Product updated successfully" |
| POST `/update-stock` | "Stock updated successfully" |

---

### ❌ Validation Error Response Format (400 Bad Request)

**RFC 7807 ProblemDetails with Typed Response:**

```json
{
  "type": "https://connect.othoba.com/api/validation-error",
  "title": "Validation Error",
  "status": 400,
  "detail": "One or more validation errors occurred.",
  "instance": "/api/azan-wholesale/update-product",
  "response": {
    "success": false,
    "message": "One or more validation errors occurred.",
    "errors": {
      "product_id": [
        "product_id is required"
      ],
      "mrp_price": [
        "mrp_price cannot be negative"
      ]
    }
  }
}
```

**Response Structure:**

| 🏷️ Field | 📝 Type | 📖 Description |
|---|---|---|
| `type` | String | Problem type URI |
| `title` | String | Problem title ("Validation Error") |
| `status` | Integer | HTTP status code (400) |
| `detail` | String | Problem description |
| `instance` | String | Request path that caused the error |
| `response` | Object | Nested ApiResponse with error details |
| `response.success` | Boolean | Always `false` for errors |
| `response.message` | String | Error message |
| `response.errors` | Object | Field-level validation errors |

**Field-Level Errors:**

```json
{
  "errors": {
    "field_name": [
      "Error message 1",
      "Error message 2"
    ]
  }
}
```

---

### 🔐 Authentication Error Response Format (401 Unauthorized)

**RFC 7807 ProblemDetails Format:**

```json
{
  "type": "https://connect.othoba.com/api/validation-error",
  "title": "Validation Error",
  "status": 401,
  "detail": "Invalid or missing API key",
  "instance": "/api/azan-wholesale/update-product",
  "response": {
    "success": false,
    "message": "Invalid or missing API key",
    "errors": null
  }
}
```

**Common 401 Scenarios:**

| 🎯 Scenario | 💬 Message |
|---|---|
| Missing header | "Invalid or missing API key" |
| Invalid key | "Invalid or missing API key" |
| Wrong header name | "Invalid or missing API key" |

---

### ⚠️ Operation Error Response Format (400 Bad Request)

**RFC 7807 ProblemDetails for Business Logic Errors:**

```json
{
  "type": "https://connect.othoba.com/api/operation-error",
  "title": "Operation Error",
  "status": 400,
  "detail": "Product with this SKU already exists",
  "instance": "/api/azan-wholesale/update-product",
  "response": {
    "success": false,
    "message": "Product with this SKU already exists",
    "errors": null
  }
}
```

---

### 🔴 Server Error Response Format (500 Internal Server Error)

**RFC 7807 ProblemDetails for Unexpected Errors:**

```json
{
  "type": "https://connect.othoba.com/api/server-error",
  "title": "Internal Server Error",
  "status": 500,
  "detail": "An unexpected error occurred while processing the request",
  "instance": "/api/azan-wholesale/update-product",
  "response": {
    "success": false,
    "message": "An unexpected error occurred while processing the request",
    "errors": null
  }
}
```

---

## 🔌 Endpoints Reference

### 🔄 POST /update-product

> **Update product information** including name, pricing, and supplier details.

**Endpoint Specifications:**

```
🔗 Method:           POST
📂 Path:             /update-product
🌐 Full URL:         https://connect.othoba.com/api/azan-wholesale/update-product
📋 Content-Type:     application/json
🔑 Auth Required:    X-API-Key or Secret-Key header
⏱️ Timeout:          30 seconds
```

**Request Example:**

```
POST /api/azan-wholesale/update-product HTTP/1.1
Host: connect.othoba.com
Content-Type: application/json
X-API-Key: your-api-key-here

{
  "product_id": "123",
  "supplier": "Azan Wholesale",
  "name": "Samsung 65-Inch 4K Smart TV",
  "sku": "TV-SAMSUNG-65-2024",
  "mrp_price": 89999.99,
  "wholesale_price": 75000.00
}
```

**✅ Success Response (200 OK):**

```
HTTP/1.1 200 OK
Date: Wed, 15 Jan 2024 10:30:45 GMT
Server: Kestrel
Content-Type: application/json
```

```json
{
  "success": true,
  "message": "Product updated successfully",
  "errors": null
}
```

**❌ Validation Error (400 Bad Request):**

```
HTTP/1.1 400 Bad Request
Content-Type: application/json
```

```json
{
  "type": "https://connect.othoba.com/api/validation-error",
  "title": "Validation Error",
  "status": 400,
  "detail": "One or more validation errors occurred.",
  "instance": "/api/azan-wholesale/update-product",
  "response": {
    "success": false,
    "message": "One or more validation errors occurred.",
    "errors": {
      "product_id": ["product_id is required"],
      "mrp_price": ["mrp_price cannot be negative"]
    }
  }
}
```

**🔐 Auth Error (401 Unauthorized):**

```
HTTP/1.1 401 Unauthorized
Content-Type: application/json
```

```json
{
  "type": "https://connect.othoba.com/api/validation-error",
  "title": "Validation Error",
  "status": 401,
  "detail": "Invalid or missing API key",
  "instance": "/api/azan-wholesale/update-product",
  "response": {
    "success": false,
    "message": "Invalid or missing API key",
    "errors": null
  }
}
```

**⚠️ Server Error (500 Internal Server Error):**

```
HTTP/1.1 500 Internal Server Error
Content-Type: application/json
```

```json
{
  "type": "https://connect.othoba.com/api/server-error",
  "title": "Internal Server Error",
  "status": 500,
  "detail": "An unexpected error occurred while processing the request",
  "instance": "/api/azan-wholesale/update-product",
  "response": {
    "success": false,
    "message": "An unexpected error occurred while processing the request",
    "errors": null
  }
}
```

---

### 📦 POST /update-stock

> **Update inventory/stock levels** for products in real-time.

**Endpoint Specifications:**

```
🔗 Method:           POST
📂 Path:             /update-stock
🌐 Full URL:         https://connect.othoba.com/api/azan-wholesale/update-stock
📋 Content-Type:     application/json
🔑 Auth Required:    X-API-Key or Secret-Key header
⏱️ Timeout:          30 seconds
```

**Request Example:**

```
POST /api/azan-wholesale/update-stock HTTP/1.1
Host: connect.othoba.com
Content-Type: application/json
X-API-Key: your-api-key-here

{
  "product_id": "123",
  "sku": "TV-SAMSUNG-65-2024",
  "supplier": "Azan Wholesale",
  "stock": 150
}
```

**✅ Success Response (200 OK):**

```
HTTP/1.1 200 OK
Date: Wed, 15 Jan 2024 10:30:47 GMT
Server: Kestrel
Content-Type: application/json
```

```json
{
  "success": true,
  "message": "Stock updated successfully",
  "errors": null
}
```

**❌ Validation Error (400 Bad Request):**

```
HTTP/1.1 400 Bad Request
Content-Type: application/json
```

```json
{
  "type": "https://connect.othoba.com/api/validation-error",
  "title": "Validation Error",
  "status": 400,
  "detail": "One or more validation errors occurred.",
  "instance": "/api/azan-wholesale/update-stock",
  "response": {
    "success": false,
    "message": "One or more validation errors occurred.",
    "errors": {
      "stock": ["stock cannot be negative"]
    }
  }
}
```

---

## ⚙️ HTTP Status Codes

**Complete HTTP Status Code Reference:**

| 🎯 Code | 📌 Status | 📂 Category | 💬 Meaning | 🔄 Retry |
|---|---|---|---|---|
| ✅ `200` | OK | Success | Request processed successfully | ❌ No |
| ⚠️ `400` | Bad Request | Client Error | Invalid request data/validation error | ❌ No |
| 🔐 `401` | Unauthorized | Auth Error | Invalid or missing API key | ❌ No |
| 🔴 `500` | Server Error | Server Error | Unexpected server error | ✅ Yes |

---

## 💻 Implementation Examples

### 📝 cURL Examples

**Update Product:**

```bash
curl -X POST https://connect.othoba.com/api/azan-wholesale/update-product \
  -H "Content-Type: application/json" \
  -H "X-API-Key: your-api-key-here" \
  -d '{
    "product_id": "123",
    "supplier": "Azan Wholesale",
    "name": "Samsung 65-Inch 4K Smart TV",
    "sku": "TV-SAMSUNG-65-2024",
    "mrp_price": 89999.99,
    "wholesale_price": 75000.00
  }'
```

**Update Stock:**

```bash
curl -X POST https://connect.othoba.com/api/azan-wholesale/update-stock \
  -H "Content-Type: application/json" \
  -H "X-API-Key: your-api-key-here" \
  -d '{
    "product_id": "123",
    "sku": "TV-SAMSUNG-65-2024",
    "supplier": "Azan Wholesale",
    "stock": 150
  }'
```

**Using Secret-Key Header (Alternative):**

```bash
curl -X POST https://connect.othoba.com/api/azan-wholesale/update-product \
  -H "Content-Type: application/json" \
  -H "Secret-Key: your-api-key-here" \
  -d '{
    "product_id": "123",
    "supplier": "Azan Wholesale",
    "name": "Samsung 65-Inch 4K Smart TV",
    "sku": "TV-SAMSUNG-65-2024",
    "mrp_price": 89999.99,
    "wholesale_price": 75000.00
  }'
```

**With Environment Variable:**

```bash
API_KEY="your-api-key-here"

curl -X POST https://connect.othoba.com/api/azan-wholesale/update-product \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $API_KEY" \
  -d '{
    "product_id": "123",
    "supplier": "Azan Wholesale",
    "name": "Samsung 65-Inch 4K Smart TV",
    "sku": "TV-SAMSUNG-65-2024",
    "mrp_price": 89999.99,
    "wholesale_price": 75000.00
  }'
```

**Pretty Print Response:**

```bash
curl -X POST https://connect.othoba.com/api/azan-wholesale/update-product \
  -H "Content-Type: application/json" \
  -H "X-API-Key: your-api-key-here" \
  -d '{
    "product_id": "123",
    "supplier": "Azan Wholesale",
    "name": "Samsung 65-Inch 4K Smart TV",
    "sku": "TV-SAMSUNG-65-2024",
    "mrp_price": 89999.99,
    "wholesale_price": 75000.00
  }' | jq '.'
```

---

### 🐘 PHP Implementation

**Basic Client with Typed Responses:**

```php
<?php
/**
 * Azan Wholesale API Client
 * @version 2.0
 * @author Othoba Platform
 */
class AzanWholesaleAPI {
    private $apiKey;
    private $baseUrl = 'https://connect.othoba.com/api/azan-wholesale';
    private $timeout = 30;
    private $headerName = 'X-API-Key';

    public function __construct($apiKey, $headerName = 'X-API-Key') {
        if (empty($apiKey)) {
            throw new InvalidArgumentException('API key cannot be empty');
        }
        $this->apiKey = $apiKey;
        $this->headerName = $headerName;
    }

    /**
     * Update product information
     * @return array Response with success/error
     */
    public function updateProduct($productId, $supplier, $name, $sku, $mrpPrice, $wholesalePrice) {
        $payload = [
            'product_id' => (string)$productId,
            'supplier' => (string)$supplier,
            'name' => (string)$name,
            'sku' => (string)$sku,
            'mrp_price' => (float)$mrpPrice,
            'wholesale_price' => (float)$wholesalePrice
        ];
        return $this->makeRequest('/update-product', $payload);
    }

    /**
     * Update product stock
     * @return array Response with success/error
     */
    public function updateStock($productId, $sku, $supplier, $stock) {
        $payload = [
            'product_id' => (string)$productId,
            'sku' => (string)$sku,
            'supplier' => (string)$supplier,
            'stock' => (int)$stock
        ];
        return $this->makeRequest('/update-stock', $payload);
    }

    /**
     * Make HTTP request
     */
    private function makeRequest($endpoint, $data) {
        $url = $this->baseUrl . $endpoint;
        $ch = curl_init($url);

        curl_setopt_array($ch, [
            CURLOPT_RETURNTRANSFER => true,
            CURLOPT_POST => true,
            CURLOPT_POSTFIELDS => json_encode($data),
            CURLOPT_HTTPHEADER => [
                'Content-Type: application/json',
                $this->headerName . ': ' . $this->apiKey
            ],
            CURLOPT_TIMEOUT => $this->timeout,
            CURLOPT_SSL_VERIFYPEER => true,
            CURLOPT_SSL_VERIFYHOST => 2
        ]);

        $response = curl_exec($ch);
        $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        $error = curl_error($ch);
        curl_close($ch);

        if ($error) {
            return ['success' => false, 'error' => $error, 'http_code' => 0];
        }

        $decoded = json_decode($response, true);
        
        if ($httpCode === 200) {
            // Success response with typed ApiResponse
            return [
                'success' => true,
                'http_code' => 200,
                'data' => $decoded
            ];
        }

        // Error response with ProblemDetails + nested response
        return [
            'success' => false,
            'data' => $decoded,
            'http_code' => $httpCode
        ];
    }
}

// Usage Example
$api = new AzanWholesaleAPI($_ENV['AZAN_API_KEY'], 'X-API-Key');

// Success case
$result = $api->updateProduct('123', 'Azan', 'Samsung TV', 'TV-001', 89999.99, 75000.00);
if ($result['success'] && $result['http_code'] === 200) {
    echo "✅ " . $result['data']['message'];
}

// Error case
$result = $api->updateProduct('999', 'Azan', 'TV', 'TV-001', -100, 75000.00);
if (!$result['success']) {
    $errors = $result['data']['response']['errors'] ?? [];
    echo "❌ Validation errors: " . json_encode($errors);
}
?>
```

---

### 📜 JavaScript Fetch Implementation

**Basic Client with Typed Responses:**

```javascript
/**
 * Azan Wholesale API Client
 * @version 2.0
 * @author Othoba Platform
 */
class AzanWholesaleAPI {
  constructor(apiKey, headerName = 'X-API-Key', baseUrl = 'https://connect.othoba.com/api/azan-wholesale') {
    if (!apiKey) {
      throw new Error('API key cannot be empty');
    }
    this.apiKey = apiKey;
    this.headerName = headerName;
    this.baseUrl = baseUrl;
    this.timeout = 30000; // 30 seconds in milliseconds
  }

  /**
   * Update product information
   * @returns {Promise<Object>} Response with success/error
   */
  async updateProduct(productId, supplier, name, sku, mrpPrice, wholesalePrice) {
    const payload = {
      product_id: String(productId),
      supplier: supplier,
      name: name,
      sku: sku,
      mrp_price: parseFloat(mrpPrice),
      wholesale_price: parseFloat(wholesalePrice)
    };
    return this.makeRequest('/update-product', payload);
  }

  /**
   * Update product stock
   * @returns {Promise<Object>} Response with success/error
   */
  async updateStock(productId, sku, supplier, stock) {
    const payload = {
      product_id: String(productId),
      sku: sku,
      supplier: supplier,
      stock: parseInt(stock, 10)
    };
    return this.makeRequest('/update-stock', payload);
  }

  /**
   * Make HTTP request
   * @private
   */
  async makeRequest(endpoint, payload) {
    const url = `${this.baseUrl}${endpoint}`;
    const headers = {
      'Content-Type': 'application/json',
      [this.headerName]: this.apiKey
    };

    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), this.timeout);

    try {
      const response = await fetch(url, {
        method: 'POST',
        headers: headers,
        body: JSON.stringify(payload),
        signal: controller.signal
      });

      clearTimeout(timeoutId);

      const data = await response.json();

      if (response.status === 200) {
        // Success response with typed ApiResponse
        return {
          success: true,
          httpCode: 200,
          data: data
        };
      }

      // Error response with ProblemDetails + nested response
      return {
        success: false,
        httpCode: response.status,
        data: data
      };

    } catch (error) {
      clearTimeout(timeoutId);

      if (error.name === 'AbortError') {
        return {
          success: false,
          httpCode: 0,
          error: 'Request timeout (30 seconds)'
        };
      }

      return {
        success: false,
        httpCode: 0,
        error: error.message || 'Network error'
      };
    }
  }
}

// Usage Example
(async () => {
  const api = new AzanWholesaleAPI(
    process.env.AZAN_API_KEY,
    'X-API-Key'
  );

  // Success case
  let result = await api.updateProduct(
    '123',
    'Azan',
    'Samsung TV',
    'TV-001',
    89999.99,
    75000.00
  );

  if (result.success && result.httpCode === 200) {
    console.log('✅', result.data.message);
  }

  // Error case
  result = await api.updateProduct(
    '999',
    'Azan',
    'TV',
    'TV-001',
    -100,
    75000.00
  );

  if (!result.success) {
    const errors = result.data?.response?.errors || {};
    console.error('❌ Validation errors:', JSON.stringify(errors, null, 2));
  }
})();
```

**Usage in Browser Environment:**

```html
<!DOCTYPE html>
<html>
<head>
  <title>Azan Wholesale API Client</title>
  <script src="azan-api-client.js"></script>
</head>
<body>
  <button id="updateBtn">Update Product</button>

  <script>
    const api = new AzanWholesaleAPI(
      'your-api-key-here',
      'X-API-Key'
    );

    document.getElementById('updateBtn').addEventListener('click', async () => {
      const result = await api.updateProduct(
        '123',
        'Azan Wholesale',
        'Samsung 65-Inch 4K Smart TV',
        'TV-SAMSUNG-65-2024',
        89999.99,
        75000.00
      );

      if (result.success && result.httpCode === 200) {
        alert('✅ ' + result.data.message);
      } else {
        const errors = result.data?.response?.errors || {};
        alert('❌ Error: ' + JSON.stringify(errors));
      }
    });
  </script>
</body>
</html>
```

**Retry with Exponential Backoff:**

```javascript
/**
 * Retry helper with exponential backoff
 */
class RetryableAzanAPI extends AzanWholesaleAPI {
  async updateProductWithRetry(productId, supplier, name, sku, mrpPrice, wholesalePrice, maxRetries = 3) {
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
      const result = await this.updateProduct(productId, supplier, name, sku, mrpPrice, wholesalePrice);

      // Success or client error (no retry needed)
      if (result.success || result.httpCode === 400 || result.httpCode === 401) {
        return result;
      }

      // Server error - retry with exponential backoff
      if (result.httpCode === 500 && attempt < maxRetries) {
        const backoffMs = Math.pow(2, attempt - 1) * 1000; // 1s, 2s, 4s
        console.log(`🔄 Retry attempt ${attempt}/${maxRetries} after ${backoffMs}ms`);
        await new Promise(resolve => setTimeout(resolve, backoffMs));
        continue;
      }

      return result;
    }
  }
}

// Usage with retry
(async () => {
  const api = new RetryableAzanAPI(process.env.AZAN_API_KEY);
  const result = await api.updateProductWithRetry(
    '123',
    'Azan',
    'Samsung TV',
    'TV-001',
    89999.99,
    75000.00
  );

  if (result.success) {
    console.log('✅ Success:', result.data.message);
  } else {
    console.error('❌ Failed after retries:', result.data);
  }
})();
```

---

## ⚠️ Error Handling

### Client-Side Best Practices

```javascript
async function updateProduct(data) {
  try {
    const response = await fetch('https://connect.othoba.com/api/azan-wholesale/update-product', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'X-API-Key': process.env.AZAN_API_KEY
      },
      body: JSON.stringify(data)
    });

    const json = await response.json();

    if (response.status === 200) {
      // ✅ Success - check typed ApiResponse
      console.log('✅ Success:', json.message);
      return json;
    } 
    
    if (response.status === 400) {
      // ⚠️ Validation error - check nested response
      const errors = json.response?.errors || {};
      console.error('❌ Validation failed:', errors);
      return null;
    } 
    
    if (response.status === 401) {
      // 🔐 Auth error
      console.error('🔐 Auth failed:', json.response?.message);
      return null;
    } 
    
    if (response.status === 500) {
      // 🔴 Server error - retry with backoff
      console.error('🔴 Server error:', json.response?.message);
      return retryWithBackoff(data);
    }
    
  } catch (error) {
    console.error('Network error:', error);
  }
}
```

---

## 📊 Performance & Limits

### ⚡ Rate Limiting Policy

| 🎯 Metric | 📊 Limit | 💡 Recommendation |
|---|---|---|
| 📈 **Hourly Limit** | 10,000 requests/hour | Plan accordingly |
| ⏱️ **Recommended Delay** | 100-200ms between requests | Avoid bursts |
| 🚀 **Max Burst Rate** | 5 requests/second | Short burst allowed |

### 📈 Performance Metrics

| 📊 Metric | ⏱️ Duration | 📝 Notes |
|---|---|---|
| ⚡ **Average Response** | 100-200ms | Typical case |
| 📈 **P95 Response** | 500ms | 95th percentile |
| 📊 **P99 Response** | 1000ms | 99th percentile |
| 🌐 **Availability** | 99.9% | SLA guaranteed |

---

## 📞 Support & Contact

### 🆘 Get Help

| 📱 Channel | 🔗 Contact | ⏱️ Hours |
|---|---|---|
| 📧 **Email** | support@othoba.com | 24/7 Support |
| 💬 **Documentation** | See this guide | Always available |

---

## 🙋 FAQ

**Q: What's the response format for successful requests?**  
A: HTTP 200 with a strongly-typed `ApiResponse` object containing `success: true`, `message`, and `errors: null` fields.

**Q: How are errors returned?**  
A: Errors use RFC 7807 `ProblemDetails` format with a nested `response` field containing the `ApiResponse` object with error details, error types, and HTTP status codes.

**Q: Which header should I use - X-API-Key or Secret-Key?**  
A: Both are fully supported! The API checks `X-API-Key` first, then `Secret-Key` if not found. Use whichever is more appropriate for your integration.

**Q: Can I switch between header names?**  
A: Yes, both headers are always supported simultaneously without any code changes needed.

**Q: What happens if I exceed the rate limit?**  
A: Requests will be rejected with HTTP 429. Implement exponential backoff retry strategy (1s, 2s, 4s).

**Q: How do I handle validation errors in my client?**  
A: Parse the `response.errors` field from the `ProblemDetails`. It contains field-level errors as arrays of strings grouped by field name.

**Q: Is there a SDK available?**  
A: SDK examples are provided for PHP and JavaScript. Both follow the same error handling and response patterns.

**Q: How do I log API requests safely?**  
A: Never log complete API keys. Mask all but the last 4 characters. Example: `sk_live_****789jkl`

**Q: What's the timeout for API requests?**  
A: 30 seconds per request. Implement client-side timeouts to match this limit.

**Q: Can I batch multiple updates?**  
A: No, each request must be a single product or stock update. Implement client-side batching with 100-200ms delays between requests.

---

## 📄 Document Information

| 🏷️ Property | 📝 Value |
|---|---|
| **Version** | 2.1 |
| **Last Updated** | January 2024 |
| **Status** | ✅ Production Ready |
| **Framework** | ASP.NET Core 6 (.NET 6) |
| **Response Model** | Typed ApiResponse + RFC 7807 ProblemDetails |
| **Architecture** | RESTful, Stateless, Async/Await |
| **SDK Examples** | PHP, JavaScript (Fetch) |
| **License** | Proprietary |

---

> 💡 **Pro Tip:** This documentation reflects the production implementation. All responses are strongly-typed and follow industry best practices (RFC 7807 ProblemDetails, async/await patterns, comprehensive error handling).

---

**© 2026 Othoba Platform. All rights reserved. Confidential.**
