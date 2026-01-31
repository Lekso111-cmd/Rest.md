API Base Information
Base URL: https://api.library.example.com/v1

Content-Type: application/json

Authentication: Bearer Token (JWT)

Rate Limiting: 100 requests/minute per user

API Versioning: URL path versioning (/v1/)

Authentication Endpoints
1. Authentication & Token Management
POST /auth/login

Description: Authenticate user and receive JWT token

Request Body:

json
{
  "username": "john.doe",
  "password": "securePassword123"
}
Response (200 OK):

json
{
  "access_token": "eyJhbGciOiJIUzI1NiIs...",
  "token_type": "bearer",
  "expires_in": 3600,
  "refresh_token": "dGhpcyBpcyBhIHJlZnJlc2ggdG9rZW4"
}
Status Codes:

200: Authentication successful

400: Invalid request format

401: Invalid credentials

429: Too many attempts

POST /auth/refresh

Description: Refresh expired access token

Headers: Authorization: Bearer {refresh_token}

Response: Same as login

POST /auth/logout

Description: Invalidate current token

Headers: Authorization: Bearer {access_token}

Response: 204 No Content

Book Management Endpoints
2. Book Collection
GET /books

Description: Retrieve paginated list of books with filtering

Query Parameters:

page_size: Integer (10-100, default: 20)

cursor: String (for pagination)

title: String (partial match)

author: String (author name)

category: String (category name)

isbn: String (exact match)

status: String (AVAILABLE, BORROWED, etc.)

year_from, year_to: Integer (publication year range)

sort_by: String (title, year, rating)

sort_order: String (asc, desc)

Response (200 OK):

json
{
  "data": [
    {
      "id": "123e4567-e89b-12d3-a456-426614174000",
      "title": "The Great Gatsby",
      "isbn": "9780743273565",
      "authors": ["F. Scott Fitzgerald"],
      "categories": ["Fiction", "Classic"],
      "publication_year": 1925,
      "available_copies": 3,
      "total_copies": 5,
      "status": "AVAILABLE",
      "_links": {
        "self": "/v1/books/123e4567-e89b-12d3-a456-426614174000",
        "borrow": "/v1/books/123e4567-e89b-12d3-a456-426614174000/borrow",
        "reserve": "/v1/books/123e4567-e89b-12d3-a456-426614174000/reserve"
      }
    }
  ],
  "pagination": {
    "next_cursor": "eyJpZCI6IjEyM2U0NTY3LWU4OWItMTJkMy1hNDU2LTQyNjYxNDE3NDAwMCJ9",
    "has_more": true,
    "total_count": 1245
  }
}
Status Codes: 200, 400, 401, 403

POST /books

Description: Create a new book (Admin/Librarian only)

Request Body:

json
{
  "title": "New Book Title",
  "isbn": "9781234567890",
  "description": "Book description here",
  "publication_year": 2023,
  "publisher": "Publisher Name",
  "language": "English",
  "page_count": 300,
  "total_copies": 5,
  "author_ids": ["author-uuid-1", "author-uuid-2"],
  "category_ids": ["category-uuid-1", "category-uuid-2"]
}
Response: 201 Created with Location header

Status Codes: 201, 400, 401, 403, 409 (ISBN conflict)

3. Individual Book
GET /books/{id}

Description: Get detailed book information

Response (200 OK):

json
{
  "id": "123e4567-e89b-12d3-a456-426614174000",
  "title": "The Great Gatsby",
  "isbn": "9780743273565",
  "description": "A novel about the American Dream...",
  "authors": [
    {
      "id": "author-uuid-1",
      "full_name": "F. Scott Fitzgerald",
      "_links": {"self": "/v1/authors/author-uuid-1"}
    }
  ],
  "categories": [
    {"id": "cat-uuid-1", "name": "Fiction"},
    {"id": "cat-uuid-2", "name": "Classic"}
  ],
  "publication_year": 1925,
  "publisher": "Charles Scribner's Sons",
  "language": "English",
  "page_count": 180,
  "total_copies": 5,
  "available_copies": 3,
  "borrowing_history": {
    "total_borrows": 124,
    "currently_borrowed": 2
  },
  "created_at": "2023-01-15T10:30:00Z",
  "updated_at": "2023-10-20T14:45:00Z",
  "_links": {
    "self": "/v1/books/123e4567-e89b-12d3-a456-426614174000",
    "borrow": "/v1/books/123e4567-e89b-12d3-a456-426614174000/borrow",
    "reserve": "/v1/books/123e4567-e89b-12d3-a456-426614174000/reserve",
    "similar": "/v1/books?category=Fiction&sort_by=popularity"
  }
}
PUT /books/{id}

