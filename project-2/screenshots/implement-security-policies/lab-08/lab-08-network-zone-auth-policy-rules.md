# Lab 8 — Add Rules Based on Network Zones to the Okta Dashboard Authentication Policy

## What is this?
This lab adds three network-zone-based rules to the **Okta Dashboard authentication policy** to enforce different access requirements based on where a user is connecting from. The rules layer on top of the baseline Okta Dashboard policy and are rolled out in phases starting with the Pilot Users group.

The three business requirements:
- **Deny** access from restricted countries (anything outside the Allowed Countries zone)
- **Allow** access from a public network using a hardware-protected possession factor requiring user interaction
- **Allow** access from the corporate network using a possession factor requiring user interaction (broader, since the network is already trusted)

## Why does it matter?
This is the most operationally consequential lab in the module. It's where the network zones built in Labs 1 and 2 actually start enforcing access — everything before this was foundation. The rules implement the core principle of **conditional access**: the strength of authentication required should match the risk of the request's origin.

Production IAM teams use this exact pattern every day. Corporate network traffic is treated as more trustworthy because it's been validated by network controls before reaching Okta; public network traffic needs stronger MFA factors (hardware-protected, phishing-resistant) because it could be coming from anywhere; restricted geographies are denied outright because there's no legitimate reason for the company to allow logins from those locations.

## What I configured

### Baseline (before changes)
The Okta Dashboard policy started with only a disabled "Password Only Rule" at Priority 1 and the system "Catch-all Rule" at Priority 2 — i.e., any successful 2-factor authentication granted access, regardless of network origin.

