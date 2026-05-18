## Methodology

1. Identify the target company on business networks like LinkedIn or Xing to enumerate the technical staff and organizational makeup.
2. Analyze active and archived job postings to map the internal infrastructure, specifically looking for required experience in databases, web frameworks, and version control systems.
3. Filter for employees in development and security roles to determine implemented security measures and internal tech focus.
4. Pivot from social media profiles to personal external resources, such as GitHub repositories, looking for project history and shared code.
5. Inspect discovered code for hardcoded credentials, personal identifiers, or framework-specific misconfigurations.

---

## Infrastructure Mapping via Job Postings

**When to use** Target infrastructure is unknown and `<DOMAIN>` has public job listings on LinkedIn or Xing.

**Commands** Filter for these specific technology categories within job descriptions to identify the stack:

```
Database: PostgreSQL, MySQL, SQL Server, Oracle
Web Frameworks: Flask, Django, Spring, ASP.NET MVC
ORM: SQLAlchemy, Hibernate, Entity Framework
CI/CD & Version Control: Git, SVN, Mercurial, Perforce, Bitbucket
Project Management: Confluence, Jira
```

**Dangerous / misconfigured settings**

- Predictable file naming based on framework **best practices** allowing for easier discovery of internal structures.
- Publicly accessible Atlassian suite resources (Confluence/Jira/Bitbucket) linked in job requirements.

**Gotchas** **Narrow search parameters** on LinkedIn (sorting by too many filters simultaneously) will return zero results; start broad and filter down.

## Employee Intelligence and SOCMINT

**When to use** Need to assess the knowledge level of the defense team or identify specific programming languages in use via employee career history.

**Commands** Analyze "About" and "Experience" sections for specific frontend and backend proficiencies:

```
Frontend: W3C specs, web components, React, Svelte, AngularJS
Backend: Java, C#, C++, Python, Ruby, PHP, Perl
Specialized: Image Processing algorithms, Containerization (Docker, Kubernetes), Redis, NumPy
```

**Edge cases**

- Employees may list specific internal systems (e.g., "BrokerVotes system" or "CRM mobile app") which provides targets for mobile application testing or internal name discovery.

**Gotchas** **Focusing only on developers** ignores security staff whose profiles often reveal the specific security measures and defensive tools the company has implemented.

## Public Repository and Secret Analysis

**When to use** Employee profiles provide links to personal GitHub accounts or open-source contributions.

**Commands** Search for hardcoded authentication tokens or specific framework vulnerabilities in public repositories:

```
jwt.decode(
    payload,
    "SECRET_KEY",
    algorithms=["HS256"]
)
```

**Dangerous / misconfigured settings**

- **Hardcoded JWT tokens** in personal projects that mirror production code.
- Personal email addresses embedded in repository metadata or code comments.
- **OWASP Top 10** misconfigurations in common frameworks like Django.

**Gotchas** **Blindly trusting framework defaults** often leads developers to name files exactly as shown in tutorials, making automated discovery trivial.