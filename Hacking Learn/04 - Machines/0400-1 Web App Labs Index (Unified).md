A centralized hub for all intentionally vulnerable web applications in the lab environment.

This index links and organizes:
- DVWA
- Mutillidae
- WebGoat
- Juice Shop
- OWASP BWA (umbrella VM containing multiple apps)

Use this page to navigate, compare, and track progress across all web app labs.

---

# 🗂 Web App Overview Table

| App | Difficulty | Status | Tech Stack | Location | Notes |
|-----|------------|--------|------------|----------|--------|
| [[0403.000 DVWA]] | -{{ Easy/Medium/Hard }}- | -{{ Not Started / In Progress / Completed }}- | PHP/MySQL | -{{ URL or IP }}- | -{{ Summary }}- |
| [[0404.000 Mutillidae]] | -{{ Easy/Medium/Hard }}- | -{{ Not Started / In Progress / Completed }}- | PHP/MySQL | -{{ URL or IP }}- | -{{ Summary }}- |
| [[0405.000 WebGoat]] | -{{ Varies by lesson }}- | -{{ Not Started / In Progress / Completed }}- | Java/Tomcat | -{{ URL or IP }}- | -{{ Summary }}- |
| [[0406.000 Juice Shop]] | -{{ Easy/Medium/Hard }}- | -{{ Not Started / In Progress / Completed }}- | Node.js/Express | -{{ URL or IP }}- | -{{ Summary }}- |
| [[0409.000 OWASP BWA]] | -{{ Mixed }}- | -{{ Not Started / In Progress / Completed }}- | LAMP/Tomcat | -{{ URL or IP }}- | -{{ Contains multiple apps }}- |

---

# 🧩 Detailed Sections for Each App

Below are expandable sections for each app.  
These mirror your VulnHub and Machine Index structures.

---

## ▶ DVWA — [[0403.000 DVWA]]

### Metadata
- Difficulty: -{{ Based on DVWA security level }}-
- OS: -{{ Linux (LAMP stack) }}-
- URL: -{{ Base URL }}-
- Notes: -{{ High-level notes }}-

### Linked Pages
- Overview → [[0403.001 DVWA Overview]]
- Findings → [[0403.002 DVWA Findings]]
- Guide → [[0403.000-1 DVWA guide]]
- Workbook → [[0204.000 DVWA Workbook]]

### Recon Summary
- Open ports: -{{ nmap results }}-
- Services: -{{ Apache/PHP/MySQL }}-

### Enumeration Summary
- Directories: -{{ gobuster results }}-
- Authentication: -{{ login behavior }}-
- Input validation: -{{ injection points }}-

### Vulnerability Summary
- Critical: -{{ }}-
- High: -{{ }}-
- Medium: -{{ }}-
- Low: -{{ }}-

### Exploitation Notes
- Entry point: -{{ vuln used }}-
- Payload: -{{ technique }}-

### Flags & Credentials
- User flag: -{{ }}-
- Admin flag: -{{ }}-
- Credentials: -{{ }}-

### Attack Chain Mapping
| Step | Action | Result |
|------|--------|--------|
| 1 | -{{ }}- | -{{ }}- |
| 2 | -{{ }}- | -{{ }}- |
| 3 | -{{ }}- | -{{ }}- |

