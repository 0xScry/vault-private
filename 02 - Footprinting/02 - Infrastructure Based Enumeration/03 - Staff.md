**Staff Enumeration (Passive)**

People leak infrastructure info constantly — LinkedIn, GitHub, job posts.

---

**Job postings** reveal the tech stack directly:

- Languages, frameworks, databases, ORMs, CI/CD tools
- Cloud/container tech (Docker, K8s, Redis)
- Third-party platforms (Atlassian = Jira/Confluence/Bitbucket access possible)

**LinkedIn profiles:**

- Career history → tech experience, current projects
- "About" sections → frameworks, GitHub links
- Look for devs _and_ security staff — security team reveals what defenses exist

**GitHub (from employee profiles):**

- Public repos may contain hardcoded secrets, JWT tokens, API keys
- Personal email addresses in commits/configs
- Project structure reveals naming conventions used in production (people follow tutorials and name files exactly as shown)

---

**What to do with findings:**

| Finding           | Action                                           |
| ----------------- | ------------------------------------------------ |
| Django/Flask used | Check OWASP Top10 for that framework             |
| Atlassian suite   | Look for exposed Jira/Confluence instances       |
| GitHub link       | Hunt for hardcoded secrets, tokens, internal IPs |
| JWT in repo       | Try to decode/forge it                           |
| Personal email    | Password reuse, OSINT pivot                      |

**Target profiles to prioritize:** developers + security staff — devs expose tech, security staff expose defenses.