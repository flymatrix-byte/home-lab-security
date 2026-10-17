# Windows Logon Attack Detection

## 1. Brute-Force Attack Against a Single Account

A brute-force attack occurs when an attacker tries multiple passwords against the same account.

**Detection indicators:**
- Multiple failed logon attempts within a short period.
- The failures come from the same source and target the same account.
- The events contain the following status codes:
  - **Event ID:** `4625`
  - **Status:** `0xC000006D` — Logon failure.
  - **SubStatus:** `0xC000006A` — The username is correct, but the password is incorrect.

**Detection threshold:** For this investigation, flag at least **5 failed logon attempts within 10 minutes** against the same account from the same source. This threshold allows some tolerance for legitimate users making typing mistakes or forgetting their passwords.

## 2. Username Enumeration Detection

Username enumeration occurs when an attacker attempts to identify valid accounts on a system, potentially preparing for a future brute-force attack.

**Detection indicators:**
- Multiple failed logon attempts using different usernames.
- The attempts originate from the same source within a short period.
- The events contain the following substatus:
  - **SubStatus:** `0xC0000064` — The specified username does not exist.

This differs from `0xC000006A`, which indicates that the username is valid but the password is incorrect.

**Detection threshold:** For this investigation, flag at least **10 attempts using distinct invalid usernames within 10 minutes** from the same source. This indicates that someone may be trying different username combinations, which is unlikely during normal user activity.

## 3. Detection Limitations and False Positives

These detection rules have limitations and may produce false positives or miss genuine attacks.

**False positives:**
- A legitimate user repeatedly enters an incorrect password.
- A new user tries different username variations because they are unsure of their account name.

These activities may resemble brute-force attacks or username enumeration.

**Missed attacks:**
- An attacker makes only two or three attempts within the detection window.
- An attacker uses different accounts or password combinations to avoid triggering the threshold.
- An attacker distributes attempts across multiple source IP addresses.
- Logon auditing is disabled, preventing the collection of relevant failed logon events.

Another possible missed attack is **password spraying**, where an attacker tries one or two passwords against many accounts. No single account may reach the threshold, making the attack difficult to detect.

These limitations mean that thresholds should be treated as investigation triggers rather than definitive proof of an attack.

## 4. Detecting Successful Attacks

After detecting suspicious failed logon attempts, investigate whether the attacker subsequently gained access.

The investigation must correlate events using the same relevant attributes, including the source address, target account, logon type and time window.

### 4.1. Successful Username Enumeration

Username enumeration is successful when the attacker identifies a valid account.

A common indicator is a sequence of failed logon events containing:
- Initial attempts with SubStatus `0xC0000064`, indicating invalid usernames.
- A subsequent attempt with SubStatus `0xC000006A`, indicating that the username is valid but the password is incorrect.

The second event suggests that the attacker has identified a valid username. However, this does not prove enumeration was successful on its own. The events must be correlated, and the result should be treated as supporting evidence.

### 4.2. Successful Brute-Force Attack

A successful brute-force attack occurs when the attacker guesses the correct password and gains access.

Unlike failed attempts, successful logons have a different event ID and generally do not contain `Status` or `SubStatus` fields.

**Detection steps:**

1. Look for a subsequent successful logon event, **Event ID `4624`**.
2. Correlate the events using the source address, target account, logon type and relevant time window.
3. Investigate whether the successful logon is related to the preceding failed attempts.

A successful logon following repeated incorrect-password attempts may indicate a successful brute-force attack. However, additional investigation is required to distinguish an attack from a legitimate user who eventually entered the correct password.
