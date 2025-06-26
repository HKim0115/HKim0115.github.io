---
categories : [Pre Security]
---

## How Websites Work

When you visit a website, your browser (like Chrome or Safari) sends a request to a web server. The server responds with the requested data, which your browser uses to render the web page.

Websites are made of two main parts:

- **Front End (Client-Side):** The visual part users see and interact with.
- **Back End (Server-Side):** The behind-the-scenes processing that handles requests and returns responses.

While there are many steps in this process, the basic idea is that your browser requests a page, and the server sends back the data needed to display it.

## Technologies Used to Build Websites

Websites are mainly built using:

- **HTML (HyperText Markup Language):** Defines the structure of the web page.
- **CSS (Cascading Style Sheets):** Adds styling and visual presentation.
- **JavaScript:** Adds interactivity and dynamic behavior.

## Basic HTML Structure

<!DOCTYPE html>
<html>
  <head>
    <title>Page Title</title>
  </head>
  <body>
    <h1>Heading</h1>
    <p>This is a paragraph.</p>
  </body>
</html>

## Key HTML Components
<!DOCTYPE html>: Declares that the document is HTML5, helping browsers interpret the page consistently.

<html>: The root element that wraps all content.

<head>: Contains metadata like the page title and linked styles/scripts.

<body>: Contains content that appears on the webpage.

<h1>: Defines a main heading.

<p>: Defines a paragraph of text.

There are many other elements like:

<button> for clickable buttons

<img> for images

<ul> / <li> for lists

Attributes in HTML
HTML elements can include attributes to control their behavior or appearance:

class: Used to assign the same style to multiple elements.
<p class="highlight">This is styled text.</p>

id: A unique identifier for a single element.
<p id="intro">Welcome!</p>

src: Specifies the source of an image or external file.
<img src="image.jpg" alt="A sample image">

Elements can have multiple attributes:
<p id="message" class="info">Hello!</p>


## Basic Web Security Concepts
Sensitive Data Exposure
Sensitive data can be unintentionally exposed if developers forget to remove it from the frontend code. This can include:

Test or temporary login credentials

Links to restricted or admin pages

API keys or other secrets

Because users can view a site's source code, exposed information can be discovered and used by attackers. Always review the page source when testing a site for vulnerabilities.

## HTML Injection
HTML injection occurs when a website includes user input in its page output without properly sanitizing it. If a user submits HTML or JavaScript and the site displays it directly, it can change the page’s structure or behavior.

The browser will render it as a heading, which could be used to trick users or execute scripts.

Input Sanitization
To prevent HTML injection, all user inputs must be sanitized—removing or escaping potentially dangerous code.

A general rule: Never trust user input. Always validate and clean it before using it in the page or in any server-side logic.

## Summary
Websites are built using HTML, CSS, and JavaScript and function by exchanging requests and responses between browsers and servers.

Important points:

HTML defines structure.

CSS handles styling.

JavaScript enables interactivity.

From a security perspective:

Always check the source code for exposed information.

Validate and sanitize all user input to prevent injection attacks.

