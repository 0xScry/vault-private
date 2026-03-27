### Password Policy Fundamentals

A successful identity protection strategy requires two distinct components: **Definition** (outlining rules and expectations) and **Enforcement** (the technology used to ensure compliance). The scope of a policy must cover the entire **password lifecycle**, including creation, storage, management, and transmission.

### Standards and Industry Evolution

While organizations often follow established IT security standards to define a baseline, compliance alone does not guarantee security.

|Feature|Legacy Approach|Modern Recommendation|Why It Matters|
|:--|:--|:--|:--|
|**Password Expiration**|Mandatory changes (e.g., every 90 days).|**Disabled expiration** (unless compromise is confirmed).|Frequent forced changes lead users to adopt **predictable, weak patterns** (e.g., Incrementing a number at the end).|

### Common Policy Weaknesses & Mitigations

Standard complexity requirements can still result in weak passwords if users follow common mutation patterns.

|Identified Weakness|Attack Implication|Mitigation|
|:--|:--|:--|
|**Predictable Mutations**|Users change "01" to "02" upon expiration.|Disable forced expiration; implement **history requirements**.|
|**Contextual Keywords**|Use of company name or industry terms.|Implement **blacklisted words** (e.g., company name, local landmarks).|
|**OSINT Vulnerability**|Passphrases based on personal info (pets, hobbies).|Use random word generation or obscure phrases; avoid publicly discoverable info.|

### Implementation & Enforcement Methodology

Once a policy is defined, it must be technically enforced and communicated to the organization.

1. **Technical Integration**: Use identity management systems to enforce compliance.
    - **Scenario**: For Windows-centric environments, use **Active Directory Password Policy GPOs** to ensure users cannot bypass requirements.
2. **Communication**: Formalize the policy and distribute it to all employees.
3. **Process Standardization**: Create procedures to ensure the policy is applied consistently across all organizational platforms.

### Password Generation Techniques

|Method|Security Benefit|Potential Drawback|
|:--|:--|:--|
|**Randomly Generated**|High complexity; resistant to **password spraying** and cracking.|Difficult to remember without a **password manager**.|
|**Passphrases**|Long length increases crack time (e.g., lyrics, sentences).|Vulnerable to **OSINT** if the phrase is personally relevant.|

**Decision Point**: Use a **password manager** to generate and store high-entropy passwords, which balances the need for complexity with the human limitation of remembering multiple unique credentials.