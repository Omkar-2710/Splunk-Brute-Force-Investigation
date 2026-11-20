
# Splunk SPL Queries – Brute Force Investigation

This file contains the 7 SPL queries I used in my Splunk project.

**Splunk Index:** `lab_auth`

## Query 1: Check Dataset Events

This query shows how many events are available for each Event ID.

```spl
index=lab_auth
| stats count by EventCode
| sort EventCode
```

## Query 2: Check Failed Login Events

This query shows failed login events along with details like username, source IP, login type and failure reason.

```spl
index=lab_auth EventCode=4625
| table _time Account_Name Source_Network_Address Source_Port Logon_Type Failure_Reason Status Sub_Status host
| sort - _time
| head 20
```

## Query 3: Find Accounts with More Failed Logins

This query shows which user accounts have the highest number of failed login attempts.

```spl
index=lab_auth EventCode=4625
| stats count AS Failed_Attempts BY Account_Name
| sort - Failed_Attempts
| head 15
```

## Query 4: Check Suspicious Source IP Addresses

This query shows which IP addresses are generating failed login attempts and how many different accounts they are trying to access.

```spl
index=lab_auth EventCode=4625
| stats count AS Failed_Attempts dc(Account_Name) AS Unique_Accounts values(Account_Name) AS Targeted_Accounts BY Source_Network_Address
| sort - Failed_Attempts
| head 15
```

## Query 5: Find Repeated Failed Login Attempts

This query checks for accounts with 8 or more failed login attempts within the same 10-minute time block.

```spl
index=lab_auth EventCode=4625
| bin _time span=10m
| stats count AS Failed_Attempts min(_time) AS First_Attempt max(_time) AS Last_Attempt BY _time Source_Network_Address Account_Name
| where Failed_Attempts >= 8
| eval First_Attempt=strftime(First_Attempt,"%Y-%m-%d %H:%M:%S")
| eval Last_Attempt=strftime(Last_Attempt,"%Y-%m-%d %H:%M:%S")
| rename _time AS Time_Bucket
| sort - Failed_Attempts
```

## Query 6: Failed Login Followed by Successful Login

This query checks whether a successful login happened after a failed login for the same user account and source IP address within 15 minutes.

```spl
index=lab_auth (EventCode=4625 OR EventCode=4624)
| sort 0 _time
| streamstats current=f last(EventCode) AS Previous_Event last(_time) AS Previous_Time BY Account_Name Source_Network_Address
| where EventCode=4624 AND Previous_Event=4625 AND (_time-Previous_Time)<=900
| eval Gap_Minutes=round((_time-Previous_Time)/60,2)
| eval Failed_Login_Time=strftime(Previous_Time,"%Y-%m-%d %H:%M:%S")
| table Account_Name Source_Network_Address Failed_Login_Time _time Gap_Minutes host
| sort - Gap_Minutes
```

## Query 7: Investigate a Specific User Account

This query shows the total failed and successful login attempts for a specific user account and source IP address. It also shows when the activity first started and last occurred.

```spl
index=lab_auth (EventCode=4625 OR EventCode=4624)
Account_Name="priya.shah"
Source_Network_Address="198.51.100.23"
| stats count(eval(EventCode=4625)) AS Failed_Logins
        count(eval(EventCode=4624)) AS Successful_Logins
        min(_time) AS First_Seen
        max(_time) AS Last_Seen
        BY Account_Name Source_Network_Address
| eval First_Seen=strftime(First_Seen,"%Y-%m-%d %H:%M:%S")
| eval Last_Seen=strftime(Last_Seen,"%Y-%m-%d %H:%M:%S")
```

## Important Information

- Event ID 4624 means a successful login.
- Event ID 4625 means a failed login.
- Event ID 4648 means a login attempt using explicit credentials.
- 900 seconds equals 15 minutes.
- The 8 failed login threshold is used for this practice project. It is not a fixed rule for every company.
- This project uses synthetic data for learning purposes.
- These queries help identify suspicious login activity. Further investigation is required before confirming a security incident.
