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

The system consists of the following components:

### User

The user submits long URLs and accesses shortened URLs.

### Application/API Server

The API server receives requests from users and coordinates the URL shortening and redirection processes.

### URL Shortener Service

The core service responsible for:

* Generating hash values.
* Creating shortened URLs.
* Handling URL redirection requests.

### Database

Stores mappings in the format:

```text
{ short_code : long_url }
```

The database is used both for storing new mappings and retrieving existing mappings during redirection.

---

# Method of Approach

There were two primary approaches considered for generating short URLs:

## Option 1: Random String Generation

Random alphanumeric strings are generated and assigned as short URLs.

### Advantages

* Easy to implement.
* Faster URL generation.
* No hashing computation required.

### Disadvantages

* Possibility of generating duplicate short URLs.
* Requires additional validation checks.
* Generates a new short URL every time even for the same original URL.
* Can increase storage requirements due to duplicate mappings.

---

## Option 2: Hash-Based Generation (Selected)

A cryptographic hash function such as SHA-256 is applied to the original URL, and a subset of the generated hash is used as the short code.

### Advantages

* Deterministic behavior.
* Same URL consistently generates the same short code.
* Easy to implement.
* Lower storage overhead for duplicate URL requests.
* Suitable for learning-oriented implementation.

### Disadvantages

* Possibility of hash collisions.
* Requires collision handling mechanisms.

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
