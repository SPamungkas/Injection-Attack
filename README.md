<img width="527" height="58" alt="image" src="https://github.com/user-attachments/assets/3d00b895-d2e9-4377-8d76-f398594be56b" /># Injection-Attack
This project provides a comprehensive demonstration of Injection Attacks, specifically focusing on SQL Injection (SQLi) one of the most critical web vulnerabilities.

# About Me 
Information Systems graduate, currently transitioning to Cyber Security. Previously served as a Technician at Oil and Gas Industry (Pertamina Patra Niaga vendor), where I developed a highly disciplined approach to managing critical infrastructure compliance and 24/7 reliability. Currently formalizing expertise through an intensive cybersecurity bootcamp at Dibimbing, I am eager to apply this technical foundation and structured approach to a security operations role.

# Tools
- Kali Linux
- Virtual Machine
- MySQL
- Burp Suite

# Notes 
This injection is carried out in a controlled environment and poses no public risk to the CIA triad. It is used purely for educational purposes.

# SQLi Walkthrough
<p align="center">
  <img width="500" height="142" alt="image" src="https://github.com/user-attachments/assets/04bb785c-41b6-47aa-b8e1-f6749f97218f" />
  <br>
</p>
Before attempting any exploitation, I conducted a baseline test by entering invalid credentials into the login form. This step is essential to observe the application’s response logic when an authentication failure occurs. When an incorrect username or password was submitted, the website returned a specific error message.
<br>
<br>

```bash
#SQLi payload
SELECT * FROM users WHERE username = '$user' AND password = '$password';
```
<p align="center">
  <img width="500" height="500" alt="image" src="https://github.com/user-attachments/assets/b1813ebe-4ba3-4a96-935c-28ca172f0de5" />
  <br>
</p>
To test for SQL Injection vulnerabilities, I deliberately injected a payloads on the username field. If the application is vulnerable, this character will break the SQL query structure on the backend.
<br>
<br>
<p align="center">
 <img width="500" height="332" alt="image" src="https://github.com/user-attachments/assets/899a13cb-6167-4200-b5f0-c7313ce89085" />
  <br>
</p>
To capture and analyze the raw HTTP request sent by the browser to the server using Burp Suite. This allows for precise manipulation of input parameters before they reach the backend database. The visibility of credentials in plaintext within Burp Suite confirms that no client-side protection or secure transmission protocol is in place. This transparency allowed me to verify the exact parameter names needed to execute the SQL Injection attack in the next phase.
<br>
<br>

```bash
#SQLi payload
' OR '1'='1
```
<p align="center">
 <img width="500" height="175" alt="image" src="https://github.com/user-attachments/assets/b3f6d06f-604e-47dc-b363-b6e23b11816a" />
  <br>
</p>
In the login form, I injected the following payload above into the password field. The application granted access to the protected area of the website, bypassing the password requirement entirely. I was logged in as the user specified in the username field, or the first user found in the database.
<br>
<br>

```bash
#SQLi payload
sqlmap --url 'http://192.168.x.x/sqlitest/login.php' --data=’username=admin&password=admin123’ 
```
<p align="center">
 <img width="500" height="82" alt="image" src="https://github.com/user-attachments/assets/38e570e0-3d20-4d72-a333-884dff93e6b7" />
  <br>
</p>
To perform an automated deep-dive analysis of the identified vulnerability using SQLmap, confirming the injection points and determining the specific type of SQL Injection present. Using the data captured from the Burp Suite interception, I executed the following command in Kali Linux to test the login parameters. Unlike the previous manual attack that triggered a syntax error or a direct bypass, Boolean-based Blind SQLi relies on asking the database "True" or "False" questions. The attacker observes whether the application returns a specific result or different result. By analyzing these binary responses (1 or 0), an attacker can systematically extract the entire database structure, table names, and even user credentials, character by character.
<br>
<br>
<p align="center">
 <img width="500" height="117" alt="image" src="https://github.com/user-attachments/assets/fe2fedde-befb-4084-8d89-726c1b66d39a" />
  <br>
</p>
In addition to the blind injection techniques, SQLmap identified that the username parameter is also susceptible to Error Based SQL Injection. This is a highly efficient exploitation technique where the attacker manipulates the database query to intentionally trigger a verbose error message. This attack leverages the database's own error-reporting mechanism. By injecting specific SQL commands the attacker forces the database to include the results of a subquery (like the database version or table names) directly within the error message returned to the frontend.
<br>
<br>
<p align="center">
 <img width="500" height="58" alt="image" src="https://github.com/user-attachments/assets/399a30b9-2707-4eba-91e2-baa913b7edd9" />
  <br>
