# 🚀 API Azan Wholesale 

> **Professional Technical Reference for Product & Inventory Synchronization**

<div style="background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); padding: 20px; border-radius: 8px; color: white; margin-bottom: 30px;">

**📌 Version:** 1.1 | **📅 Released:** February 2026 | **Updated:** January 2024


```
🔗 API Endpoint:  https://connect.othoba.com/api/azan-wholesale
🔒 Protocol:      REST (HTTPS/TLS 1.2+)
📦 Format:        JSON (application/json)
🔑 Auth:          API Key (X-API-Key or Secret-Key Header)
⚡ Framework:     ASP.NET Core 6

```

</div>

## 📋 Quick Overview

The **API for Azan Wholesale** provides a modern, secure REST interface for real-time synchronization of:

- ✅ **Product Information** - Names, pricing, supplier details
- ✅ **Inventory Levels** - Stock quantities, availability
- ✅ **Error Reporting** - Detailed validation feedback

### Key Highlights

| 🎯 Feature | 📝 Details |
|-----------|-----------|
| **Performance** | 100-200ms average response time |
| **Scalability** | Stateless, distributed architecture |
| **Security** | API Key + HTTPS/TLS 1.2+ |
| **Validation** | Comprehensive input validation |
| **Reliability** | Exponential backoff support |
| **Rate Limit** | 1,000 requests/hour |

---

## 📚 Table of Contents

