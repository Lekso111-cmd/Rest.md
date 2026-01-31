Richardson Maturity Model Implementation
Level 3: HATEOAS (Hypermedia as the Engine of Application State)
All API responses include _links object with relevant actions:

json
{
  "_links": {
    "self": { "href": "/v1/books/123" },
    "borrow": { 
      "href": "/v1/books/123/borrow",
      "method": "POST",
      "condition": "available_copies > 0"
    },
    "reserve": {
      "href": "/v1/books/123/reserve", 
      "method": "POST",
      "condition": "available_copies == 0"
    },
    "similar": {
      "href": "/v1/books?category=Fiction&author=Hemingway"
    }
  }
}
Link Relations Used:
self: Resource itself

collection: Parent collection

create: Create new resource

edit: Update resource

delete: Delete resource

borrow, return, renew: Book actions

next, prev, first, last: Pagination

search: Search endpoint

author, categories: Related resources

Authentication & Authorization Details
JWT Token Structure:
json
// Header
{
  "alg": "HS256",
  "typ": "JWT"
}

// Payload
{
  "sub": "user-uuid",
  "username": "johndoe",
  "role": "MEMBER",
  "permissions": ["books:read", "books:borrow"],
  "iat": 1672531200,
  "exp": 1672534800,
  "iss": "library-api"
}

// Signature: HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
Permission Matrix:
Role	Books (Read)	Books (Write)	Users (Read)	Users (Write)	Borrow
ADMIN	✓	✓	✓	✓	✓
LIBRARIAN	✓	✓	✓	✗	✓
MEMBER	✓	✗	Self only	Self only	✓
GUEST	Limited	✗	✗	✗	✗
Authentication Errors:
401 Unauthorized: Missing/invalid token

403 Forbidden: Insufficient permissions

429 Too Many Requests: Rate limit exceeded

419 Token Expired: Refresh token expired

Pagination Implementation
Cursor-based Pagination:
json
{
  "data": [...],
  "pagination": {
    "next_cursor": "base64_encoded_last_id",
    "prev_cursor": "base64_encoded_first_id",
    "has_more": true,
    "total_count": 1250,
    "page_size": 20
  }
}
Implementation:

sql
-- Example query for next page
SELECT * FROM books 
WHERE id > :cursor 
ORDER BY id 
LIMIT :page_size;
Pagination Headers:
X-Pagination-Total-Count: Total records

X-Pagination-Page-Size: Records per page

Link: RFC 5988 compliant pagination links

Caching Strategy
Cache-Control Headers:
Public Resources (Books, Authors, Categories):

text
Cache-Control: public, max-age=300, stale-while-revalidate=60
User-specific Resources (Borrowings, Reservations):

text
Cache-Control: private, max-age=60, no-cache
Dynamic Resources (Search results):

text
Cache-Control: no-cache, max-age=0, must-revalidate
ETag Implementation:
json
{
  "data": {...},
  "etag": "\"a1b2c3d4e5f6\"",
  "last_modified": "2023-12-15T10:30:00Z"
}
Redis Cache Structure:
javascript
// Book cache key
`book:{id}:v{version}`

// Author with books
`author:{id}:books:v{version}`

// Search results
`search:{query_hash}:page:{page}:size:{size}`
Cache Invalidation Strategy:
Write-through for frequently read data

Time-based expiration for lists

Manual invalidation on data changes

Version tags for cache busting

Error Handling
Standard Error Response:
json
{
  "error": {
    "code": "BOOK_NOT_FOUND",
    "message": "The requested book was not found",
    "details": {
      "book_id": "invalid-uuid"
    },
    "timestamp": "2023-12-15T10:30:00Z",
    "request_id": "req-123456",
    "documentation_url": "https://docs.api.library.example.com/errors/BOOK_NOT_FOUND"
  },
  "_links": {
    "home": "/v1",
    "books": "/v1/books"
  }
}
Common Error Codes:
VALIDATION_ERROR: Request validation failed

RESOURCE_NOT_FOUND: Requested resource doesn't exist

DUPLICATE_RESOURCE: Resource already exists (e.g., ISBN)

INSUFFICIENT_PERMISSIONS: User lacks required permissions

RESOURCE_UNAVAILABLE: Book not available for borrowing

QUOTA_EXCEEDED: User borrowing limit reached

CONFLICT_STATE: Resource in conflicting state

Request/Response Examples
Complete Borrow Flow:
1. Search for books:

http
GET /v1/books?title=great&category=Fiction&page_size=10
2. Get book details:

http
GET /v1/books/123e4567-e89b-12d3-a456-426614174000
3. Borrow book:

http
POST /v1/books/123e4567-e89b-12d3-a456-426614174000/borrow
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
Content-Type: application/json

{
  "expected_return_date": "2024-01-20"
}
Response (201 Created):

json
{
  "id": "borrow-uuid-123",
  "book_id": "123e4567-e89b-12d3-a456-426614174000",
  "user_id": "user-uuid-456",
  "borrow_date": "2023-12-15",
  "due_date": "2024-01-05",
  "expected_return_date": "2024-01-20",
  "status": "ACTIVE",
  "renewal_count": 0,
  "_links": {
    "self": "/v1/borrowings/borrow-uuid-123",
    "book": "/v1/books/123e4567-e89b-12d3-a456-426614174000",
    "user": "/v1/users/user-uuid-456",
    "return": {
      "href": "/v1/borrowings/borrow-uuid-123/return",
      "method": "POST"
    },
    "renew": {
      "href": "/v1/borrowings/borrow-uuid-123/renew",
      "method": "POST",
      "condition": "renewal_count < 2"
    }
  }
}
Rate Limiting
Rate Limit Headers:
text
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1672531260
Retry-After: 60
Buckets:
Default: 100 requests/minute

Search: 30 requests/minute

Authentication: 5 requests/minute

Admin endpoints: 500 requests/minute

Monitoring & Metrics
Health Check:
http
GET /health
json
{
  "status": "healthy",
  "timestamp": "2023-12-15T10:30:00Z",
  "services": {
    "database": {"status": "up", "latency": "12ms"},
    "cache": {"status": "up", "memory_used": "45%"},
    "search_index": {"status": "up"}
  }
}
Metrics Endpoint (Admin only):
http
GET /metrics
Requests per endpoint

Average response time

Error rates

Cache hit ratio

Active users

Webhook Support
Event Types:
book.borrowed

book.returned

book.overdue

reservation.created

reservation.expired

user.registered

Webhook Payload:
json
{
  "event": "book.borrowed",
  "timestamp": "2023-12-15T10:30:00Z",
  "data": {
    "borrowing_id": "borrow-uuid",
    "book_id": "book-uuid",
    "user_id": "user-uuid",
    "due_date": "2024-01-05"
  },
  "links": {
    "borrowing": "/v1/borrowings/borrow-uuid"
  }
}
API Versioning Strategy
URL Path Versioning: /v1/books

Content Negotiation: Accept: application/vnd.library.v1+json

Deprecation Headers:

text
Deprecation: true
Sunset: Tue, 31 Dec 2024 23:59:59 GMT
Link: </v2/books>; rel="successor-version"
Security Headers
text
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Content-Security-Policy: default-src 'self'
X-XSS-Protection: 1; mode=block
