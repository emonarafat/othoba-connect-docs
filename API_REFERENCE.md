# API for Azan Wholesale  - Professional Technical Reference

**Document Version:** 1.0  
**Release Date:** January 2024  
**API Endpoint:** `https://connect.othoba.com/api/azan-wholesale`  
**API Protocol:** REST (HTTPS/TLS 1.2+)  
**Content Negotiation:** JSON (application/json)  
**Authentication Scheme:** API Key (HTTP Header)  

---

## Executive Summary

The Azan Wholesale API enables secure, RESTful synchronization of product information and inventory levels between Azan Wholesale's supply chain management system and the Othoba e-commerce platform. The API provides two primary operations: product data updates and real-time stock synchronization.

**Key Features:**
- High-performance REST API with JSON payloads
- Stateless, scalable architecture
- Comprehensive input validation
- Detailed error reporting
- Rate limiting and backoff support
- Full SSL/TLS encryption

---

## Table of Contents

- [Authentication & Security](#authentication--security)
- [API Models & Schemas](#api-models--schemas)
- [Endpoints Reference](#endpoints-reference)
- [HTTP Status Codes](#http-status-codes)
- [Implementation Examples](#implementation-examples)
- [Error Handling](#error-handling)
- [Rate Limiting & Performance](#rate-limiting--performance)

---

## Authentication & Security

### API Key Authentication

All requests require authentication via API Key transmitted in the HTTP request header.

**Authentication Header Format:**
```
X-API-Key: {api-key}
```

**Authentication Requirements:**
| Requirement | Specification |
|-------------|---------------|
| **Protocol** | HTTPS/TLS 1.2 or higher (mandatory) |
| **Header Name** | X-API-Key (case-insensitive) |
| **Key Format** | Alphanumeric string (provided by Othoba) |
| **Transmission** | HTTP Header only (never in URL or body) |
| **Validation** | Performed on every request |

**Invalid Request Example (Missing Auth):**
```http
POST /api/azan-wholesale/update-product HTTP/1.1
Host: connect.othoba.com
Content-Type: application/json

{...}
```
**Result:** `401 Unauthorized` - Missing API Key

**Valid Request Example:**
```http
POST /api/azan-wholesale/update-product HTTP/1.1
Host: connect.othoba.com
Content-Type: application/json
X-API-Key: sk_live_abc123def456ghi789jkl

{...}
```
**Result:** `204 No Content` - Authenticated and processed

### Security Best Practices

| Practice | Implementation Details |
|----------|------------------------|
| **Storage** | Store API keys in secure environment variables, never hardcode |
| **Logging** | Never log complete API keys; mask all but last 4 characters |
| **Rotation** | Rotate API keys quarterly minimum |
| **Access Control** | Restrict API key usage to specific IP ranges if possible |
| **Monitoring** | Monitor for unusual API usage patterns or repeated failures |
| **Revocation** | Immediately revoke compromised keys; request new key from Othoba |

---

## API Models & Schemas

### ProductUpdateRequest

**Purpose:** Synchronize product details to the Othoba platform.

**JSON Schema:**
```json
{
  "type": "object",
  "properties": {
    "product_id": { "type": "string" },
    "supplier": { "type": "string" },
    "name": { "type": "string" },
    "sku": { "type": "string" },
    "mrp_price": { "type": "number" },
    "wholesale_price": { "type": "number" }
  },
  "required": ["product_id", "supplier", "name", "sku", "mrp_price", "wholesale_price"]
}
```

**Field Reference:**

| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| `product_id` | String | ✓ | Non-empty, alphanumeric | Unique product identifier in Othoba system |
| `supplier` | String | ✓ | 1-500 chars | Supplier/vendor name or identifier |
| `name` | String | ✓ | 1-1000 chars | Product name/title/description |
| `sku` | String | ✓ | 1-500 chars, alphanumeric | Stock Keeping Unit code |
| `mrp_price` | Decimal | ✓ | ≥ 0, max 2 decimals | Maximum Retail Price (in local currency) |
| `wholesale_price` | Decimal | ✓ | ≥ 0, max 2 decimals | Wholesale/cost price (in local currency) |

**Example Request Payload:**
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
- All fields mandatory; null values rejected
- String fields must not be empty or whitespace-only
- `product_id` must exist in Othoba system
- Prices must be non-negative with maximum 2 decimal places
- String length constraints enforced (field-level validation)

---

### StockUpdateRequest

**Purpose:** Update inventory/stock levels for existing products.

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

| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| `product_id` | String | ✓ | Non-empty, alphanumeric | Product identifier (must exist) |
| `sku` | String | ✓ | 1-500 chars, alphanumeric | Stock Keeping Unit |
| `supplier` | String | ✓ | 1-500 chars | Supplier identifier |
| `stock` | Integer | ✓ | ≥ 0, ≤ 2,147,483,647 | Available quantity (units) |

**Example Request Payload:**
```json
{
  "product_id": "123",
  "sku": "TV-SAMSUNG-65-2024",
  "supplier": "Azan Wholesale",
  "stock": 150
}
```

**Validation Rules:**
- All fields mandatory
- `stock` must be non-negative integer
- `product_id` must match existing product
- `sku` must match product's SKU in system
- Stock value limited to 32-bit signed integer range

---

## Endpoints Reference

### POST /update-product

**Summary:** Update product information including name, pricing, and supplier details.

**Endpoint Details:**
```
Method:            POST
Path:              /update-product
Full URL:          https://connect.othoba.com/api/azan-wholesale/update-product
Content-Type:      application/json
Authentication:    Required (X-API-Key header)
Request Timeout:   30 seconds
```

**Request Format:**
```http
POST /api/azan-wholesale/update-product HTTP/1.1
Host: connect.othoba.com
Content-Type: application/json
Content-Length: 242
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

**Successful Response (204 No Content):**
```http
HTTP/1.1 204 No Content
Date: Wed, 15 Jan 2024 10:30:45 GMT
Server: Kestrel
Content-Length: 0
```

**Validation Error Response (400 Bad Request):**
```http
HTTP/1.1 400 Bad Request
Content-Type: application/json
Content-Length: 315
Date: Wed, 15 Jan 2024 10:30:46 GMT

{
  "errors": {
    "product_id": [
      "product_id is required"
    ],
    "supplier": [
      "supplier is required"
    ],
    "mrp_price": [
      "mrp_price cannot be negative"
    ]
  }
}
```

**Unauthorized Response (401 Unauthorized):**
```http
HTTP/1.1 401 Unauthorized
Content-Type: application/json
Content-Length: 45

{
  "error": "Invalid or missing API key"
}
```

**Server Error Response (500 Internal Server Error):**
```http
HTTP/1.1 500 Internal Server Error
Content-Type: application/json
Content-Length: 95

{
  "error": "An unexpected error occurred while processing the request"
}
```

**Status Codes:**

| Code | Status | Meaning | Recommended Action |
|------|--------|---------|-------------------|
| `204` | No Content | Successfully processed | No action needed |
| `400` | Bad Request | Validation failed | Check error details and retry |
| `401` | Unauthorized | Auth failed | Verify API key |
| `500` | Internal Server Error | Server error | Retry with exponential backoff |

---

### POST /update-stock

**Summary:** Update inventory/stock levels for products.

**Endpoint Details:**
```
Method:            POST
Path:              /update-stock
Full URL:          https://connect.othoba.com/api/azan-wholesale/update-stock
Content-Type:      application/json
Authentication:    Required (X-API-Key header)
Request Timeout:   30 seconds
```

**Request Format:**
```http
POST /api/azan-wholesale/update-stock HTTP/1.1
Host: connect.othoba.com
Content-Type: application/json
Content-Length: 125
X-API-Key: your-api-key-here

{
  "product_id": "123",
  "sku": "TV-SAMSUNG-65-2024",
  "supplier": "Azan Wholesale",
  "stock": 150
}
```

**Successful Response (204 No Content):**
```http
HTTP/1.1 204 No Content
Date: Wed, 15 Jan 2024 10:30:47 GMT
Server: Kestrel
Content-Length: 0
```

**Validation Error Response (400 Bad Request):**
```http
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "errors": {
    "stock": [
      "stock cannot be negative"
    ]
  }
}
```

**Status Codes:**

| Code | Status | Meaning | Recommended Action |
|------|--------|---------|-------------------|
| `204` | No Content | Successfully processed | No action needed |
| `400` | Bad Request | Validation failed | Check error details and retry |
| `401` | Unauthorized | Auth failed | Verify API key |
| `500` | Internal Server Error | Server error | Retry with exponential backoff |

---

## HTTP Status Codes

**Complete HTTP Status Code Reference:**

| Code | Status | Category | Meaning | Retry Policy |
|------|--------|----------|---------|--------------|
| `204` | No Content | Success | Request processed successfully | No |
| `400` | Bad Request | Client Error | Invalid request data/validation error | No (fix data first) |
| `401` | Unauthorized | Auth Error | Invalid or missing API key | No (fix credentials) |
| `500` | Server Error | Server Error | Unexpected server error | Yes (exponential backoff) |

**Error Response Structure:**

All error responses return JSON with the following format:

**Validation Errors:**
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

**Authentication Errors:**
```json
{
  "error": "Invalid or missing API key"
}
```

**Server Errors:**
```json
{
  "error": "An unexpected error occurred while processing the request"
}
```

---

## Implementation Examples

### PHP Implementation

**Basic Client:**
```php
<?php
/**
 * Azan Wholesale API Client
 * @version 1.0
 */
class AzanWholesaleAPI {
    private $apiKey;
    private $baseUrl = 'https://connect.othoba.com/api/azan-wholesale';
    private $timeout = 30;

    public function __construct($apiKey) {
        if (empty($apiKey)) {
            throw new InvalidArgumentException('API key cannot be empty');
        }
        $this->apiKey = $apiKey;
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
                'X-API-Key: ' . $this->apiKey
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

// Usage Example
$api = new AzanWholesaleAPI($_ENV['AZAN_API_KEY']);
$result = $api->updateProduct('123', 'Azan', 'Samsung TV', 'TV-001', 89999.99, 75000.00);
echo $result['success'] ? 'Success!' : 'Error: ' . json_encode($result['data']);
?>
```

**Advanced Client with Batch Processing:**
```php
<?php
class AzanWholesaleAPIAdvanced {
    private $apiKey;
    private $baseUrl = 'https://connect.othoba.com/api/azan-wholesale';
    private $timeout = 30;
    private $maxRetries = 3;

    public function __construct($apiKey) {
        $this->apiKey = $apiKey;
    }

    /**
     * Batch update products with retry logic
     */
    public function batchUpdateProducts(array $products) {
        $results = ['successful' => 0, 'failed' => 0, 'items' => []];

        foreach ($products as $product) {
            $result = $this->updateProductWithRetry(
                $product['product_id'],
                $product['supplier'],
                $product['name'],
                $product['sku'],
                $product['mrp_price'],
                $product['wholesale_price']
            );

            if ($result['success']) {
                $results['successful']++;
            } else {
                $results['failed']++;
            }

            $results['items'][] = [
                'product_id' => $product['product_id'],
                'status' => $result['success'] ? 'SUCCESS' : 'FAILED'
            ];

            usleep(100000); // Rate limiting
        }

        return $results;
    }

    /**
     * Update with exponential backoff retry
     */
    private function updateProductWithRetry($productId, $supplier, $name, $sku, $mrpPrice, $wholesalePrice, $attempt = 1) {
        $payload = [
            'product_id' => (string)$productId,
            'supplier' => (string)$supplier,
            'name' => (string)$name,
            'sku' => (string)$sku,
            'mrp_price' => (float)$mrpPrice,
            'wholesale_price' => (float)$wholesalePrice
        ];

        $result = $this->makeRequest('/update-product', $payload);

        // Retry on 500 errors with exponential backoff
        if (!$result['success'] && $result['http_code'] === 500 && $attempt < $this->maxRetries) {
            $waitTime = pow(2, $attempt - 1);
            sleep($waitTime);
            return $this->updateProductWithRetry($productId, $supplier, $name, $sku, $mrpPrice, $wholesalePrice, $attempt + 1);
        }

        return $result;
    }

    private function makeRequest($endpoint, $data) {
        $url = $this->baseUrl . $endpoint;
        $ch = curl_init($url);

        curl_setopt_array($ch, [
            CURLOPT_RETURNTRANSFER => true,
            CURLOPT_POST => true,
            CURLOPT_POSTFIELDS => json_encode($data),
            CURLOPT_HTTPHEADER => ['Content-Type: application/json', 'X-API-Key: ' . $this->apiKey],
            CURLOPT_TIMEOUT => $this->timeout,
            CURLOPT_SSL_VERIFYPEER => true,
            CURLOPT_SSL_VERIFYHOST => 2
        ]);

        $response = curl_exec($ch);
        $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        curl_close($ch);

        return $httpCode === 204
            ? ['success' => true, 'http_code' => 204]
            : ['success' => false, 'data' => json_decode($response, true), 'http_code' => $httpCode];
    }
}
?>
```

---

### Python Implementation

**Basic Client:**
```python
import requests
import json
from typing import Dict, Any

class AzanWholesaleAPI:
    """Azan Wholesale API Client"""

    def __init__(self, api_key: str, base_url: str = "https://connect.othoba.com/api/azan-wholesale"):
        if not api_key:
            raise ValueError("API key cannot be empty")
        self.api_key = api_key
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
        headers = {"Content-Type": "application/json", "X-API-Key": self.api_key}

        try:
            response = requests.post(url, headers=headers, json=payload, timeout=self.timeout)
            if response.status_code == 204:
                return {"success": True}
            return {"success": False, "errors": response.json(), "http_code": response.status_code}
        except requests.RequestException as e:
            return {"success": False, "error": str(e)}

# Usage
api = AzanWholesaleAPI(api_key="your-api-key")
result = api.update_product("123", "Azan", "Samsung TV", "TV-001", 89999.99, 75000.00)
print("Success!" if result['success'] else f"Error: {result}")
```

**Advanced Client with Batch Processing:**
```python
import requests
import json
import time
import logging
from typing import Dict, Any, List
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

class AzanWholesaleAPIAdvanced:
    """Advanced Azan Wholesale API Client with retry logic"""

    def __init__(self, api_key: str, base_url: str = "https://connect.othoba.com/api/azan-wholesale"):
        self.api_key = api_key
        self.base_url = base_url
        self.timeout = 30
        self.logger = logging.getLogger(__name__)
        self.session = self._create_session()

    def _create_session(self) -> requests.Session:
        """Create session with retry strategy"""
        session = requests.Session()
        retry_strategy = Retry(total=3, status_forcelist=[500, 502, 503], backoff_factor=1)
        adapter = HTTPAdapter(max_retries=retry_strategy)
        session.mount("https://", adapter)
        return session

    def batch_update_products(self, products: List[Dict[str, Any]]) -> Dict[str, Any]:
        """Batch update products with rate limiting"""
        results = {"successful": 0, "failed": 0, "items": []}

        for i, product in enumerate(products):
            result = self.update_product(
                product['product_id'], product['supplier'], product['name'],
                product['sku'], product['mrp_price'], product['wholesale_price']
            )

            if result['success']:
                results['successful'] += 1
            else:
                results['failed'] += 1

            results['items'].append({'product_id': product['product_id'], 'status': 'SUCCESS' if result['success'] else 'FAILED'})
            time.sleep(0.1)  # Rate limiting

        return results

    def update_product(self, product_id: str, supplier: str, name: str, sku: str,
                      mrp_price: float, wholesale_price: float) -> Dict[str, Any]:
        """Update product with validation"""
        if not all([product_id, supplier, name, sku]):
            return {"success": False, "error": "Missing required fields"}

        payload = {
            "product_id": str(product_id),
            "supplier": supplier,
            "name": name,
            "sku": sku,
            "mrp_price": mrp_price,
            "wholesale_price": wholesale_price
        }
        return self._make_request('/update-product', payload)

    def _make_request(self, endpoint: str, payload: Dict[str, Any]) -> Dict[str, Any]:
        """Make HTTP request with error handling"""
        url = f"{self.base_url}{endpoint}"
        headers = {"Content-Type": "application/json", "X-API-Key": self.api_key}

        try:
            response = self.session.post(url, headers=headers, json=payload, timeout=self.timeout)
            if response.status_code == 204:
                return {"success": True}
            return {"success": False, "errors": response.json(), "http_code": response.status_code}
        except requests.RequestException as e:
            self.logger.error(f"Request error: {str(e)}")
            return {"success": False, "error": str(e)}

# Usage
api = AzanWholesaleAPIAdvanced(api_key="your-api-key")
products = [
    {"product_id": "123", "supplier": "Azan", "name": "TV", "sku": "TV-001", "mrp_price": 89999.99, "wholesale_price": 75000.00},
    {"product_id": "124", "supplier": "Azan", "name": "Phone", "sku": "PH-001", "mrp_price": 49999.99, "wholesale_price": 40000.00}
]
results = api.batch_update_products(products)
print(json.dumps(results, indent=2))
```

---

## Error Handling

### Common Error Scenarios

**Scenario 1: Invalid Product ID**
```json
Request: {"product_id": "999", ...}
Response (400): {"errors": {"product_id": ["product_id must exist in system"]}}
```

**Scenario 2: Negative Price**
```json
Request: {"mrp_price": -100, ...}
Response (400): {"errors": {"mrp_price": ["mrp_price cannot be negative"]}}
```

**Scenario 3: Missing API Key**
```
Request: (no X-API-Key header)
Response (401): {"error": "Invalid or missing API key"}
```

---

## Rate Limiting & Performance

**Rate Limiting Policy:**
- Limit: 10,000 requests per hour
- Recommended: 100-200ms between requests
- Burst: Maximum 5 requests per second

**Performance Metrics:**
- Average response time: 100-200ms
- P95 response time: 500ms
- P99 response time: 1000ms

---

## Support

**Contact Information:**
- Email: info@othoba.com

**Hours of Operation:**
- Email: 24/7
- Phone: 9:00 AM - 6:00 PM UTC+6 (Business Days)
