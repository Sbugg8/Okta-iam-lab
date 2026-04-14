# Lab 02 – Create Users in Okta

## What is this?
In this lab, I created a new user account in Okta, populated the user 
profile with enterprise attributes, activated the account through the 
email flow, and verified lifecycle events in the system log.

## Why does it matter?
User provisioning is one of the most fundamental IAM tasks in any 
enterprise. This lab simulates onboarding a new employee — understanding 
the activation flow, profile attributes, and lifecycle logging is critical 
for both day-to-day IAM operations and compliance auditing.

---

## What I built/configured

### Step 1 – Create the User Account
Created a new user account for Nina Shah in Directory > People without 
setting a password, triggering the activation email workflow.

![Create User Nina Shah](Lab2%20Create%20users%20in%20Okta/create-user-nina-shah.png)
*New user creation form for Nina Shah. No password set — forces 
activation via email, which is the secure default for real onboarding.*

### Step 2 – Verify Pending Status
After saving, verified the account status was Pending user action — 
confirming the activation email was sent and awaiting user response.

![Pending Status](Lab2%20Create%20users%20in%20Okta/nina-shah-pending-status.png)
*Nina Shah's account showing Pending user action status immediately 
after creation — activation email has been sent.*

### Step 3 – Populate Profile Attributes
Edited Nina Shah's profile to add enterprise-level attributes including 
title, department, user type, cost center, organization, and region.

![Edit Attributes](Lab2%20Create%20users%20in%20Okta/nina-shah-edit-attributes-and-set-values.png)
*Profile tab showing attribute fields being populated for Nina Shah.*

![Added Attributes Continued](Lab2%20Create%20users%20in%20Okta/nina-shah-added-attributes-cont.png)
*Additional attributes configured including Region set to AMER — 
using the custom attribute created in Lab 01.*

### Step 4 – Verify Active Status
After Nina Shah completed the activation flow and set up Okta Verify, 
the account status updated to Active.

![Active Status](Lab2%20Create%20users%20in%20Okta/nina-shah-active-status.png)
*Admin console confirming Nina Shah's status is Active after successful 
account activation and MFA setup.*

### Step 5 – View Lifecycle Activity in System Log
Reviewed the system log under Reports > User lifecycle activity to 
verify all events were captured for audit purposes.

![Lifecycle Activity](Lab2%20Create%20users%20in%20Okta/nina-shah-lifecycle-activity.png)
*System log showing Create Okta user and Activate Okta user events — 
both confirmed SUCCESS for Nina Shah.*

---

## What I learned
- Not setting a password at creation forces the user through email 
verification — the secure default for real employee onboarding.
- Okta captures every lifecycle event in the system log automatically — 
this is the audit trail used for SOC 2 and compliance reporting.
- The Region attribute populated here used the custom schema extension 
built in Lab 01 — showing how profile customization connects directly 
to user provisioning.
- Pending user action status means the ball is in the user's court — 
the admin has done their part.