### Hardening Recommendations
- -{{ Fix #1 }}-
- -{{ Fix #2 }}-

---

## ▶ Mutillidae — [[0404.000 Mutillidae]]

### Metadata
- Difficulty: -{{ Based on configuration }}-
- OS: -{{ Linux (LAMP stack) }}-
- URL: -{{ Base URL }}-

### Linked Pages
- Overview → [[0404.001 Overview]]
- Findings → [[0404.002 Findings]]
- Guide → [[0404.000-1 Mutillidae Guide]]
- Workbook → [[0210.000 Mutillidae Workbook]]

### Recon Summary
- Open ports: -{{ nmap results }}-
- Services: -{{ Apache/PHP/MySQL }}-

### Enumeration Summary
- Directories: -{{ gobuster results }}-
- Authentication: -{{ login behavior }}-
- Input validation: -{{ injection points }}-
- DB interaction: -{{ SQL error behavior }}-

### Vulnerability Summary
- Critical: -{{ }}-
- High: -{{ }}-
- Medium: -{{ }}-
- Low: -{{ }}-

### Exploitation Notes
- Entry point: -{{ vuln used }}-
- Payload: -{{ technique }}-

### Flags & Credentials
- User flag: -{{ }}-
- Admin flag: -{{ }}-
- Credentials: -{{ }}-

### Attack Chain Mapping
| Step | Action | Result |
|------|--------|--------|
| 1 | -{{ }}- | -{{ }}- |
| 2 | -{{ }}- | -{{ }}- |
| 3 | -{{ }}- | -{{ }}- |

### Hardening Recommendations
- -{{ Fix #1 }}-
- -{{ Fix #2 }}-

---

## ▶ WebGoat — [[0405.000 WebGoat]]

### Metadata
- Difficulty: -{{ Varies by lesson }}-
- OS: -{{ Linux (Java/Tomcat) }}-
- URL: -{{ Base URL }}-

### Linked Pages
- Overview → [[0405.001 Overview]]
- Findings → [[0405.002 Findings]]
- Guide → [[0405.000-1 WebGoat guide]]
- Workbook → [[0209.000 WebGoat Workbook]]

### Recon Summary
- Open ports: -{{ nmap results }}-
- Services: -{{ Tomcat/Java }}-

### Enumeration Summary
- Lessons: -{{ list of vulnerable lessons }}-
- Authentication: -{{ login behavior }}-
- Input validation: -{{ injection points }}-

### Vulnerability Summary
- Critical: -{{ }}-
- High: -{{ }}-
- Medium: -{{ }}-
- Low: -{{ }}-

### Exploitation Notes
- Entry point: -{{ vuln used }}-
- Payload: -{{ technique }}-

### Flags & Credentials
- User flag: -{{ }}-
- Admin flag: -{{ }}-
- Credentials: -{{ }}-

### Attack Chain Mapping
| Step | Action | Result |
|------|--------|--------|
| 1 | -{{ }}- | -{{ }}- |
| 2 | -{{ }}- | -{{ }}- |
| 3 | -{{ }}- | -{{ }}- |

### Hardening Recommendations
- -{{ Fix #1 }}-
- -{{ Fix #2 }}-

---

## ▶ Juice Shop — [[0406.000 Juice Shop]]

### Metadata
- Difficulty: -{{ Based on challenge selection }}-
- OS: -{{ Linux (Node.js/Express) }}-
- URL: -{{ Base URL }}-

### Linked Pages
- Overview → [[0406.001 Overview]]
- Findings → [[0406.002 Findings]]
- Guide → [[0406.000-1 Juice Shop guide]]
- Workbook → [[0208.000 Juice Shop Workbook]]

### Recon Summary
- Open ports: -{{ nmap results }}-
- Services: -{{ Node.js/Express }}-

### Enumeration Summary
- API routes: -{{ /rest/* endpoints }}-
- Authentication: -{{ login behavior }}-
- Input validation: -{{ injection points }}-

### Vulnerability Summary
- Critical: -{{ }}-
- High: -{{ }}-
- Medium: -{{ }}-
- Low: -{{ }}-

### Exploitation Notes
- Entry point: -{{ vuln used }}-
- Payload: -{{ technique }}-

### Flags & Credentials
- User flag: -{{ }}-
- Admin flag: -{{ }}-
- Credentials: -{{ }}-

### Attack Chain Mapping
| Step | Action | Result |
|------|--------|--------|
| 1 | -{{ }}- | -{{ }}- |
| 2 | -{{ }}- | -{{ }}- |
| 3 | -{{ }}- | -{{ }}- |

### Hardening Recommendations
- -{{ Fix #1 }}-
- -{{ Fix #2 }}-

---

## ▶ OWASP BWA — [[0409.000 OWASP BWA]]

### Metadata
- Difficulty: -{{ Mixed }}-
- OS: -{{ Ubuntu-based }}-
- URL: -{{ Base URL }}-

### Linked Pages
- Overview → [[0409.001 Overview]]
- Findings → [[0409.002 Findings]]
- Guide → [[0409.000-1 OWASP BWA Guide]]
- Workbook → [[0205.000 OWASP BWA Workbook]]

### Recon Summary
- Open ports: -{{ nmap results }}-
- Services: -{{ Apache/PHP/MySQL/Tomcat }}-

### Enumeration Summary
- Hosted apps: -{{ DVWA, Mutillidae, WebGoat, WebWolf }}-
- Authentication: -{{ login behavior }}-
- Input validation: -{{ injection points }}-

### Vulnerability Summary
- Critical: -{{ }}-
- High: -{{ }}-
- Medium: -{{ }}-
- Low: -{{ }}-

### Exploitation Notes
- Entry point: -{{ vuln used }}-
- Payload: -{{ technique }}-

### Flags & Credentials
- User flag: -{{ }}-
- Admin flag: -{{ }}-
- Credentials: -{{ }}-

### Attack Chain Mapping
| Step | Action | Result |
|------|--------|--------|
| 1 | -{{ }}- | -{{ }}- |
| 2 | -{{ }}- | -{{ }}- |
| 3 | -{{ }}- | -{{ }}- |

### Hardening Recommendations
- -{{ Fix #1 }}-
- -{{ Fix #2 }}-

---

# 🧭 Related Workbooks
- [[0204.000 DVWA Workbook]]
- [[0210.000 Mutillidae Workbook]]
- [[0209.000 WebGoat Workbook]]
- [[0208.000 Juice Shop Workbook]]
- [[0205.000 OWASP BWA Workbook]]

---

# 🧰 Related Cheat Sheets
- [[0603.002 Web Application Cheat Sheet]]
- [[0603.004 Enumeration Cheat Sheet]]

---

# 📝 Notes & Journals
- [[0701.000 Daily Lab Log]]
- [[0702.000 Reflections]]
