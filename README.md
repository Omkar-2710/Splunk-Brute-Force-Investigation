
# Brute Force Investigation Using Splunk SIEM

## 1. Project Overview

This is a hands-on SOC Analyst practice project using Splunk Enterprise.

The purpose of this project is to investigate suspicious login activities using Windows Security Event Logs.

## 2. Tools Used

- Splunk Enterprise
- SPL (Search Processing Language)
- Windows Security Event Logs

## 3. Dataset Information

- Dataset: Synthetic Windows Authentication Events
- Total Events: 7,384
- Splunk Index: lab_auth

| Event ID | Description | Event Count |
|---|---|---:|
| 4624 | Successful Login | 5,463 |
| 4625 | Failed Login | 1,896 |
| 4648 | Explicit Credentials | 25 |

## 4. Investigation Activities

- Uploaded and checked authentication events in Splunk.
- Analysed successful and failed login activities.
- Identified accounts with repeated failed login attempts.
- Investigated source IP addresses.
- Checked failed logins followed by successful logins.
- Documented suspicious login patterns.

## 5. Key Investigation Finding

During the investigation, I identified the following pattern in the synthetic dataset:

- 85 failed login attempts against one user account.
- One subsequent successful login from the same source IP.
- NTLM authentication with Logon Type 3.
- A 58-second gap between the last failed login and the successful login.

This pattern was consistent with possible brute-force activity followed by successful authentication. However, it does not confirm account compromise.

## 6. SPL Commands Practised

- search
- stats
- eventstats
- streamstats
- eval
- where
- sort
- table

## 7. Project Structure

- data: Contains the Windows authentication CSV dataset.
- queries: Contains the SPL investigation queries.
- screenshots: Contains screenshots of the Splunk investigation.

## 8. Important Note

This project uses synthetic data created for educational purposes. It does not represent a real-world security incident.

The external-looking IP addresses in the dataset are reserved documentation addresses and do not represent real attackers.

## 9. Learning Outcome

This project helped me understand how a SOC Analyst can investigate login activities, identify suspicious patterns, use SPL queries and document investigation findings.