- [🔐 Authentication & Security](#-authentication--security)
- [📦 API Models & Schemas](#-api-models--schemas)
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

### ❌ Without Authentication (Missing API Key)


```
POST /api/azan-wholesale/update-product HTTP/1.1
Host: connect.othoba.com
Content-Type: application/json

{ "product_id": "123", ... }

```

**Response:** `❌ 401 Unauthorized`

### ✅ With Authentication (Using X-API-Key)


```
POST /api/azan-wholesale/update-product HTTP/1.1
Host: connect.othoba.com
Content-Type: application/json
X-API-Key: sk_live_abc123def456ghi789jkl

{ "product_id": "123", ... }

```

**Response:** `✅ 204 No Content`

### ✅ Alternative Authentication (Using Secret-Key)


```
POST /api/azan-wholesale/update-product HTTP/1.1
Host: connect.othoba.com
Content-Type: application/json
Secret-Key: sk_live_abc123def456ghi789jkl

{ "product_id": "123", ... }

```

**Response:** `✅ 204 No Content`

### 🛡️ Security Best Practices

| 🔐 Practice | 💡 Implementation |
|---|---|
| 🔒 **Storage** | Use secure environment variables (never hardcode) |
| 📋 **Logging** | Never log complete keys; mask all but last 4 chars |
| 🔄 **Rotation** | Rotate API keys quarterly minimum |
| 🚫 **Access Control** | Restrict to specific IP ranges when possible |
| 👁️ **Monitoring** | Alert on unusual patterns, repeated failures |
| 🚨 **Revocation** | Immediately revoke compromised keys |

**Environment Variable Example:**

```sh
export AZAN_API_KEY="sk_live_your_actual_key_here"

```

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

**Validation Rules:**
- ✅ All fields mandatory; null values rejected
- ✅ String fields must not be empty or whitespace-only
- ✅ `product_id` must exist in Othoba system
- ✅ Prices non-negative with max 2 decimal places
- ✅ String length constraints enforced

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

**Validation Rules:**
- ✅ All fields mandatory
- ✅ `stock` must be non-negative integer
- ✅ `product_id` must match existing product
- ✅ `sku` must match product's SKU in system
- ✅ Stock value limited to 32-bit signed integer

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

**✅ Success Response (204 No Content):**

```
HTTP/1.1 204 No Content
Date: Wed, 15 Jan 2024 10:30:45 GMT
Server: Kestrel
Content-Length: 0

```

**❌ Validation Error (400 Bad Request):**

```
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "errors": {
    "product_id": ["product_id is required"],
    "mrp_price": ["mrp_price cannot be negative"]
  }
}

```

**🔐 Auth Error (401 Unauthorized):**

```
HTTP/1.1 401 Unauthorized
Content-Type: application/json

{
  "error": "Invalid or missing API key"
}

```

**⚠️ Server Error (500 Internal Server Error):**

```
HTTP/1.1 500 Internal Server Error
Content-Type: application/json

{
  "error": "An unexpected error occurred while processing the request"
}

```

**Response Status Reference:**

| 🎯 Code | 📌 Status | 💬 Meaning | 👉 Action |
|---|---|---|---|
| ✅ `204` | No Content | Successfully processed | No action needed |
| ⚠️ `400` | Bad Request | Validation failed | Check error details & retry |
| 🔐 `401` | Unauthorized | Auth failed | Verify API key |
| 🔴 `500` | Server Error | Server error | Retry with exponential backoff |

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

**✅ Success Response (204 No Content):**

```
HTTP/1.1 204 No Content
Date: Wed, 15 Jan 2024 10:30:47 GMT
Server: Kestrel
Content-Length: 0

```

**❌ Validation Error (400 Bad Request):**

```
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "errors": {
    "stock": ["stock cannot be negative"]
  }
}

```

**Response Status Reference:**

| 🎯 Code | 📌 Status | 💬 Meaning | 👉 Action |
|---|---|---|---|
| ✅ `204` | No Content | Successfully processed | No action needed |
| ⚠️ `400` | Bad Request | Validation failed | Check error details & retry |
| 🔐 `401` | Unauthorized | Auth failed | Verify API key |
| 🔴 `500` | Server Error | Server error | Retry with exponential backoff |

---

## ⚙️ HTTP Status Codes

**Complete HTTP Status Code Reference:**

| 🎯 Code | 📌 Status | 📂 Category | 💬 Meaning | 🔄 Retry |
|---|---|---|---|---|
| ✅ `204` | No Content | Success | Request processed successfully | ❌ No |
| ⚠️ `400` | Bad Request | Client Error | Invalid request data/validation error | ❌ No |
| 🔐 `401` | Unauthorized | Auth Error | Invalid or missing API key | ❌ No |
| 🔴 `500` | Server Error | Server Error | Unexpected server error | ✅ Yes |

### 📋 Error Response Structures

**Validation Errors (400):**

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

**Authentication Errors (401):**

```json
{
  "error": "Invalid or missing API key"
}

```

**Server Errors (500):**

```json
{
  "error": "An unexpected error occurred while processing the request"
}

```

### 💡 Response Handling Guide

| 🎯 Scenario | 📌 Status | 🎯 How to Handle |
|---|---|---|
| 📤 Request succeeded | `204` | ✅ Success - proceed normally |
| 🔍 Invalid data sent | `400` | ❌ Fix data, check error details, retry |
| 🔑 Auth failed | `401` | ❌ Verify API key is correct and header name (supports X-API-Key or Secret-Key) |
| ⚠️ Server error | `500` | ⏳ Implement exponential backoff & retry |

---

## 💻 Implementation Examples

### 🐘 PHP Implementation

**Basic Client:**

```php
<?php
/**
 * Azan Wholesale API Client
 * @version 1.0
 * @author Othoba Platform
 */
class AzanWholesaleAPI {
    private $apiKey;
    private $baseUrl = 'https://connect.othoba.com/api/azan-wholesale';
    private $timeout = 30;
    private $headerName = 'X-API-Key'; // Can also use 'Secret-Key'

    public function __construct($apiKey, $headerName = 'X-API-Key') {
        if (empty($apiKey)) {
            throw new InvalidArgumentException('API key cannot be empty');
        }
        $this->apiKey = $apiKey;
        $this->headerName = $headerName; // 'X-API-Key' or 'Secret-Key'
    }

    /**
     * Update product information
     * @return array API response
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
     * @return array API response
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

        if ($httpCode === 204) {
            return ['success' => true, 'http_code' => 204];
        }

        $decoded = json_decode($response, true);
        return ['success' => false, 'data' => $decoded, 'http_code' => $httpCode];
    }
}

// Usage Example with X-API-Key
$api = new AzanWholesaleAPI($_ENV['AZAN_API_KEY'], 'X-API-Key');
$result = $api->updateProduct('123', 'Azan', 'Samsung TV', 'TV-001', 89999.99, 75000.00);
echo $result['success'] ? 'Success!' : 'Error: ' . json_encode($result['data']);

// Or use Secret-Key header
$api = new AzanWholesaleAPI($_ENV['AZAN_API_KEY'], 'Secret-Key');
$result = $api->updateProduct('123', 'Azan', 'Samsung TV', 'TV-001', 89999.99, 75000.00);
?>

```

---

### 🐍 Python Implementation

**Basic Client:**

```python
import requests
import json
from typing import Dict, Any

class AzanWholesaleAPI:
    """Azan Wholesale API Client"""

    def __init__(self, api_key: str, header_name: str = "X-API-Key", 
                 base_url: str = "https://connect.othoba.com/api/azan-wholesale"):
        if not api_key:
            raise ValueError("API key cannot be empty")
        self.api_key = api_key
        self.header_name = header_name  # 'X-API-Key' or 'Secret-Key'
        self.base_url = base_url
        self.timeout = 30

    def update_product(self, product_id: str, supplier: str, name: str, sku: str,
                      mrp_price: float, wholesale_price: float) -> Dict[str, Any]:
        """Update product information"""
        payload = {
            "product_id": str(product_id),
            "supplier": supplier,
            "name": name,
            "sku": sku,
            "mrp_price": mrp_price,
            "wholesale_price": wholesale_price
        }
        return self._make_request('/update-product', payload)

    def update_stock(self, product_id: str, sku: str, supplier: str, stock: int) -> Dict[str, Any]:
        """Update product stock"""
        payload = {
            "product_id": str(product_id),
            "sku": sku,
            "supplier": supplier,
            "stock": stock
        }
        return self._make_request('/update-stock', payload)

    def _make_request(self, endpoint: str, payload: Dict[str, Any]) -> Dict[str, Any]:
        """Make HTTP request"""
        url = f"{self.base_url}{endpoint}"
        headers = {
            "Content-Type": "application/json",
            self.header_name: self.api_key
        }

        try:
            response = requests.post(url, headers=headers, json=payload, timeout=self.timeout)
            if response.status_code == 204:
                return {"success": True}
            return {"success": False, "errors": response.json(), "http_code": response.status_code}
        except requests.RequestException as e:
            return {"success": False, "error": str(e)}

# Usage with X-API-Key
api = AzanWholesaleAPI(api_key="your-api-key", header_name="X-API-Key")
result = api.update_product("123", "Azan", "Samsung TV", "TV-001", 89999.99, 75000.00)
print("Success!" if result['success'] else f"Error: {result}")

# Or use Secret-Key header
api = AzanWholesaleAPI(api_key="your-api-key", header_name="Secret-Key")
result = api.update_product("123", "Azan", "Samsung TV", "TV-001", 89999.99, 75000.00)

```

---

## ⚠️ Error Handling

### Common Error Scenarios

**Scenario 1️⃣ : Invalid Product ID**

```json
Request:  { "product_id": "999", ... }
Response: { "errors": { "product_id": ["product_id must exist in system"] } }
Status:   400 Bad Request

```

**Scenario 2️⃣ : Negative Price**

```json
Request:  { "mrp_price": -100, ... }
Response: { "errors": { "mrp_price": ["mrp_price cannot be negative"] } }
Status:   400 Bad Request

```

**Scenario 3️⃣ : Missing API Key**

```
Request:  (no X-API-Key or Secret-Key header)
Response: { "error": "Invalid or missing API key" }
Status:   401 Unauthorized

```

**Scenario 4️⃣ : Server Error**

```json
Response: { "error": "An unexpected error occurred while processing the request" }
Status:   500 Internal Server Error
Action:   Retry with exponential backoff (1s, 2s, 4s)

```

### 🛠️ Client-Side Error Handling Best Practices


```javascript
// Pseudo-code pattern
if (response.status === 204) {
  // ✅ Success
  console.log('Request processed successfully');
} else if (response.status === 400) {
  // ⚠️ Validation error - don't retry
  console.error('Invalid data:', response.errors);
} else if (response.status === 401) {
  // 🔐 Auth error - check API key and header name
  console.error('Invalid API key or header. Supported headers: X-API-Key, Secret-Key');
} else if (response.status === 500) {
  // 🔴 Server error - retry with backoff
  retryWithExponentialBackoff();
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
| 💾 **Batch Size** | 50-100 items | Optimize performance |

### 📈 Performance Metrics

| 📊 Metric | ⏱️ Duration | 📝 Notes |
|---|---|---|
| ⚡ **Average Response** | 100-200ms | Typical case |
| 📈 **P95 Response** | 500ms | 95th percentile |
| 📊 **P99 Response** | 1000ms | 99th percentile |
| 🌐 **Availability** | 99.9% | SLA guaranteed |

### 💡 Optimization Tips

✅ **Do:**
- Use batch operations when possible
- Implement exponential backoff for retries
- Cache responses where appropriate
- Monitor API usage patterns
- Support both X-API-Key and Secret-Key headers in client implementations

❌ **Don't:**
- Make synchronous sequential calls
- Send duplicate requests
- Bypass rate limiting
- Store API keys in code

---

## 📞 Support & Contact

### 🆘 Get Help

| 📱 Channel | 🔗 Contact | ⏱️ Hours |
|---|---|---|
| 📧 **Email** | support@othoba.com | 24/7 Support |
| 💬 **Documentation** | See this guide | Always available |

### 📚 Additional Resources

- 📖 **This API Reference** - Complete technical documentation
- 🧪 **Testing Guide** - Validation testing procedures
- 💻 **Code Examples** - PHP & Python implementations
- 🔧 **Implementation Guide** - Step-by-step integration

---

## 🎓 Quick Start Checklist

Before going live:

- [ ] 🔑 Obtain API key from Othoba
- [ ] 📝 Review authentication requirements
- [ ] 💾 Store API key in environment variables
- [ ] 🧪 Test endpoints in development environment
- [ ] 📊 Implement error handling & retry logic
- [ ] ⚡ Set up rate limiting (100-200ms between requests)
- [ ] 📋 Validate request payloads
- [ ] 🔍 Test all error scenarios
- [ ] 📈 Monitor API usage in production
- [ ] 🆘 Configure alerting for failures

---

## 🙋 FAQ

**Q: How do I get an API key?**  
A: Contact support@othoba.com to request an API key.

**Q: Can I use HTTP instead of HTTPS?**  
A: No, HTTPS/TLS 1.2+ is mandatory for security.

**Q: What's the request timeout?**  
A: 30 seconds per request.

**Q: Can I batch multiple updates?**  
A: Yes, implement client-side batching with 100-200ms delays between requests.

**Q: What happens if I exceed the rate limit?**  
A: Requests will be rejected. Implement exponential backoff retry strategy.

**Q: Which header should I use for API key - X-API-Key or Secret-Key?**  
A: Both are supported! The API checks `X-API-Key` first, then falls back to `Secret-Key` if not found. Use whichever is more appropriate for your integration.

**Q: Do I need to change my implementation if I want to switch between header names?**  
A: No, both headers are always supported simultaneously. You can use either one, or even migrate from one to the other without service disruption.

---

## 📄 Document Information

| 🏷️ Property | 📝 Value |
|---|---|
| **Version** | 1.1 |
| **Last Updated** | January 2024 |
| **Status** | Production Ready |
| **Framework** | ASP.NET Core 6 |
| **Language** | C# 10 |
| **License** | Proprietary |

---

> 💡 **Pro Tip:** Keep this documentation handy during integration. Refer to specific sections when debugging API issues.

---

**© 2024 Othoba Platform. All rights reserved.**