Description: Update book information (Admin/Librarian)

Status Codes: 200, 400, 401, 403, 404

DELETE /books/{id}

Description: Delete book (Admin only, soft delete)

Status Codes: 204, 401, 403, 404, 409 (if borrowed)

4. Book Actions
POST /books/{id}/borrow

Description: Borrow a book

Request Body:

json
{
  "user_id": "user-uuid",
  "expected_return_date": "2024-01-15"
}
Response: 201 Created with borrowing record

Status Codes: 201, 400, 401, 403, 404, 409 (unavailable)

POST /books/{id}/return

Description: Return a borrowed book

Request Body:

json
{
  "borrowing_id": "borrow-uuid",
  "condition": "GOOD"
}
Status Codes: 200, 400, 401, 403, 404

POST /books/{id}/reserve

Description: Reserve an unavailable book

Status Codes: 201, 400, 401, 403, 404, 409

Author Management Endpoints
5. Author Collection
GET /authors

Description: List authors with pagination

Query Parameters: name, nationality, page_size, cursor

Response: Paginated list of authors

POST /authors

Description: Create new author (Admin/Librarian)

Request Body:

json
{
  "first_name": "Ernest",
  "last_name": "Hemingway",
  "birth_date": "1899-07-21",
  "death_date": "1961-07-02",
  "biography": "American novelist and short story writer...",
  "nationality": "American"
}


6. Individual Author
GET /authors/{id}

Description: Get author details with books

Response: Author details with embedded book list

PUT /authors/{id}, DELETE /authors/{id}

Description: Update/delete author

User Management Endpoints
7. User Collection
GET /users (Admin only)

Description: List users with pagination

Query Parameters: role, status, email, name

POST /users/register

Description: Register new user (public endpoint)

Request Body:

json
{
  "email": "user@example.com",
  "username": "johndoe",
  "first_name": "John",
  "last_name": "Doe",
  "password": "SecurePass123!",
  "password_confirmation": "SecurePass123!"
}


8. Individual User
GET /users/{id}

Description: Get user profile (self or Admin)

Response: User details with borrowing history

PUT /users/{id}, PATCH /users/{id}

Description: Update user profile

GET /users/{id}/borrowings

Description: Get user's borrowing history

Query Parameters: status, from_date, to_date

Borrowing Management Endpoints
9. Borrowing Records
GET /borrowings

Description: List borrowing records (Admin/Librarian)

Query Parameters: status, user_id, book_id, overdue_only

GET /borrowings/{id}

Description: Get borrowing record details

POST /borrowings/{id}/renew

Description: Renew borrowing period

Status Codes: 200, 400, 401, 403, 404, 409 (max renewals)

Category Management Endpoints
10. Category Hierarchy
GET /categories

Description: Get category tree

Response:

json
[
  {
    "id": "cat-uuid-1",
    "name": "Fiction",
    "description": "Fictional works",
    "subcategories": [
      {"id": "cat-uuid-2", "name": "Science Fiction"},
      {"id": "cat-uuid-3", "name": "Fantasy"}
    ],
    "_links": {"books": "/v1/books?category=Fiction"}
  }
]
Search Endpoint
11. Unified Search
GET /search

Description: Global search across books, authors, categories

Query Parameters: q (search term), type (books/authors/all)

Response: Aggregated search results

Reporting Endpoints
12. Reports
GET /reports/popular-books

Description: Most borrowed books report

Query Parameters: timeframe (week/month/year/all)

GET /reports/overdue-books

Description: List overdue books with user details

GET /reports/user-activity

Description: User borrowing statistics