</p>
The most dangerous vulnerability identified by SQLmap on the username parameter is Stacked Queries Injection. This vulnerability occurs when the applications database driver allows multiple SQL statements to be executed in a single call, typically separated by a semicolon (;). In a standard SQL injection, the attacker is limited to modifying the existing query. With Stacked Queries, the attacker can terminate the original query and start a completely new, malicious command.
<br>
<br>

<p align="center">
 <img width="500" height="71" alt="image" src="https://github.com/user-attachments/assets/7bfac94a-471f-472e-a8cf-79bb152e4247" />
  <br>
</p>
The final vulnerability detected by SQLmap on the username parameter is Time-Based Blind SQL Injection. This is an advanced inference-based attack used when the application does not return any visible database errors or different content for True/False conditions. This attack relies on the database's ability to execute a time-delay command. I observed that the attacker can inject a conditional query that triggers a delay only if a specific condition is met.
<br>
<br>

<p align="center">
 <img width="500" height="88" alt="image" src="https://github.com/user-attachments/assets/8918483a-0784-41bb-9626-305c7225c125" />
  <br>
</p>
The username parameter was also found to be vulnerable to UNION-Based SQL Injection. This is one of the most effective SQLi techniques, as it allows an attacker to append (JOIN) the results of their own malicious query to the results of the original application query. This attack utilizes the UNION operator to combine two separate SELECT statements into a single result set. For this to work, the attacker first determines the number of columns in the original query and their data types.The presence of UNION Based SQLi indicates that the application is directly rendering database results into the UI without proper output encoding or input parameterization. This vulnerability transforms a simple login form into an open gateway to the entire backend storage.

# Database Dump 
To demonstrate the full impact of the identified UNION Based SQL Injection by extracting sensitive information from the backend database. This phase proves that the vulnerability is not just theoretical but a high-risk entry point for a full data breach.
```bash
#payload
sqlmap --url 'http://192.168.x.x/sqlitest/login.php' --data='username=admin&password=admin123' --dbs 
```
<p align="center">
 <img width="500" height="212" alt="image" src="https://github.com/user-attachments/assets/ea8cfe0f-4bf4-4ecc-a040-839c2e4dd866" />
  <br>
</p>
Before extracting data, the first step in a professional penetration test is to enumerate the environment. The goal is to identify all available databases on the server to pinpoint the specific target for exfiltration.
<br>
<br>

```bash
#payload
sqlmap --url 'http://192.168.x.x/sqlitest/login.php' --data='username=admin&password=admin123' -D testdb --tables
```
<p align="center">
 <img width="500" height="127" alt="image" src="https://github.com/user-attachments/assets/137bb368-58c8-47a5-ae48-add3f6ebce45" />
  <br>
</p>
Once the database name was confirmed, the next step was to explore the internal structure of testdb. The goal of this phase is to identify specific tables that likely contain sensitive information, such as user credentials.
<br>
<br>

```bash
#payload
sqlmap --url 'http://192.168.1.3/sqlitest/login.php'--data='username=admin&password=admin123' -D testdb -T users --columns
```
<p align="center">
 <img width="500" height="211" alt="image" src="https://github.com/user-attachments/assets/bea8080b-4458-422e-8d28-4d83236ed24c" />
  <br>
</p>
After identifying the users table, I proceeded to map its internal structure. Enumerating columns is essential to understand exactly what types of sensitive data are stored (e.g., whether passwords are stored alongside usernames) and to prepare for a precise data extraction. The command successfully revealed the structure of the users table.
<br>
<br>

```bash
#payload
sqlmap --url 'http://192.168.1.3/sqlitest/login.php'--data='username=admin&password=admin123' -D testdb -T users --columns --dump
```
<p align="center">
 <img width="500" height="290" alt="image" src="https://github.com/user-attachments/assets/6db37cce-83b9-47b3-8a30-37069d3449b3" />
  <br>
</p>
With the database, table, and columns mapped, I executed the --dump command. This instruction tells SQLmap to retrieve all row data and save it locally for offline analysis. The successful dump of the users table marks a complete compromise of the application's Confidentiality. An attacker now possesses the credentials for every user in the database. This allows for unauthorized account takeovers, identity theft, and potential lateral movement into other connected systems.

# Mitigation Reccommedations
The security assessment identified that the application is susceptible to multiple high risk SQL Injection (SQLi) vectors, including Boolean Based, Time Based, Error Based, Stacked Queries, and UNION Based attacks. To mitigate these risks and harden the applications defenses, the following industry standard practices must be implemented:
- Implementation of Prepared Statements (Parameterized Queries)
- Utilization of Secure Stored Procedures
- Enforcing Strict Input Validation (Allow-listing)
- Principle of Least Privilege (PoLP)
