# PortSwigger Web Security Academy: SQL Injection in WHERE Clause

<p align="center">
  <img src="https://img.shields.io/badge/Platform-PortSwigger-orange?style=for-the-badge" alt="PortSwigger">
  <img src="https://img.shields.io/badge/Category-SQL%20Injection-blue?style=for-the-badge" alt="SQL Injection">
  <img src="https://img.shields.io/badge/Difficulty-Apprentice-brightgreen?style=for-the-badge" alt="Apprentice">
  <img src="https://img.shields.io/badge/Status-Solved-success?style=for-the-badge" alt="Solved">
</p>

<h2>📌 Lab Information</h2>

<table>
  <tr><th>Information</th><th>Details</th></tr>
  <tr><td>Platform</td><td>PortSwigger Web Security Academy</td></tr>
  <tr><td>Lab Name</td><td>SQL injection vulnerability in WHERE clause allowing retrieval of hidden data</td></tr>
  <tr><td>Category</td><td>SQL Injection</td></tr>
  <tr><td>Difficulty</td><td>Apprentice</td></tr>
  <tr><td>Vulnerability</td><td>SQL Injection (SQLi)</td></tr>
  <tr><td>Injection Point</td><td>category parameter</td></tr>
  <tr><td>Tool Used</td><td>Burp Suite Community Edition</td></tr>
  <tr><td>Status</td><td>Successfully Solved</td></tr>
</table>

<h2>📝 Introduction</h2>

<p>This write-up documents my successful completion of the <strong>SQL injection vulnerability in WHERE clause allowing retrieval of hidden data</strong> lab on PortSwigger Web Security Academy.</p>

<p>The objective of this lab was to exploit a SQL injection vulnerability in a product filtering feature and retrieve hidden products that were not displayed under the selected category.</p>

<p>The vulnerability occurs because the application incorporates user-supplied input into an SQL query without properly preventing SQL injection.</p>

<p>By manipulating the category parameter, I was able to modify the application's database query and retrieve products from multiple categories.</p>

<h2>🔍 Step 1: Initial Reconnaissance</h2>

<p>After launching the lab, I explored the shopping application and examined its available product categories.</p>

<p>The application provided a category filtering feature that allowed users to browse products based on selected categories.</p>

<p>While examining the URL, I noticed that the selected category was passed through a URL parameter named <code>category</code>.</p>

<p>This parameter became my primary target for SQL injection testing.</p>

<h2>🔎 Step 2: Identifying the Injection Point</h2>

<p>I examined the application's URL structure and identified the category parameter:</p>

<pre><code>https://&lt;lab-instance&gt;.web-security-academy.net/filter?category=Food+%26+Drink</code></pre>

<p>Since the application appeared to use this parameter to filter database records, I investigated whether its input was properly handled.</p>

<p>I intercepted the HTTP request using <strong>Burp Suite Community Edition</strong> and forwarded it to Repeater for manual testing.</p>

<h3>Screenshot 1: Successful Lab and Injection Result</h3>

<p align="center">
  <img width="1919" height="884" alt="sql1" src="https://github.com/user-attachments/assets/ee773938-9b15-426d-b512-de890ccc504e" />
</p>

<p align="center"><em>Figure 1: The browser displays the successful lab completion message and the SQL injection payload in the category parameter.</em></p>

<h2>💉 Step 3: Exploiting the SQL Injection Vulnerability</h2>

<p>To investigate the vulnerability, I modified the category parameter using a Boolean-based SQL injection payload.</p>

<p>The payload used was:</p>

<pre><code class="language-sql">' OR 1=1--</code></pre>

<p>URL-encoded, the payload was transmitted as:</p>

<pre><code>filter?category='+OR+1=1--</code></pre>

<p>The injection works by introducing a single quote to close the original SQL string, followed by an <code>OR</code> condition that always evaluates to true.</p>

<p>The double hyphen (<code>--</code>) begins an SQL comment, causing the remaining portion of the original query to be ignored in database systems that support this comment syntax.</p>

<p>For example, the vulnerable query may conceptually resemble:</p>

<pre><code class="language-sql">SELECT *
FROM products
WHERE category = 'Food & Drink'
AND released = 1;</code></pre>

<p>After injection, the resulting query may conceptually become:</p>

<pre><code class="language-sql">SELECT *
FROM products
WHERE category = '' OR 1=1--'
AND released = 1;</code></pre>

