**Cross-Site Scripting (XSS)** is a security vulnerability in web applications where an attacker injects malicious scripts (usually JavaScript) into webpages viewed by other users. This allows the attacker to execute arbitrary code in the context of the victim’s browser, leading to potential data theft, account hijacking, and other malicious activities.

There are **three main types** of XSS attacks:

### 1. **Stored XSS (Persistent XSS)**
   - **Description**: In stored XSS, the attacker’s malicious script is permanently stored on the server (e.g., in a database, comment section, or any persistent storage). The script is then served to users who access the page containing the malicious code.
   - **Example**: An attacker posts a comment on a website, which contains a malicious script. The comment is stored in the website’s database. Later, when a user views the comment, the script executes in their browser.
   
   **Scenario**:  
   - An attacker submits the following comment in a forum:
     ```html
     <script>alert('XSS Attack!');</script>
     ```
   - The comment is stored in the database and displayed on the page when other users visit.
   - When another user views the comment, the malicious script executes and displays an alert message on the user's screen.

   **Consequences**: The attacker could steal session cookies, redirect users to malicious sites, or perform actions on behalf of the victim.

---

### 2. **Reflected XSS (Non-Persistent XSS)**
   - **Description**: Reflected XSS occurs when the malicious script is included in the URL or request sent to the server and is reflected back in the server's response without being properly sanitized. This type of XSS is typically delivered via malicious links.
   - **Example**: An attacker sends a malicious URL to the victim, which includes a malicious script as a query parameter. When the victim clicks the link, the script executes.
   
   **Scenario**:
   - An attacker crafts a URL that looks like this:
     ```
     http://example.com/search?query=<script>alert('XSS');</script>
     ```
   - The application reflects the `query` parameter in the search results page without sanitizing it. The attacker sends this link to the victim via email or social media.
   - When the victim clicks the link, the script is executed in their browser, showing the alert message.

   **Consequences**: An attacker could steal session cookies, log keystrokes, or redirect the victim to a malicious website.

---

### 3. **DOM-based XSS**
   - **Description**: DOM-based XSS occurs when the vulnerability is in the client-side JavaScript code rather than the server-side code. The malicious script is executed when the victim interacts with the page, and it exploits insecure manipulation of the DOM (Document Object Model).
   - **Example**: The attacker exploits how JavaScript modifies the page dynamically based on input from the URL, cookies, or other user input.
   
   **Scenario**:
   - Suppose a website uses JavaScript to retrieve query parameters from the URL and display them on the page:
     ```javascript
     var userInput = window.location.hash.substring(1);  // Grabs input from URL hash
     document.getElementById('output').innerHTML = userInput;
     ```
   - If an attacker sends a link like:
     ```
     http://example.com/#<script>alert('XSS');</script>
     ```
   - The script from the URL is inserted directly into the webpage via `innerHTML`, executing the attacker’s code.
   
   **Consequences**: The attacker can execute arbitrary JavaScript on the victim’s browser, leading to possible data theft or malicious actions.

---

### Prevention Techniques

To prevent XSS attacks, developers must sanitize and validate user input and avoid directly injecting user input into web pages without proper escaping. Here are several common mitigation strategies:

1. **Output Encoding (Escaping)**:
   - Ensure that any user-supplied data is properly encoded when displayed on the webpage. This converts potentially dangerous characters like `<`, `>`, and `&` into their HTML-encoded equivalents (`&lt;`, `&gt;`, and `&amp;`).
   - For example, instead of rendering raw input, output it as a safe string:
     ```html
     <!-- Dangerous -->
     <div>user input: <script>alert('XSS');</script></div>

     <!-- Safe -->
     <div>user input: &lt;script&gt;alert('XSS');&lt;/script&gt;</div>
     ```

2. **Content Security Policy (CSP)**:
   - Implement a strong **CSP** to limit the sources from which scripts can be loaded and executed. This reduces the risk of executing unauthorized scripts.
   - Example of a CSP header:
     ```
     Content-Security-Policy: default-src 'self'; script-src 'self' https://trusted.cdn.com;
     ```

3. **Sanitize User Inputs**:
   - Always sanitize user inputs on both the client-side and server-side before rendering them on a page. Use libraries or frameworks that automatically sanitize inputs.

4. **Use HttpOnly and Secure Cookies**:
   - Set the `HttpOnly` flag on cookies to prevent JavaScript from accessing them.
   - Example:
```http
     Set-Cookie: sessionid=abcd1234; HttpOnly; Secure;
```

5. **Avoid `innerHTML`, `document.write()`, and `eval()`**:
   - Never directly insert untrusted user data using methods like `innerHTML`, `document.write()`, or `eval()` since these can lead to XSS vulnerabilities.
   - Use safer alternatives, such as `textContent` or `setAttribute()`, to safely insert data into the page.

6. **Use Safe JavaScript Libraries**:
   - Use libraries like **DOMPurify** or **OWASP Java HTML Sanitizer** to clean and sanitize user input before rendering it to prevent XSS.

7. **Validate Input Types**:
   - Enforce strict input validation rules for any user-submitted data (e.g., form fields, query parameters). Ensure that only expected input types are allowed (e.g., numbers, letters).

8. **Regular Security Audits**:
   - Continuously monitor and audit your web application for potential security vulnerabilities, including XSS.

---

### Summary Table

| **XSS Type**           | **Description**                                       | **Example**                                                 | **Preventive Measures**                                |
|------------------------|-------------------------------------------------------|-------------------------------------------------------------|--------------------------------------------------------|
| **Stored XSS**          | Malicious script stored on the server and reflected in responses to users. | Comment section or profile update that stores harmful code. | Sanitize user inputs, use output encoding.             |
| **Reflected XSS**       | Malicious script reflected in the server's response based on user input (URL, form submission). | Malicious URL with script in a query string.               | Validate inputs, use input sanitization, avoid reflecting user input in HTML directly. |
| **DOM-based XSS**       | Malicious script executed by client-side JavaScript manipulating the DOM. | URL with script injected in the fragment identifier (e.g., `#<script>alert('XSS');</script>`) | Avoid unsafe DOM manipulations, use `textContent` instead of `innerHTML`. |

By following these preventive measures and properly handling user inputs, developers can effectively mitigate the risks associated with XSS vulnerabilities.