# Production VPS — SSH Security Hardening and Access Control

## Incident / Hardening Note

### Findings

A review of the authentication and authorisation configuration of the production VPS was conducted, including analysis of SSH access activity using `journalctl`. The following findings were identified:

1. **SSH brute-force activity:** Multiple unauthorised SSH login attempts targeting the `root` account were identified.
2. **Direct root authentication:** Direct SSH access to the server as `root` using password-based authentication was enabled.
3. **No dedicated administrative account:** No separate administrative user was configured for routine system administration, resulting in the privileged `root` account being used for daily administrative activities.
4. **SSH service configuration:** The existing SSH configuration permitted password-based authentication and direct remote access to a highly privileged account.

### Risk Assessment

The identified configuration presented risks across the **Confidentiality, Integrity and Availability (CIA) triad**:

* **Confidentiality:** Successful compromise of the root password could provide an attacker with unrestricted access to sensitive system information and potentially any data accessible from the server.
* **Integrity:** Routine use of the `root` account increased the potential impact of accidental or malicious changes to system files, configurations, applications and security controls.
* **Availability:** Repeated brute-force attempts generated unnecessary authentication activity and increased exposure of the SSH service to resource exhaustion or denial-of-service conditions. More importantly, compromise of a root account could allow an attacker to stop services, modify configurations or otherwise disrupt system availability.
* **Credential exposure:** Password-based authentication increased the attack surface compared with SSH public-key authentication, particularly for a privileged account exposed to the internet.
* **Privilege separation:** The absence of a dedicated administrative account reduced accountability and separation of privileges for administrative activities.
* **Accountability:** Performing routine activities directly as `root` made it more difficult to attribute administrative actions to a specific individual account.

### Remediation

The following SSH security hardening measures were implemented.

#### 1. Dedicated Administrative Account

* Created a dedicated user account for routine server administration.
* Granted the account `sudo` privileges to perform tasks requiring elevated privileges.
* Removed the requirement to use `root` directly for routine administrative activities.

#### 2. SSH Key-Based Authentication

* Generated an SSH key pair on the trusted administration workstation.
* Installed the public key in the new user's `authorized_keys` file.
* Configured the SSH key directory and authentication file with restrictive permissions:

  * `.ssh`: `700`
  * `authorized_keys`: `600`
* Verified that the files were owned by the appropriate administrative user.

#### 3. SSH Service Hardening

The SSH daemon configuration was modified to enforce restricted administrative access:

```text
PermitRootLogin no
AllowUsers <username>
PasswordAuthentication no
```

These controls:

* Prevent direct remote SSH login as `root`.
* Restrict SSH access to the explicitly authorised administrative account.
* Disable password-based SSH authentication.
* Require possession of the corresponding private SSH key for remote authentication.

This reduces exposure to password-based brute-force attacks and follows the principle of **least privilege** by separating routine user access from privileged operations.

### Verification

To reduce the risk of losing administrative access during the hardening process, the existing SSH session was intentionally kept open throughout the configuration and verification process.

#### 1. SSH Configuration Validation

The SSH daemon configuration was validated before restarting the service:

```bash
sudo sshd -t
```

An error caused by a configuration typo was identified during validation. The configuration was corrected and the validation was repeated successfully before proceeding.

This demonstrated the importance of validating configuration changes before applying them to a production service.

#### 2. SSH Service Verification

After the configuration passed validation:

* The SSH service was restarted.
* A new SSH session was established using the newly created administrative account and SSH key.
* Successful key-based authentication was confirmed.
* `sudo` access was tested to confirm that administrative privileges could be obtained when required.
* Direct SSH authentication as `root` was tested and confirmed to be unsuccessful.
* Password-based SSH authentication was tested and confirmed to be disabled.

### Service Identification

During the hardening process, it was identified that the SSH service name can vary between Linux distributions and system configurations.

Rather than assuming the service name, the system was inspected to determine the correct SSH service and service-management configuration for the specific VPS.

This reinforced the importance of **environment-specific verification** when administering Linux systems, particularly across different distributions, versions and service-management implementations.

### Lessons Learned

The hardening process reinforced several important security engineering practices:

* **Validate before applying:** Configuration changes should be tested before restarting production services. Running `sshd -t` identified a configuration error before it could potentially disrupt SSH access.
* **Maintain a recovery path:** Keeping the existing administrative session open during SSH configuration changes provided a recovery path if the new configuration prevented authentication.
* **Use least privilege:** Routine administration should be performed through a dedicated user account, with `sudo` used only when elevated privileges are required.
* **Prefer key-based authentication:** SSH public-key authentication reduces reliance on passwords for remote administrative access.
* **Verify rather than assume:** Service names, configuration paths and management commands can differ between Linux distributions and versions. The actual environment should be investigated before applying changes.
* **Test the security controls:** Hardening is not complete until the intended restrictions have been tested, including both successful legitimate access and unsuccessful unauthorised access.
