# Design of Tiny URL Service

## Objective

To design a URL shortening service that converts long URLs into shorter URLs and redirects users back to the original URL.

---

# Problem Statement

Design a Tiny URL service that converts a long URL into a shorter URL and redirects users to the original URL whenever the shortened URL is accessed.

---

# Requirements Analysis

The proposed system shall satisfy the following requirements:

* Accept a long URL from the user.
* Generate a unique short URL.
* Store the URL mapping in the database.
* Redirect users to the original URL.
* Handle duplicate URL requests efficiently.

---

# Proposed Solution

The proposed solution follows a client-server architecture.

* When a user submits a long URL, the system validates the URL and generates a hash using the SHA-256 hashing algorithm.
* A portion of the generated hash is extracted and used as the short code.
* The short code and original URL are then stored in the database.
* Whenever a user accesses the short URL, the system retrieves the corresponding original URL from the database and redirects the user accordingly.

---

# Flow Chart

```text
Start
   |
   v
User Inputs Long URL
   |
   v
Generate SHA-256 Hash
   |
   v
Extract Short Code
   |
   v
Collision Check
   |
   v
Store {Short Code, Long URL}
   |
   v
Return Tiny URL
   |
   v
Stop
```

---

# System Architecture Diagram

> Architecture diagram is provided in the architecture folder.

The enhanced architecture consists of the following components:

### User

The user interacts with the application through a web-based user interface to create and access shortened URLs.

### UI Layer

The UI layer provides a simple and user-friendly interface where users can:

* Enter a long URL.
* Generate a shortened URL.
* View the generated short URL.
* Access previously generated URLs.

The UI improves usability by eliminating the need for direct API interaction.

### Application/API Server

The API server receives requests from the UI layer and coordinates URL shortening and redirection operations.

### URL Shortener Service

The core service responsible for:

* Generating SHA-256 hash values.
* Creating shortened URLs.
* Handling URL redirection requests.
* Managing collision resolution.

### Redis Cache (LRU)

A Redis cache is introduced between the application and database layers.

Frequently accessed URL mappings are stored in memory, allowing faster retrieval without repeatedly querying the database.

The cache uses the Least Recently Used (LRU) eviction policy, which automatically removes the least recently accessed entries when cache capacity is reached.

Advantages of Redis Cache:

* Faster URL lookup.
* Reduced database load.
* Improved application performance.
* Better scalability for high-traffic scenarios.

### Database

Stores mappings in the format:

short_code → long_url

The database serves as the permanent source of truth for all URL mappings.

---

# Method of Approach

The Tiny URL generation process is based on SHA-256 hashing.

When a user submits a long URL, the system first validates the input URL and then applies the SHA-256 hashing algorithm to generate a unique hash value.

Since the complete SHA-256 hash is too large to be used as a short URL, only a subset of the generated hash is extracted and used as the short code.

The generated short code is then checked against existing database records to ensure uniqueness.

If a collision is detected, additional characters from the hash value are used until a unique short code is obtained.

The final short code is stored together with the original URL in the database and returned to the user as the shortened URL.

### Advantages of the Hash-Based Approach

* Deterministic behavior, where the same URL consistently generates the same short code.
* Simple and straightforward implementation.
* Easy integration with Python through built-in hashing libraries.
* Reduced storage overhead for duplicate URL requests.
* Suitable for learning-oriented and small-to-medium scale implementations.
* Can be extended in the future to support alternative URL generation strategies if required.

### Limitation

The primary limitation of this approach is the possibility of hash collisions. However, collisions are extremely rare when using SHA-256 and can be effectively handled using the collision resolution mechanism implemented in the system.

---

# Why Hashing Was Selected

Hashing was selected as the preferred approach for the initial version of the system due to its simplicity, predictability, and ease of implementation.

The same URL consistently produces the same hash value, reducing duplicate mappings and improving storage efficiency.

The approach is intuitive and straightforward to implement using Python's built-in hashing libraries.

Although collisions are theoretically possible, they can be effectively handled through collision detection and resolution mechanisms.

---

# Collision Handling Mechanism

Since the short URL is generated using only a portion of the SHA-256 hash value, there is a possibility that two different URLs may produce the same short code.

To ensure uniqueness of generated short codes, the following procedure will be followed:

1. Generate the SHA-256 hash of the URL.
2. Extract the first few characters as the short code.
3. Check whether the generated short code already exists in the database.
4. If a collision is detected:

   * Use additional hash characters.
   * Append a unique suffix if required.
5. Repeat the process until a unique short code is obtained.

This strategy ensures reliable URL generation while maintaining implementation simplicity.

---

# Database Design

## Table: URLS

| Column Name | Data Type   | Description          |
| ----------- | ----------- | -------------------- |
| id          | BIGINT      | Primary Key          |
| long_url    | TEXT        | Original URL         |
| short_code  | VARCHAR(20) | Generated Short Code |
| created_at  | TIMESTAMP   | Creation Timestamp   |

---

# Cache Design

The system incorporates Redis as an in-memory caching solution to improve URL retrieval performance.

Whenever a shortened URL is accessed, the system first checks Redis Cache.

### Cache Hit

If the URL mapping exists in Redis, the original URL is immediately returned without querying the database.

### Cache Miss

If the mapping does not exist in Redis, the database is queried. The retrieved mapping is then stored in Redis for future requests.

### Cache Eviction Policy

The cache uses the Least Recently Used (LRU) policy.

When cache capacity is reached, the least recently accessed entries are removed automatically, ensuring that frequently accessed URL mappings remain available in memory.

### Why Redis Was Selected

Redis was selected because it provides:

* Extremely fast read and write operations.
* Efficient key-value storage.
* Easy integration with Python applications.
* Support for LRU-based cache management.
* Reduced database load for frequently accessed URLs.

This design significantly improves application performance while maintaining simplicity and scalability.

---

# API Design

## Create Short URL

### Endpoint

```http
POST /shorten
```

### Request

```json
{
  "url": "https://www.amazon.in/mobile/samsung-galaxy-s24-ultra-5g-smartphone"
}
```

### Response

```json
{
  "short_url": "http://localhost:5000/7d7930"
}
```

---

## Redirect URL

### Endpoint

```http
GET /7d7930
```

### Response

```http
302 Redirect
```

The user is redirected to the corresponding original URL.

---

# Design Principles Followed

## Single Responsibility Principle (SRP)

Each component performs a single well-defined responsibility.

* API Layer handles requests and responses.
* Hash Generator creates short codes.
* Database Layer stores URL mappings.
* Redirect Service handles URL redirection.

## Separation of Concerns

Business logic, storage logic, and request handling logic are separated into independent modules, making the system easier to maintain and extend.

## Modularity

Each component can be modified independently without affecting the overall system functionality.

---

# Conclusion

The proposed Tiny URL service provides an efficient mechanism for converting lengthy URLs into compact and shareable links.

The design follows established software engineering principles, including modularity, separation of concerns, and maintainability.

A hash-based URL generation strategy has been selected due to its simplicity, deterministic behavior, and ease of implementation.

The architecture has been designed to support future scalability and enhancements, ensuring that the system can evolve to meet increasing user demands while maintaining reliability and performance.
