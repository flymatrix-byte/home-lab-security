# SSH Access Lockout — Investigation and Root Cause

## Investigation

Following changes to the SSH configuration, new SSH sessions became unavailable for all users. Initial troubleshooting focused on `sshd_config` and the SSH service.

SSH logs showed no configuration errors or rejected authentication attempts. Instead, connections resulted in a **timeout**, indicating that the traffic was likely being blocked before reaching `sshd`. An active SSH rejection would normally generate a response or authentication error rather than a connection timeout.

## Findings

* New SSH connections for both `root` and the administrative user timed out.
* No corresponding connection attempts were recorded by the SSH service.
* `ufw` confirmed that TCP port 22 was permitted.
* Investigation therefore shifted to other host-based security controls.

## Root Cause

The VPS had multiple security controls in place, including **UFW, iptables and Fail2Ban**.

Fail2Ban logs confirmed that repeated SSH connection attempts from the same source IP, generated during troubleshooting, had triggered the SSH protection rules. The source IP was temporarily banned for **60 minutes**.

The SSH service itself was therefore not the cause of the lockout. The connection was being blocked at the host security layer before reaching `sshd`.

An important observation was that by the time the root cause was identified, the **60-minute Fail2Ban ban had already expired**. This was confirmed by reviewing the Fail2Ban logs, which showed that the IP had subsequently been unbanned.

## Lessons Learned

This incident reinforced the importance of investigating the **entire network and security stack**, rather than focusing only on the service being troubleshot.

A connection timeout can indicate that traffic is being silently dropped before reaching the target service. Future troubleshooting will therefore follow the path:

**Network → Firewall → Security Controls → Service → Authentication**

The incident also demonstrated that troubleshooting activities can unintentionally trigger security controls and create secondary issues unrelated to the original configuration problem.