<p>Because <code>1=1</code> is always true, the category restriction is bypassed and the query can return products from all categories.</p>

<p><strong>Note:</strong> The SQL statements above illustrate the likely query structure; the application's actual backend query was not directly verified.</p>

<h3>Screenshot 2: Burp Suite Request and Response</h3>

<p align="center">
 <img width="1915" height="901" alt="sql2" src="https://github.com/user-attachments/assets/9cc97d4f-d068-4620-b785-9fed642b794b" />
 </p>

<p align="center"><em>Figure 2: Testing the SQL injection payload in Burp Suite Repeater and inspecting the server response.</em></p>

<h2>📊 Step 4: Retrieving Hidden Product Data</h2>

<p>After sending the modified request, I examined the server response.</p>

<p>The application returned products from categories beyond the originally selected category.</p>

<p>This confirmed that the injected Boolean condition successfully altered the filtering behavior.</p>

<p>The result demonstrated that an attacker could manipulate the application's SQL query to retrieve hidden product information.</p>

<h3>Screenshot 3: Retrieved Products in Burp Suite</h3>

<p align="center">
 <img width="1916" height="927" alt="sql3" src="https://github.com/user-attachments/assets/0c6a29e0-fff6-40ca-bc3f-b99e5837266b" />
</p>

<p align="center"><em>Figure 3: The rendered response displays products from multiple categories after successful SQL injection.</em></p>

<h2>✅ Step 5: Lab Completion</h2>

<p>After confirming that the injected request successfully retrieved products from different categories, the PortSwigger platform recognized the vulnerability as exploited.</p>

<p>The lab displayed the confirmation message:</p>

<blockquote>
  <p><strong>Congratulations, you solved the lab!</strong></p>
</blockquote>

<p>This confirmed successful completion of the exercise.</p>

<h2>🛡️ Vulnerability Analysis</h2>

<p>The vulnerability was caused by unsafe handling of user-controlled input within an SQL query.</p>

<p>By injecting SQL syntax into the category parameter, it was possible to modify the original database query and bypass the intended category restriction.</p>

<h3>Security Impact</h3>

<ul>
  <li>Unauthorized retrieval of products from restricted categories.</li>
  <li>Bypassing application-level filtering conditions.</li>
  <li>Potential exposure of database information.</li>
  <li>Possible escalation to more serious database attacks depending on the application's configuration and database privileges.</li>
</ul>

<h2>🔐 Recommended Mitigations</h2>

<p>Developers can prevent this vulnerability by implementing the following security measures:</p>

<ol>
  <li>Use parameterized SQL queries and prepared statements.</li>
  <li>Never concatenate untrusted user input directly into SQL statements.</li>
  <li>Apply appropriate input validation.</li>
  <li>Use database accounts with restricted privileges.</li>
  <li>Implement secure error handling to avoid exposing database information.</li>
</ol>

<h2>💡 Lessons Learned</h2>

<p>This lab helped reinforce several important SQL injection concepts:</p>

<ul>
  <li>How user-controlled URL parameters can introduce SQL injection vulnerabilities.</li>
  <li>How Boolean-based conditions can manipulate SQL query logic.</li>
  <li>How SQL comments can be used to neutralize the remainder of an injected query.</li>
  <li>How Burp Suite Repeater can be used to modify and test HTTP requests.</li>
  <li>Why parameterized queries are essential for database security.</li>
</ul>

<h2>🧰 Tools Used</h2>

<ul>
  <li>Kali Linux</li>
  <li>Mozilla Firefox</li>
  <li>Burp Suite Community Edition</li>
  <li>PortSwigger Web Security Academy</li>
</ul>

<h2>🏁 Conclusion</h2>

<p>The <strong>SQL injection vulnerability in WHERE clause allowing retrieval of hidden data</strong> lab provided practical experience in identifying and exploiting SQL injection vulnerabilities.</p>

<p>By identifying the category parameter as an injection point and using the Boolean-based payload <code>' OR 1=1--</code>, I successfully bypassed the application's category filtering mechanism and retrieved hidden product data.</p>

<p>This exercise strengthened my understanding of SQL query manipulation, HTTP request analysis, and the importance of secure database interaction.</p>

<p><strong>Lab Status: Successfully Completed! 🚀</strong></p>

<hr>

<p align="center">
  <strong>Hack. Learn. Improve. Repeat.</strong>
  <br>
  <em>Documenting my journey in cybersecurity and ethical hacking.</em>
</p>
