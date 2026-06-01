# Tiny URL Service Design Document

## Objective

To design a URL shortening service that converts long URLs into shorter URLs and redirects users back to the original URLs.

---

# 1. Problem Statement

Design a Tiny URL service that accepts a long URL, generates a shortened URL, stores the mapping between the original and shortened URLs, and redirects users to the original URL whenever the shortened URL is accessed.

Example:

Original URL:

https://www.amazon.in/mobile/samsung-galaxy-s24-ultra-5g-smartphone

Short URL:

https://tinyurl.com/7d7930

---

# 2. Functional Requirements

The system shall support the following functionalities:

1. The system shall allow users to submit a valid URL for shortening.

2. The system shall generate a shortened URL for every valid URL submitted.

3. The system shall maintain a mapping between shortened URLs and original URLs.

4. The system shall redirect users to the original URL when a valid shortened URL is accessed.

5. The system shall ensure uniqueness of shortened URLs.

6. The system shall handle duplicate URL requests efficiently.

---

# 3. Non-Functional Requirements

### Performance

The system should generate and retrieve shortened URLs with minimal latency.

### Scalability

The architecture should support an increasing number of users and URL mappings.

### Reliability

URL mappings should remain available even after server restarts.

### Maintainability

The system should follow modular design principles to facilitate future enhancements.

### Security

The application should validate user inputs and prevent malicious URL submissions.

---

# 4. Proposed Solution

The proposed solution follows a client-server architecture.

When a user submits a long URL, the system validates the URL and generates a hash using the SHA-256 hashing algorithm.

A portion of the generated hash is extracted and used as the short code.

The short code and original URL are then stored in the database.

Whenever a user accesses the short URL, the system retrieves the corresponding original URL from the database and redirects the user accordingly.

---

# 5. System Workflow

### URL Creation Workflow

1. User submits a long URL.
2. The system validates the URL.
3. SHA-256 hashing is applied to the URL.
4. A portion of the hash is selected as the short code.
5. Collision validation is performed.
6. The URL mapping is stored in the database.
7. The shortened URL is returned to the user.

### URL Redirection Workflow

1. User accesses the shortened URL.
2. The short code is extracted.
3. The database is searched for the corresponding original URL.
4. The original URL is retrieved.
5. The user is redirected to the original URL.

---

# 6. System Architecture

The architecture consists of the following components:

### User

The user submits long URLs and accesses shortened URLs.

### API Layer

Handles incoming requests and returns responses.

### Hash Generator

Generates short codes using SHA-256 hashing.

### Database Layer

Stores mappings between short codes and original URLs.

### Redirect Service

Retrieves original URLs and performs redirection.

---

# 7. Database Design

Table Name: URLS

| Column Name | Data Type   | Description          |
| ----------- | ----------- | -------------------- |
| id          | BIGINT      | Primary Key          |
| long_url    | TEXT        | Original URL         |
| short_code  | VARCHAR(20) | Generated Short Code |
| created_at  | TIMESTAMP   | Creation Timestamp   |

### Purpose of Each Field

id:
Uniquely identifies each URL record.

long_url:
Stores the original URL provided by the user.

short_code:
Stores the generated short code.

created_at:
Stores the timestamp when the URL was created.

---

# 8. URL Generation Approaches Considered

Before finalizing the implementation approach, multiple URL generation techniques were evaluated.

## Option 1: Random String Generation

Random alphanumeric strings are generated and assigned as short codes.

### Advantages

* Simple implementation
* Fast generation process

### Disadvantages

* Possibility of collisions
* Requires additional validation checks
* Generates a new code every time even for identical URLs

---

## Option 2: Hash-Based Generation (Selected)

A cryptographic hash function such as SHA-256 is applied to the original URL and a subset of the generated hash is used as the short code.

### Advantages

* Deterministic output
* Easy to implement
* Same URL consistently generates the same short code
* Low implementation complexity

### Disadvantages

* Possibility of hash collisions
* Requires collision resolution mechanisms

---

# 9. Justification for Selecting Hashing

Hashing has been selected as the preferred approach for the initial version of the system due to its simplicity, predictability, and ease of implementation.

The same URL consistently produces the same hash value, which helps reduce duplicate mappings.

The approach is intuitive and suitable for a learning-oriented implementation while maintaining good performance.

Although collisions are theoretically possible, they can be effectively handled through collision detection and resolution mechanisms.

---

# 10. Collision Handling Mechanism

Since the short URL is generated using only a portion of the SHA-256 hash value, collisions may occur when two different URLs produce the same short code.

To resolve collisions, the generated short code is first checked against existing database records.

If the short code does not already exist, the mapping is stored directly.

If the short code already exists and is associated with a different URL, additional characters from the hash value are included to generate a longer and unique short code.

This process continues until a unique short code is obtained.

As SHA-256 produces highly unique hash values, collisions are expected to be extremely rare.

---

# 11. API Design

## Create Short URL

### Endpoint

POST /shorten

### Request

```json
{
  "url": "https://www.amazon.in/mobile/samsung-galaxy-s24-ultra-5g-smartphone"
}
```

### Response

```json
{
  "short_code": "7d7930",
  "short_url": "http://localhost:5000/7d7930"
}
```

---

## Redirect URL

### Endpoint

GET /7d7930

### Response

302 Redirect

The user is redirected to the corresponding original URL.

---

# 12. Design Principles Followed

## Single Responsibility Principle (SRP)

Each component performs only one specific responsibility.

* API Layer handles requests and responses.
* Hash Generator generates short codes.
* Database Layer stores URL mappings.
* Redirect Service handles URL redirection.

## Separation of Concerns

Business logic, storage logic, and request handling logic are separated into independent modules.

## Modularity

The system is designed in a modular manner, enabling future enhancements without affecting existing functionality.

---

# 13. Security Considerations

The following security measures are considered:

* URL validation before processing.
* Input sanitization.
* Prevention of invalid URL submissions.
* Secure database access.
* HTTPS communication in production environments.

---

# 14. Future Enhancements

The following enhancements can be incorporated in future versions:

* User authentication and authorization.
* URL expiration support.
* Click analytics and reporting.
* Redis caching.
* Load balancing.
* Distributed database architecture.
* QR code generation.

---

# 15. Conclusion

The proposed Tiny URL service provides an efficient mechanism for converting lengthy URLs into compact and shareable links.

The design follows software engineering principles such as modularity, maintainability, and separation of concerns.

A hash-based URL generation strategy has been selected due to its simplicity, deterministic behavior, and ease of implementation.

The architecture is designed to support future scalability and enhancements while maintaining reliability and performance.
