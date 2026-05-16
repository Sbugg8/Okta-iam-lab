# Lab 9 — Set Up a Password Policy

## What is this?
This lab creates a custom **password policy** for the All Employees group with strong password requirements and conditional self-service account recovery rules based on network zone. Two rules govern recovery: one allows self-service password change, reset, and unlock from inside the Allowed Countries zone; the other blocks recovery from anywhere else.

## Why does it matter?
Password policies are the bedrock of identity assurance. Even in an MFA-enforced environment, the password remains the first factor — a weak, reused, or recoverable password lowers the cost of attack for everyone targeting the account.

This lab layers three categories of password security:
1. **Strength enforcement** — minimum length, complexity, common-password rejection, history tracking
2. **Lockout protection** — limits brute-force attempts against the password factor itself
3. **Conditional self-service recovery** — lets legitimate users recover their accounts without help desk intervention, but only from network locations the company considers safe

The third layer is the most operationally important. Self-service password recovery is one of the highest-value features for reducing help desk load — but unguarded, it's also one of the highest-risk paths. An attacker who has compromised a user's recovery email can use the self-service flow to take over the account and lock out the legitimate user. Conditioning recovery on a network zone limits where that attack can originate.

## What I configured

### Password policy: Okta Password Policy Employees
Created the policy assigned to the **All Employees** group, applies to Okta.

**Password Settings:**
- Minimum length: 12 characters
- Complexity: does not contain part of username, first name, or last name
- Common password check: restrict use of common passwords
- Password age: enforce password history for last 4 passwords
- Lockout: lock out user after 3 unsuccessful attempts

[![Password policy — top section](https://github.com/Sbugg8/Okta-iam-lab/raw/main/project-2/screenshots/implement-security-policies/lab-09/lab09a-create-password-policy-employees.png)](/Sbugg8/Okta-iam-lab/blob/main/project-2/screenshots/implement-security-policies/lab-09/lab09a-create-password-policy-employees.png)
*Top section of the Okta Password Policy Employees configuration — Active, assigned to All Employees, 12-character minimum with complexity rules blocking username/first/last name inclusion.*

[![Password policy — bottom section](https://github.com/Sbugg8/Okta-iam-lab/raw/main/project-2/screenshots/implement-security-policies/lab-09/lab09b-create-password-policy-employees.png)](/Sbugg8/Okta-iam-lab/blob/main/project-2/screenshots/implement-security-policies/lab-09/lab09b-create-password-policy-employees.png)
*Bottom section configuring Password Security (breached credentials protection, immediate log-out on expiry), Password age (history of last 4 passwords), and Lockout after 3 unsuccessful attempts.*

### Rule 1 — Allow self-service account recovery
IF User's IP is **In zone: Allowed Countries** → Users can perform self-service password change, password reset, and unlock account. Access control delegated to the Authentication policy.

[![Allow self-service rule](https://github.com/Sbugg8/Okta-iam-lab/raw/main/project-2/screenshots/implement-security-policies/lab-09/lab09-create-rule-for-password-policy.png)](/Sbugg8/Okta-iam-lab/blob/main/project-2/screenshots/implement-security-policies/lab-09/lab09-create-rule-for-password-policy.png)
*Allow self-service account recovery rule — when user is inside the Allowed Countries zone, all three self-service options (password change, reset, unlock account) are permitted. Access control is set to Authentication policy, meaning the recovery flow defers to the authentication policy for actual factor requirements.*

### Rule 2 — Block self-service account recovery
IF User's IP is **Not in zone: Allowed Countries** → all self-service options cleared.

[![Block self-service rule](https://github.com/Sbugg8/Okta-iam-lab/raw/main/project-2/screenshots/implement-security-policies/lab-09/lab09-create-second-rule-block-self-service-acct-recovery.png)](/Sbugg8/Okta-iam-lab/blob/main/project-2/screenshots/implement-security-policies/lab-09/lab09-create-second-rule-block-self-service-acct-recovery.png)
*Block self-service account recovery rule — when user is outside the Allowed Countries zone, all self-service options are cleared. The Okta warning highlights that without Password change enabled, users with expiring passwords have no path to recover themselves and must contact admin support.*

### Testing — password reset, lockout, unlock
Tested the policy by signing in as Krista Scott:
1. Reset her password from account settings using a new password that met the 12-character + complexity requirements
2. Triggered the lockout by entering an incorrect password three times → account moved to LOCKED_OUT state
3. From the sign-in page, used **Unlock account?** → authenticated via push notification → account auto-unlocked

The System Log captured the complete enforcement sequence: password reset event, max sign-in attempts exceeded (LOCKED_OUT), then auto-unlock after the push challenge succeeded.

[![System Log — password policy enforcement test](https://github.com/Sbugg8/Okta-iam-lab/raw/main/project-2/screenshots/implement-security-policies/lab-09/lab09-test-results-system-log-policy-enforcement.png)](/Sbugg8/Okta-iam-lab/blob/main/project-2/screenshots/implement-security-policies/lab-09/lab09-test-results-system-log-policy-enforcement.png)
*System Log showing the full test sequence: user password reset success (19:51:11), max sign-in attempts exceeded with LOCKED_OUT failure (19:54:13), then auto-unlock success after push authentication (19:55:20). The chronological order provides audit evidence that each policy enforcement fired exactly as designed.*

## What I learned
- **Password policies are prospective, not retroactive.** A new 12-character minimum doesn't break the existing 8-character passwords of current users. The policy constrains the next password change or reset — not the active credentials. This is important operationally: tightening password requirements doesn't cause mass lockouts, but it also means the stronger policy isn't fully realized until users naturally rotate their passwords (or an admin forces a reset event).
- **Self-service recovery is a network-zone problem, not just a credential problem.** The risk in self-service recovery isn't that the reset flow is technically weak — it's that an attacker who has captured the recovery email or possesses a stolen device can use that flow to take over the account. Conditioning recovery on the Allowed Countries zone limits where that attack can originate, even when the attacker has the recovery factor in hand.
- **Lockout policies balance security and operational pain.** Three attempts is tight enough to defeat credential stuffing but loose enough to absorb typos and sticky keyboards. Setting it too low (1–2) is effectively a denial-of-service against legitimate users; setting it too high (10+) makes brute force feasible. Three is the operating sweet spot for most environments.
- **"Authentication policy" as the access control is the integration point.** Setting the recovery rule's access control to **Authentication policy** delegates the factor requirements for the recovery flow back to the authentication policy built in Lab 8. The user can self-recover, but they still have to satisfy the auth policy's network-zone-conditioned MFA requirements during recovery itself. The two policies layer rather than compete.
- **System Log evidence is the audit trail.** The lockout/unlock/reset sequence in the Log is what proves the policy fired as designed. In a real incident review, this is what distinguishes a typo from a coordinated brute-force attack — and what shows whether an unlock came from legitimate self-service or admin intervention.
- **No policy in this module stands alone.** Lab 9's password policy interacts with Lab 5's enrollment policy (the user must have a recovery authenticator to use self-service unlock), Lab 8's authentication policy (the recovery flow defers to it for actual factor requirements), and Lab 2's network zones (the entire self-service split depends on the Allowed Countries zone being defined). The Implement Security Policies module composes — these labs aren't independent, they're a layered system.