[![Dashboard policy baseline](https://github.com/Sbugg8/Okta-iam-lab/raw/main/project-2/screenshots/implement-security-policies/lab-08/lab08-dashboard-policy-baseline.png)](/Sbugg8/Okta-iam-lab/blob/main/project-2/screenshots/implement-security-policies/lab-08/lab08-dashboard-policy-baseline.png)
*Baseline state of the Okta Dashboard authentication policy before any network-zone rules were added. The Catch-all Rule at Priority 2 was the only enforcement layer.*

### Rule 1 — Restricted Countries (Deny)
IF Pilot Users **AND** User's IP is **Not in** Allowed Countries → **Access Denied**

[![Restricted Countries IF config](https://github.com/Sbugg8/Okta-iam-lab/raw/main/project-2/screenshots/implement-security-policies/lab-08/lab08-restricted-countries-rule-added.png)](/Sbugg8/Okta-iam-lab/blob/main/project-2/screenshots/implement-security-policies/lab-08/lab08-restricted-countries-rule-added.png)
*Add Rule form for Restricted Countries — IF section scoped to Pilot Users group.*

[![Restricted Countries THEN denied](https://github.com/Sbugg8/Okta-iam-lab/raw/main/project-2/screenshots/implement-security-policies/lab-08/lab08b-restricted-countries-rule-added.png)](/Sbugg8/Okta-iam-lab/blob/main/project-2/screenshots/implement-security-policies/lab-08/lab08b-restricted-countries-rule-added.png)
*Restricted Countries rule completed — IF Not in Allowed Countries zone, THEN Access Denied.*

### Rule 2 — Public Network (Allow with hardware-protected possession)
IF Pilot Users **AND** User's IP is **Not in** Corporate Network → **Allowed** with any 2 factor types, **Hardware protected** + Require user interaction. Prompt every sign-in.

[![Public Network IF config](https://github.com/Sbugg8/Okta-iam-lab/raw/main/project-2/screenshots/implement-security-policies/lab-08/lab08-add-rule-public-network.png)](/Sbugg8/Okta-iam-lab/blob/main/project-2/screenshots/implement-security-policies/lab-08/lab08-add-rule-public-network.png)
*Public Network rule IF section — applies when user is outside the Corporate Network zone.*

[![Public Network THEN hardware-protected](https://github.com/Sbugg8/Okta-iam-lab/raw/main/project-2/screenshots/implement-security-policies/lab-08/lab08b-add-rule-public-network.png)](/Sbugg8/Okta-iam-lab/blob/main/project-2/screenshots/implement-security-policies/lab-08/lab08b-add-rule-public-network.png)
*Public Network THEN section — Hardware protected possession factor required. Notably, Okta Verify TOTP is excluded from the satisfying authenticators because TOTP codes are not hardware-protected.*

### Rule 3 — Corporate Network (Allow with possession factor + user interaction)
IF Pilot Users **AND** User's IP is **In** Corporate Network → **Allowed** with any 2 factor types, Require user interaction (no hardware-protection requirement)

[![Corporate Network IF config](https://github.com/Sbugg8/Okta-iam-lab/raw/main/project-2/screenshots/implement-security-policies/lab-08/lab08-add-rule-corporate-network.png)](/Sbugg8/Okta-iam-lab/blob/main/project-2/screenshots/implement-security-policies/lab-08/lab08-add-rule-corporate-network.png)
*Corporate Network rule IF section — applies when user is inside the Corporate Network zone.*

[![Corporate Network THEN any method](https://github.com/Sbugg8/Okta-iam-lab/raw/main/project-2/screenshots/implement-security-policies/lab-08/lab08b-add-rule-corporate-network.png)](/Sbugg8/Okta-iam-lab/blob/main/project-2/screenshots/implement-security-policies/lab-08/lab08b-add-rule-corporate-network.png)
*Corporate Network THEN section — broader authenticator set including Okta Verify TOTP and Google Authenticator, because the trusted network reduces the required factor strength.*

### Verification
Confirmed the three rules sit above the system Catch-all rule in the correct priority order: Restricted Countries (Priority 1) → Public Network (Priority 2) → Corporate Network (Priority 3) → Catch-all (Priority 4).

[![Restricted Countries rule enabled](https://github.com/Sbugg8/Okta-iam-lab/raw/main/project-2/screenshots/implement-security-policies/lab-08/lab08-active-policy-rule.png)](/Sbugg8/Okta-iam-lab/blob/main/project-2/screenshots/implement-security-policies/lab-08/lab08-active-policy-rule.png)
*Restricted Countries rule active and enforcing Deny for Pilot Users outside the Allowed Countries zone.*

[![Rule priority ordering](https://github.com/Sbugg8/Okta-iam-lab/raw/main/project-2/screenshots/implement-security-policies/lab-08/lab08-rule-placement-top-down.png)](/Sbugg8/Okta-iam-lab/blob/main/project-2/screenshots/implement-security-policies/lab-08/lab08-rule-placement-top-down.png)
*Rule priority ordering with Catch-all rule sitting below the new network-zone rules.*

[![Public Network at Priority 2](https://github.com/Sbugg8/Okta-iam-lab/raw/main/project-2/screenshots/implement-security-policies/lab-08/lab08-verification-public-rule-is-priority-2.png)](/Sbugg8/Okta-iam-lab/blob/main/project-2/screenshots/implement-security-policies/lab-08/lab08-verification-public-rule-is-priority-2.png)
*Verification that the Public Network rule sits at Priority 2 above the Corporate Network and Catch-all rules.*

### Testing — IP zone manipulation
To test both the Public Network and Corporate Network rules without actually changing my real network location, I manipulated the Corporate Network zone's IP definition:

1. Changed the Corporate Network zone to `10.10.10.10` (a private RFC 1918 address my real traffic would never match). This made my real IP effectively "public" for testing purposes.
2. Signed in as Krista, confirmed the Public Network rule fired (push notification required, TOTP option excluded).
3. Verified in the System Log that the Public Network Rule was the target of the evaluation.
4. Changed the Corporate Network zone back to my real current IP address.
5. Signed in as Krista again, confirmed the Corporate Network rule now fired (TOTP option became available, broader authenticator set allowed).
6. Verified in the System Log that the Corporate Network Rule was the target of the evaluation.

[![Change Corporate Network zone for public test](https://github.com/Sbugg8/Okta-iam-lab/raw/main/project-2/screenshots/implement-security-policies/lab-08/lab08-change-corporate-network-zone.png)](/Sbugg8/Okta-iam-lab/blob/main/project-2/screenshots/implement-security-policies/lab-08/lab08-change-corporate-network-zone.png)
*First IP swap — Corporate Network set to 10.10.10.10 to simulate a user outside the corporate network.*

[![Restore Corporate Network zone to real IP](https://github.com/Sbugg8/Okta-iam-lab/raw/main/project-2/screenshots/implement-security-policies/lab-08/lab08-change-corporate-network-zone-again.png)](/Sbugg8/Okta-iam-lab/blob/main/project-2/screenshots/implement-security-policies/lab-08/lab08-change-corporate-network-zone-again.png)
*Second IP swap — Corporate Network restored to real current IP to test the Corporate Network rule path.*

### System Log evidence
The System Log entries for both test sign-ins explicitly named the rule that fired during the policy evaluation, providing the audit trail that proves the rules behaved as designed.

[![System Log — Public Network rule verified](https://github.com/Sbugg8/Okta-iam-lab/raw/main/project-2/screenshots/implement-security-policies/lab-08/lab08-krista-scott-evaluation-of-sign-on-policy-target-public-network.png)](/Sbugg8/Okta-iam-lab/blob/main/project-2/screenshots/implement-security-policies/lab-08/lab08-krista-scott-evaluation-of-sign-on-policy-target-public-network.png)
*System Log evaluation of sign-on policy showing the Public Network rule was the target during Krista's first test sign-in.*

[![System Log — Corporate Network rule verified](https://github.com/Sbugg8/Okta-iam-lab/raw/main/project-2/screenshots/implement-security-policies/lab-08/lab08-krista-scott-corporate-rule-evaluation-of-sign-on-verified.png)](/Sbugg8/Okta-iam-lab/blob/main/project-2/screenshots/implement-security-policies/lab-08/lab08-krista-scott-corporate-rule-evaluation-of-sign-on-verified.png)
*System Log evaluation showing the Corporate Network rule was the target after the IP zone was restored to my real address.*

## What I learned
- **IP volatility surfaces a real production design lesson.** During testing, my actual IP shifted, which broke the brittle single-IP zone definition I had built in Lab 1. In a real production environment, defining the Corporate Network zone as a single IP would mean any DHCP renewal, ISP change, or office move silently breaks conditional access. The production answer is **CIDR ranges** (allowing entire subnets the company controls) or **trusted proxy IPs** (when corporate traffic egresses through a known proxy or VPN concentrator). Single-IP zones are fine for labs; they're a liability in production.
- **Hardware-protected possession excludes TOTP for a reason.** TOTP codes are software-generated and can be phished by an attacker who tricks the user into typing the code into a fake page. Hardware-protected possession factors (FastPass, Push with device attestation) require a cryptographic challenge bound to the physical device. The Public Network rule deliberately excludes TOTP to defend against the higher phishing risk on untrusted networks.
- **Policy strength should match request risk.** The Corporate Network rule allows TOTP and broader authenticators because the user has already passed the implicit network-level trust check (being inside the corporate zone). The Public Network rule is stricter because the network has provided no trust. Same authentication policy, different requirements per request context — that's conditional access in practice.
- **Priority ordering encodes the policy logic.** With Restricted Countries at Priority 1, denies fire before any allow rule can evaluate. With Public Network at Priority 2 and Corporate Network at Priority 3, the system evaluates the more restrictive condition first. If priorities were flipped (Corporate at 2, Public at 3), a user inside the corporate network would still hit the Public Network rule when their IP didn't match Corporate Network — but the rule order in this configuration makes the intent unambiguous.
- **System Log is the proof.** Without checking the Log's "evaluation of sign-on policy" entries, I could only assume the right rule fired. With the Log evidence, I have audit-grade confirmation of which rule made which decision. In a production incident, this is the trail that explains why a user was or wasn't granted access.
- **Phased rollouts via Pilot Users group are non-negotiable.** Pushing these rules to the whole org on day one would risk locking real users out if any rule has a bug. The Pilot Users pattern is the same safety net as Lab 5 — limit the blast radius, validate, then expand.
