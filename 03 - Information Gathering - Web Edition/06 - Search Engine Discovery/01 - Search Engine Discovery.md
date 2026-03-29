### **Search Engine Discovery & Google Dorking**

Search engine discovery, or **OSINT (Open Source Intelligence) gathering**, uses search algorithms to extract data not readily visible on a website,. This technique is used during **web reconnaissance** to uncover employee information, sensitive documents, hidden login pages, and exposed credentials.

#### **Operational Methodology**

1. **Identify the Target Scope:** Use operators to limit results to the specific target domain.
2. **Filter for Sensitive Assets:** Apply specific terms (e.g., "password", "confidential") or file extensions (e.g., .pdf, .docx) to find internal documentation.
3. **Refine Results:** Use Boolean logic (AND, OR, NOT) or wildcards to narrow the search to high-value pages like admin panels or login portals,.
4. **Analyze Cached Data:** If a page is modified or inaccessible, view the version stored by the search engine to see previous content.

---

#### **Search Operator Reference**

|Operator|Purpose|Example Command|
|:--|:--|:--|
|**site:**|Limits results to a specific website or domain.|`site:<DOMAIN>`|
|**inurl:**|Finds pages with a specific term in the URL (e.g., identifying panels).|`inurl:login`|
|**filetype:**|Searches for files of a particular type (e.g., configuration or docs).|`filetype:pdf`|
|**intitle:**|Finds pages with specific terms in the HTML title.|`intitle:"confidential report"`|
|**intext: / inbody:**|Searches for terms within the body text of pages.|`intext:"password reset"`|
|**cache:**|Displays the cached version of a webpage to see past content.|`cache:<DOMAIN>`|
|**link:**|Identifies other websites linking to the target.|`link:<DOMAIN>`|
|**related:**|Discovers websites similar to the target.|`related:<DOMAIN>`|
|**info:**|Provides a summary of information about a webpage.|`info:<DOMAIN>`|
|**numrange: / ..**|Searches for numbers within a specific numerical range,.|`site:<DOMAIN> 1000..2000`|
|**allintext:**|Requires all specified words to appear in the body text.|`allintext:admin password reset`|
|**allinurl:**|Requires all specified words to appear in the URL.|`allinurl:admin panel`|
|**allintitle:**|Requires all specified words to appear in the title.|`allintitle:confidential report 2023`|
|**" "**|Searches for an exact phrase.|`"information security policy"`|
|*****|Wildcard representing any character or word.|`site:<DOMAIN> filetype:pdf user* manual`|
|**- / NOT**|Excludes specific terms from the results,.|`site:<DOMAIN> -inurl:sports`|

---

#### **Logic and Precision Operators**

|Logic|Action|Example Command|
|:--|:--|:--|
|**AND**|Narrows results by requiring all terms to be present.|`site:<DOMAIN> AND (inurl:admin OR inurl:login)`|
|**OR**|Broadens results by including pages with any of the terms.|`"linux" OR "ubuntu" OR "debian"`|

---

#### **Google Dorking (Google Hacking)**

Google Dorking is a specialized technique using the operators above to specifically uncover **security vulnerabilities**, **sensitive information**, or **hidden content**.

**Note on Attack Implications:**

- **Technique Unlock:** Successfully utilizing dorks can provide a foothold via hidden login panels or direct access to sensitive data via exposed files,.
- **Failure Conditions:** Search engines do not index all information; data that is deliberately hidden or protected by the site owner may not appear in results.
- **External Resource:** For complex queries, refer to the **Google Hacking Database (GHDB)**.