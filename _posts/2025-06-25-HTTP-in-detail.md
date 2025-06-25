---
title: "Intro to HTTP & HTTPS"
categories: [Pre Security]
layout: single
---

# Introduction to HTTP & HTTPS

## What is HTTP?

**HTTP (HyperText Transfer Protocol)** is a set of rules for transferring data (HTML, images, video, etc.) between a client (browser) and a web server. It was developed between 1989–1991 by Tim Berners-Lee's team.

## What is HTTPS?

**HTTPS (HTTP Secure)** is the encrypted version of HTTP. It provides data confidentiality and ensures you're connecting to the legitimate server, not a fake one.

---

## What is a URL?

A **URL (Uniform Resource Locator)** is the full address used to locate a resource on the internet. A URL can include the following components:

- **Scheme**: The protocol (e.g., `http`, `https`, `ftp`)
- **User Info**: Optional authentication (`user:password@`)
- **Host**: Domain name or IP address
- **Port**: Default for HTTP is 80, HTTPS is 443 (can be custom)
- **Path**: Location of the resource (e.g., `/blog`)
- **Query String**: Parameters (e.g., `?id=1`)
- **Fragment**: Internal page location (e.g., `#section3`)

Example URL:  
`http://user:pass@tryhackme.com:80/view-room?id=1#section3`

---

## Making a Request

An HTTP request typically looks like:

### Key Points:
- `GET / HTTP/1.1`: Request home page using HTTP/1.1
- `Host`: Specifies the server
- `User-Agent`: Info about the browser making the request
- `Referer`: Previous page that referred the request

---

## Common HTTP Methods

| Method | Purpose |
|--------|---------|
| `GET` | Retrieve data |
| `POST` | Submit new data |
| `PUT` | Update existing data |
| `DELETE` | Remove data |

---

## HTTP Status Code Categories

| Code Range | Meaning |
|------------|---------|
| 100–199 | Informational |
| 200–299 | Success |
| 300–399 | Redirection |
| 400–499 | Client Errors |
| 500–599 | Server Errors |

---

## Common HTTP Status Codes

- `200 OK`: Request succeeded
- `201 Created`: New resource created
- `301 Moved Permanently`: URL changed permanently
- `302 Found`: Temporary redirect
- `400 Bad Request`: Invalid request from client
- `401 Unauthorized`: Authentication required
- `403 Forbidden`: Access denied
- `404 Not Found`: Resource not found
- `405 Method Not Allowed`: Method not supported
- `500 Internal Server Error`: Server failure
- `503 Service Unavailable`: Server overloaded or down

---

## What are HTTP Headers?

HTTP headers are optional key-value pairs sent between the client and server to pass extra information during a request or response. While not strictly required, they are crucial for proper website functionality.

---

## Common **Request Headers**

| Header | Description |
|--------|-------------|
| **Host** | Specifies the target host/domain (useful for servers hosting multiple websites). |
| **User-Agent** | Identifies the browser software and version, helping servers optimize content. |
| **Content-Length** | Indicates the size of the request body (especially important for form submissions). |
| **Accept-Encoding** | Lists supported compression methods (e.g., gzip, deflate). |
| **Cookie** | Sends stored cookies back to the server. |

---

## Common **Response Headers**

| Header | Description |
|--------|-------------|
| **Set-Cookie** | Used to send cookies to the client for storage. |
| **Cache-Control** | Instructs how long the response can be cached. |
| **Content-Type** | Tells the client what kind of content is being returned (e.g., text/html, image/png). |
| **Content-Encoding** | Indicates the compression used for the response content. |

---

## What Are Cookies?

**Cookies** are small text-based data stored on the client’s browser. They help websites remember things like:

- Logged-in sessions
- User preferences
- Shopping carts

### How They Work:

1. The server sends a cookie using the `Set-Cookie` header.
2. The browser stores this cookie.
3. On future requests, the browser sends it back via the `Cookie` header.

Cookies are crucial for personalizing web experiences and maintaining sessions across visits.

---

## Summary

HTTP and HTTPS are the foundational protocols of web communication. Understanding how URLs, methods, and status codes work is essential for web development and cybersecurity. HTTPS ensures secure data exchange, while HTTP methods like GET and POST define the nature of client-server interaction.  
