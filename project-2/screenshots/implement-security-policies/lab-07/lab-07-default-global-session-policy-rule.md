# Lab 07 – Add a Rule to the Default Global Session Policy

## What is this?

In this lab, I added a custom rule called "Standards" to Okta's Default Global Session Policy to align the user and global session settings with company standards. The rule defines session lifetime, idle timeout, and cookie persistence behavior for all users in the org who fall under the default policy.

## Why does it matter?

Global session settings govern how long a user stays signed in to Okta after authentication, how long an idle session can persist before termination, and whether sessions survive browser closures. These controls directly affect both user experience and security posture — sessions held too long leave stolen cookies useful to attackers; sessions held too short create user friction from constant re-authentication. Aligning these settings with company standards is a foundational hardening step in any Okta deployment.

---

## What I configured

- Navigated to Security > Global Session Policy > Default Policy
- Added a rule with these settings:
  * Rule name: Standards
  * IF User's IP is: Anywhere
  * AND Identity provider is: Any
  * AND Authenticates via: Any
  * THEN Access is: Allowed
  * Establish the user session with: Any factor used to meet the Authentication Policy requirements
  * Multifactor authentication (MFA): Not required (handled by Authentication Policy)
  * Maximum Okta global session lifetime: 12 hours
  * Maximum Okta global session idle time: 30 minutes
  * Okta global session cookies persist across browser sessions: Disabled
- Created the rule and verified it activated automatically

[![Standards Rule Configuration](https://github.com/Sbugg8/Okta-iam-lab/raw/main/project-2/screenshots/implement-security-policies/lab-07/lab-07-standards-rule-configuration.png)](/Sbugg8/Okta-iam-lab/blob/main/project-2/screenshots/implement-security-policies/lab-07/lab-07-standards-rule-configuration.png) *Add Rule form showing the Standards rule configured per company standards: 12-hour maximum session lifetime, 30-minute idle timeout, persistent cookies disabled, and session establishment delegated to the Authentication Policy.*

[![Default Policy with Standards Rule](https://github.com/Sbugg8/Okta-iam-lab/raw/main/project-2/screenshots/implement-security-policies/lab-07/lab-07-default-policy-with-standards-rule.png)](/Sbugg8/Okta-iam-lab/blob/main/project-2/screenshots/implement-security-policies/lab-07/lab-07-default-policy-with-standards-rule.png) *Default Policy view confirming the Standards rule landed at the expected priority and is active.*

---

## What I learned

The Global Session Policy governs the **Okta IdP session** (also called the SSO session) — the master session that gets established once a user signs in to Okta and which then enables seamless app access without re-prompting. This is distinct from the Authentication Policy, which governs *how* a user signs in. The sequence is: Authentication Policy challenges the user → Global Session Policy holds the resulting session.

Disabling persistent session cookies forces users to re-authenticate after closing their browser, limiting the blast radius of session hijacking from stolen cookies. The 12-hour maximum lifetime acts as a hard ceiling regardless of activity, while the 30-minute idle timeout terminates inactive sessions sooner — together these create defense in depth around session compromise scenarios.

I also confirmed an important Okta behavior: when you create a policy rule via "Create rule," it activates automatically. There is no separate activation step — creation and activation happen in one operation. Rules can be deactivated post-creation if needed, but the default state of a newly created rule is active.
