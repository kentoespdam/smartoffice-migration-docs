# 3. Common Conventions

## 3.1 Request/Response Envelope

**Success Response:**
```json
{
  "success": true,
  "message": "Operation completed successfully",
  "data": { ... },
  "meta": {
    "page": 1,
    "size": 20,
    "totalItems": 150,
    "totalPages": 8
  }
}
```

**Error Response:**
```json
{
  "success": false,
  "message": "Human-readable error message",
  "errorCode": "MAIL_RECIPIENT_DUPLICATE",
  "errors": [
    { "field": "recipientId", "message": "Karyawan ini sudah terdaftar dalam list penerima" }
  ],
  "timestamp": "2026-03-12T06:41:52Z",
  "path": "/api/v1/mails/123/recipients"
}
```

## 3.2 HTTP Status Codes

| Code | Usage |
|---|---|
| `200` | OK — Read, Update success |
| `201` | Created — New resource created |
| `204` | No Content — Delete success |
| `400` | Bad Request — Validation error |
| `401` | Unauthorized — Token invalid/expired |
| `403` | Forbidden — Insufficient permission |
| `404` | Not Found — Resource not found |
| `409` | Conflict — Duplicate (folder name, recipient, category code) |
| `422` | Unprocessable Entity — Business rule violation |
| `500` | Internal Server Error |

## 3.3 Pagination

Query params (untuk semua list endpoints):

| Param | Type | Default | Keterangan |
|---|---|---|---|
| `page` | int | 1 | Halaman (1-indexed) |
| `size` | int | 20 | Items per halaman (max 100) |
| `sort` | string | varies | Format: `field,direction`. Contoh: `createdDate,desc` |

## 3.4 Filtering

Filter menggunakan query parameters langsung:

```
GET /v1/mails?folderId=2&typeId=1&startDate=2026-01-01&endDate=2026-03-12&keyword=rapat
```

## 3.5 Date/Time Format

- Request: ISO 8601 → `2026-03-12` (date) / `2026-03-12T06:41:52Z` (datetime)
- Response: ISO 8601 selalu dengan timezone UTC

<!-- nav -->
---
**Navigation:** [< Prev](02-authentication-security.md) | [Next >](04-entity-relationship-mapping.md)
<!-- /nav -->
