**Cross-Site Request Forgery (CSRF)** is a type of attack where a malicious user tricks a victim into making unintended requests to a web application where the victim is authenticated. This allows the attacker to perform actions on behalf of the victim without their consent, typically using the victim’s session or credentials.

### Examples of CSRF Attacks

1. **Changing Account Settings**:
   Imagine a user is logged into their banking site, and the site uses a simple URL to change their email address:
   ```
   https://bank.com/change-email?email=attacker@evil.com
   ```
   If the user is tricked into clicking a malicious link (e.g., by an attacker embedding it in a webpage), the attacker's email address could be set as the victim’s email, allowing the attacker to gain control of the victim's account.

2. **Performing a Money Transfer**:
   A user is logged into their online banking application, which requires a POST request to transfer money:
   ```html
   <form action="https://bank.com/transfer" method="POST">
       <input type="hidden" name="amount" value="1000">
       <input type="hidden" name="recipient" value="attacker_account">
       <input type="submit" value="Submit">
   </form>
   ```
   An attacker could craft a hidden form on their own website that automatically submits the form when the victim visits it, transferring funds from the victim’s account to the attacker's account.

3. **Changing Passwords**:
   If a victim is logged in to a service (like a social network or an email account), an attacker could embed a malicious script in a forum or blog post that makes a request to change the user’s password:
   ```javascript
   document.location = 'https://example.com/change-password?newPassword=attackerpassword';
   ```
   This would change the password of the user’s account without them realizing it.

---

### How to Prevent CSRF

There are several ways to protect against CSRF attacks:

4. **Use Anti-CSRF Tokens**:
   The most common way to prevent CSRF is to include a unique token in each form or request. The server generates a token when a session is started and requires it to be sent along with every form submission or state-changing request (e.g., POST, PUT, DELETE). 

   Example (for a login or form submission):
   - The server generates a token and embeds it within a form:
     ```html
     <input type="hidden" name="csrf_token" value="uniqueToken12345">
     ```
   - The server checks the token when the form is submitted to ensure it is valid and matches the one generated for the session.

5. **SameSite Cookies**:
   The `SameSite` cookie attribute can be used to control how cookies are sent with cross-site requests. By setting the `SameSite` attribute to `Strict` or `Lax`, the browser will only send cookies when the request originates from the same site.
```http
   Set-Cookie: sessionid=abcd1234; SameSite=Strict
```
   This prevents cookies from being sent with cross-site requests, reducing the risk of CSRF.

6. **Check Referrer or Origin Headers**:
   You can validate the `Referer` or `Origin` HTTP headers on sensitive requests to ensure that they originate from your site.
   - For example, when a user submits a form, check the `Referer` header to ensure the request came from your domain:
     ```python
     if request.headers['Referer'] != 'https://your-site.com':
         abort(403)  # Forbidden
     ```

7. **Use Custom HTTP Headers**:
   Use custom headers (such as `X-CSRF-Token`) for AJAX requests to prevent them from being sent automatically by the browser with cross-origin requests. This requires the attacker to manually set the header, which they cannot do unless they know the token.
   - Example of setting an `X-CSRF-Token` header with JavaScript:
     ```javascript
     fetch('https://your-site.com/update', {
       method: 'POST',
       headers: {
         'X-CSRF-Token': csrfToken
       },
       body: JSON.stringify({ data: 'test' })
     });
     ```

8. **Use Multi-Factor Authentication (MFA)**:
   While MFA doesn't directly prevent CSRF attacks, it can help mitigate the risk by adding an extra layer of security. Even if an attacker manages to perform a CSRF attack, they would still need the second factor (like a code sent to the victim's phone) to complete a sensitive action.

9. **Validate and Sanitize Inputs**:
   Ensure that inputs received from users are validated and sanitized. This helps prevent attackers from injecting harmful payloads into requests, reducing the potential attack surface.

### Summary
To prevent CSRF attacks:
- **Use Anti-CSRF tokens** in forms and sensitive requests.
- **Implement SameSite cookies** to prevent cookies from being sent cross-site.
- **Validate Referer or Origin headers** to check the request’s origin.
- Use **custom HTTP headers** for AJAX requests to protect against cross-origin attacks.
- **Multi-factor authentication (MFA)** can further help mitigate risks.
- **Input validation and sanitization** are key to avoiding other forms of injection attacks that can work alongside CSRF.

By combining these measures, you can significantly reduce the risk of CSRF attacks on your website or web application.