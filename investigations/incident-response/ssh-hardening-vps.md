Production VPS — SSH Security Hardening and Access Control
Findings
A review of SSH authentication and access controls, including journalctl analysis, identified:
•	Repeated unauthorised SSH login attempts targeting root.
•	Direct root SSH access using password authentication was enabled.
•	No dedicated administrative account was configured.
•	Password-based authentication increased exposure to brute-force attacks.
•	Routine use of root reduced privilege separation and accountability.
Risk
The configuration increased the potential impact of a successful compromise:
•	Confidentiality: Root compromise could expose sensitive system and application data.
•	Integrity: An attacker could modify system files, configurations and security controls.
•	Availability: Root access could allow services or system configurations to be disabled or altered.
•	Accountability: Direct root administration reduced traceability of individual administrative actions.
Remediation
1. Dedicated Administrative Account
•	Created a dedicated administrative user.
•	Granted controlled sudo privileges.
•	Removed the need for routine direct root access.
2. SSH Key Authentication
•	Generated an SSH key pair on the trusted administration workstation.
•	Configured the public key in authorized_keys.
•	Applied restrictive permissions:
o	.ssh — 700
o	authorized_keys — 600
•	Verified correct file ownership.
3. SSH Hardening
Configured SSH to enforce:
PermitRootLogin no
AllowUsers <username>
PasswordAuthentication no
This prevents direct root login, restricts SSH access to the authorised account, and disables password-based authentication.
Configuration Validation
Initial sshd -t validation identified a configuration error caused by a typo. The configuration was corrected and successfully revalidated before restarting the service.
During subsequent verification, sshd -T showed that PasswordAuthentication was still enabled despite the intended configuration. Further investigation identified an SSH drop-in configuration overriding the setting.
A dedicated 01-hardening.conf configuration was therefore created and included in drop-in directory so that it will gets read first because 01- sorts before 50-, ensuring the intended settings were applied according to OpenSSH's first-value-wins behaviour.
Verification
To prevent accidental loss of access, the existing SSH session was kept open during the hardening process.
The following were verified:
•	sshd -t completed successfully.
•	Effective configuration was checked using sshd -T.
•	A live SSH connection was tested using the administrative account and key.
•	sudo access was confirmed.
•	Password-based authentication was tested to confirm the intended restriction.
•	Direct SSH authentication as root was tested and confirmed to be unsuccessful
•	The correct SSH service and service-management configuration were identified rather than assumed, accounting for Linux distribution differences.
Lessons Learned
•	Validate before applying: Use sshd -t before restarting SSH.
•	Maintain a recovery path: Keep an existing administrative session open during changes.
•	Apply least privilege: Use a dedicated account with sudo rather than direct root access.
•	Prefer key authentication: Reduce reliance on passwords for remote administration.
•	Verify effective configuration: Configuration files do not always represent the final active settings; use sshd -T and investigate drop-ins.
•	Perform live verification: A configuration check alone does not prove the security control is working. Test the actual SSH connection and authentication behaviour.
•	Environment matters: Service names, configuration paths and management procedures can vary between Linux distributions.
Key takeaway: Always perform both configuration-level validation and live functional testing after security changes. This confirms what is configured and, more importantly, what is actually enforced.

