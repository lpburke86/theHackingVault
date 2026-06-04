### _Week 06 — MITM, Proxying, Tunneling, Traffic Manipulation_

**Estimated Time:** 90–120 minutes  
**Difficulty:** Intermediate → Advanced  
**Tools:** Burp Suite Community Edition, Firefox/Chromium, DVWA/Mutillidae, SSH (for later chaining), local lab environment

---

## **1. Purpose of This Lab**

This lab teaches students how to **manipulate live web traffic** using Burp Suite.  
By the end, students will be able to:

- Intercept and modify HTTP/HTTPS requests in‑flight
- Rewrite headers, parameters, and HTTP methods
- Break assumptions in server‑side logic
- Manipulate cookies and session tokens
- Understand how servers react to malformed or unexpected input
- Use Repeater for controlled, iterative testing# **Lab 6.1 — Burp Suite Request/Response Manipulation**

### _Week 06 — MITM, Proxying, Tunneling, Traffic Manipulation_

**Estimated Time:** 90–120 minutes  
**Difficulty:** Intermediate → Advanced  
**Tools:** Burp Suite Community Edition, Firefox/Chromium, DVWA/Mutillidae, SSH (for later chaining), local lab environment

---

## **1. Purpose of This Lab**

This lab teaches students how to **manipulate live web traffic** using Burp Suite.  
Up to this point, students have only _observed_ traffic.  
This is the first time they will:

- Intercept requests in‑flight
- Modify headers, parameters, and methods
- Break assumptions in the application
- Force unexpected server behavior
- Understand how servers react to malformed or altered data
- Use Burp as an active traffic‑shaping tool

This lab is foundational for Weeks 7–8, where these manipulations become full exploitation techniques.

---

## **2. Pre‑Lab Requirements**

Students must have completed:

- Lab 3.1 (interception basics)
- Lab 3.2 (TLS interception)
- Lab 3.3 (Host headers & DNS)
- Lab 3.4 (application mapping)

Browser proxy must be configured to Burp.

---

## **3. Lab Tasks**

---

# **Task 1 — Intercept and Modify a Simple GET Request**

### **Objective**

Understand how modifying a request affects server behavior.

### **Steps**

1. Turn **Intercept ON**.
2. Visit a simple page in DVWA, e.g.:
    
    ```
    http://dvwa.local/vulnerabilities/sqli/
    ```
    
3. When the request appears in Burp, modify:
    - User‑Agent
    - Accept headers
    - Query parameters
4. Forward the request.

### **What to Observe**

- Changing the User‑Agent may alter server responses.
- Removing Accept headers may cause fallback behavior.
- Altering parameters may break or bypass logic.

### **Deliverable**

A short note describing how the server responded to each modification.

---

# **Task 2 — Change HTTP Methods (GET → POST, POST → GET)**

### **Objective**

Test how the application handles unexpected HTTP methods.

### **Steps**

1. Navigate to a form submission page.
2. Intercept the POST request.
3. Change:
    
    ```
    POST /path → GET /path
    ```
    
4. Forward the request.
5. Repeat in reverse:
    - Change GET → POST
    - Add a dummy body

### **What to Observe**

- Some servers reject invalid method changes.
- Some servers ignore the method entirely.
- Some servers behave unpredictably.

### **Deliverable**

A table documenting method changes and server responses.

---

# **Task 3 — Remove Required Parameters**

### **Objective**

Identify how the server handles missing or malformed input.

### **Steps**

1. Intercept a request with multiple parameters.
2. Remove one parameter entirely.
3. Remove all parameters.
4. Replace parameters with empty values.
5. Forward each variation.

### **What to Observe**

- Does the server return an error?
- Does it use default values?
- Does it expose debugging information?
- Does it behave insecurely?

### **Deliverable**

A list of at least **3 interesting behaviors** caused by missing parameters.

---

# **Task 4 — Modify Cookies and Session Tokens**

### **Objective**

Understand how session state is enforced.

### **Steps**

1. Log in to DVWA.
2. Intercept a request containing the session cookie.
3. Modify the cookie:
    - Remove it
    - Change one character
    - Replace it with a random string
4. Forward each variation.

### **What to Observe**

- Does the server invalidate the session?
- Does it create a new session?
- Does it crash or error?
- Does it reveal session handling weaknesses?

### **Deliverable**

A short explanation of how the application handles invalid session tokens.

---

# **Task 5 — Inject Unexpected Headers**

### **Objective**

Test how the server reacts to additional or conflicting headers.

### **Steps**

1. Intercept any request.
2. Add headers such as:
    
    ```
    X-Forwarded-For: 127.0.0.1
    X-Original-URL: /admin
    X-HTTP-Method-Override: DELETE
    ```
    
3. Forward the request.

### **What to Observe**

- Some frameworks trust X‑Forwarded‑For for IP logic.
- Some frameworks allow URL override.
- Some frameworks allow method override.

### **Deliverable**

A list of headers that produced meaningful changes in behavior.

---

# **Task 6 — Use Repeater for Controlled Manipulation**

### **Objective**

Perform systematic testing of request variations.

### **Steps**

1. Send a request to **Repeater**.
2. Create multiple tabs for variations:
    - Different parameters
    - Different headers
    - Different methods
    - Different cookies
3. Compare responses side‑by‑side.

### **What to Observe**

- Response codes
- Response lengths
- Error messages
- Behavioral differences

### **Deliverable**

A structured comparison of at least **5 manipulated requests**.

---

# **Task 7 — Identify Server‑Side Validation Weaknesses**

### **Objective**

Determine whether the application relies on client‑side validation.

### **Steps**

1. Find a form with client‑side validation (e.g., required fields).
2. Submit the form normally.
3. Intercept the request.
4. Remove or alter the validated fields.
5. Forward the request.

### **What to Observe**

- Does the server enforce validation?
- Does it accept invalid data?
- Does it reveal internal errors?

### **Deliverable**

A short write‑up describing whether the application enforces server‑side validation.

---

## **4. Completion Criteria**

A student has successfully completed Lab 6.1 when they can:

- Intercept and modify live traffic
- Change HTTP methods and observe effects
- Remove or alter parameters to test server behavior
- Manipulate cookies and session tokens
- Inject custom headers
- Use Repeater for controlled testing
- Identify server‑side validation weaknesses

---

## **5. Next Steps**

When you’re ready, say:

**“Next lab.”**

And I’ll generate **Lab 6.2 — Proxy Chaining Through SSH Tunnels** in the same course style.