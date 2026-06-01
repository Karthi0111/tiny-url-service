Objective:
	To design a URL shortening service that converts long URLs into shorter URLs and redirects users back to the original URL.

Problem Statement:
	Design a Tiny URL service, which converts a long URL to shorter URL and redirects to the original URL.

Requirements Analysis:
•	Accept a long URL from the user
•	Generate a unique short URL.
•	Store the URL mapping in the database
•	Redirect users to the original URL
•	Handle duplicate requests

Proposed Solution:
•	The proposed solution follows a client server architecture.
•	When a user submits a long URL, the system validates the URL and generates a hash using the SHA-256 hashing algorithm.
•	A portion of the generated hash is extracted and used as the short code.
•	The short code and original URL are then stored in the database.
•	Whenever a user accesses the short URL, the system retrieves the corresponding original URL from the database and redirects the user accordingly.
 

Method of approach:
•	I had to options to choose how I could generate the tiny URL.
•	The one was random generating and the other is Hash – based generation.
•	I chose hash – based generation because of creating unique short URL for each long URLs.
•	How the random strings generated?
o	It generated based on random alphanumeric strings which creates a short URL.
•	The random string generator is easy implement its way quicker than what I chose, but it has a drawback that it may create a same URL for another long URL at one point.
•	Another drawback is that for every time I want to create a short URL, it make a new one which makes the use of more storage it quite difficult for real world situation.
•	So, I chose the hash – based generation because at every time it creates a unique URL based on the hash value.
•	Its, very a straight forward implementation considered to that of random generator.
•	How this Hash URLs are made?
o	A cryptographic hash function such as SHA-256 is applied to the original URL, and a subset of the generated hash is used as the short code.
•	It has one drawback there’s a possibility of hash collisions but I considered using hash collision mechanisms.


Collison handling mechanism:
To ensure uniqueness of generated short codes, the following procedure will be followed:
•	Generate SHA-256 hash of the URL.
•	Extract the first few characters as the short code.
•	Check whether the generated code already exists.
•	If a collision is detected:
o	Use additional hash characters.
o	Append a unique suffix if necessary.
•	Repeat until we get a unique shortened URL.
This strategy ensures reliable URL generation without comprising simplicity.

Justification for choosing Hashing:
•	Hashing has been selected as the preferred approach for the initial version of the system due to its simplicity, predictability, and ease of implementation.
•	The approach is intuitive and suitable for learning – oriented implementation. 
•	It enables the same URL to consistently generate the same short code while maintaining relatively low implementation complexity.
•	Although collisions are theoretically possible, they can be effectively managed through validation and collision-handling mechanisms.

API DESIGN:
Create a short URL
Endpoint
POST /shorten
Request
{
  "url": “https://www.amazon.in/mobile/samsung-galaxy-s24-ultra-5g-smartphone"
}
Response
{
    "short_url": "http://localhost:5000/7d7930"
}

Redirected URL:
•	Endpoint
                  GET /7d7930
•	Response
                  302 Redirect
The user is redirected to the whole corresponding original URL

Design principles followed
Single responsibility principle
Each component performs a single well-defined responsibility.
•	API layer handles requests.
•	Hash generator creates short codes.
•	Database layer stores mappings.
•	Redirect service handles URL redirection.

Conclusion
The proposed Tiny URL service provides an efficient mechanism for converting lengthy URLs into compact and shareable links. The design follows established software engineering principles, including modularity, separation of concerns, and maintainability.
A hash-based URL generation strategy has been selected due to its simplicity and ease of implementation. The architecture has been designed to support future scalability and enhancements, ensuring that the system can evolve to meet increasing user demands while maintaining reliability and performance.
This version looks like something a final-year student would submit to a mentor before moving to implementation. It is professional enough for review while still sounding like a genuine student design rather than an advanced distributed-systems document.
