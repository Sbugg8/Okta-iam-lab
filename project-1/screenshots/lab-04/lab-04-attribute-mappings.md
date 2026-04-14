# Lab 04 – Attribute Mappings: Okta to Okta Workflows

## What is this?
This lab was designed to test profile attribute mappings between the Okta 
Universal Directory and the Okta Workflows application using the 
Profile Editor.

## Sandbox Limitation
The cohort sandbox environment does not include access to Okta Workflows, 
which is required to complete the mapping verification steps. This was 
confirmed with the Okta learning cohort community — it is a known 
environment constraint, not a configuration error.

---

## What the lab covers

The objective was to verify and test attribute mappings from Okta User 
profiles to Okta Workflows using the Profile Editor:

| Okta User Profile | Okta Workflows Profile |
|---|---|
| user.firstName | givenName |
| user.lastName | familyName |
| user.email | email |
| Role-based expression | primaryRole |

The primaryRole mapping uses Okta Expression Language to assign roles 
dynamically based on admin permissions — Super Org Admin, Workflows 
Admin, or Member.

### Two mapping trigger modes tested in this lab:
- **Apply mapping on user create and update** — keeps downstream app 
in sync with every profile change
- **Apply mapping on user create only** — one-time sync at provisioning, 
no live updates

---

## Why this matters
Attribute mapping is how identity data flows from your source of truth 
(Okta) to downstream applications. Getting the trigger mode wrong causes 
stale data in connected apps — a real operational problem in enterprise 
environments. This concept applies directly to SCIM provisioning and 
app integrations at scale.

---

## Next Step
This lab will be revisited if a Workflows-enabled sandbox becomes 
available. The mapping concepts documented here will be applied 
practically in Project 2 when configuring application integrations 
and automated provisioning workflows.
