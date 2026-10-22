Sysmon Installation and Registry Persistence Detection
1. Purpose
By default, Windows does not log all the events defenders need for effective security monitoring and investigation. Windows Security logs also have limitations in the level of detail provided for certain events.
Microsoft Sysmon (System Monitor) provides additional system activity logging to support threat detection and incident investigation. It records detailed information about processes, registry changes, and other system events, depending on the configured rules.
Sysmon configuration files allow defenders to include or exclude specific events, helping them focus on relevant activity and reduce unnecessary logs.
This lab explores Sysmon installation, process auditing, and registry persistence detection. It also compares Sysmon events with Windows Security Event ID 4688 (process creation).
Important information provided by Sysmon includes:
•	Process hashes, which help verify file integrity and identify known files.
•	Company, product, and file version information.
•	Process GUIDs, which help correlate events involving the same process.
•	Parent process command lines, which provide additional context about process execution.
2. Configuration and Testing
2.1 Installing Sysmon
Sysmon was downloaded from the official Microsoft Sysinternals website:
Download Sysmon
Installation was performed using an elevated PowerShell session. The following command installs Sysmon and applies the specified configuration file:
.\Sysmon64.exe -i .\config.xml
The configuration file determines which events Sysmon records.
2.2 Verifying the Service
After installation, the Sysmon service was checked to confirm that it was running:
Get-Service Sysmon64
The service status showed Running, confirming that Sysmon was active.
2.3 Verifying File Integrity
The SHA-256 hash of a downloaded file can be calculated using PowerShell:
Get-FileHash -Path "C:\Tools\Sysmon64.exe" -Algorithm SHA256
The calculated hash can then be compared with a trusted reference hash, if one is available, to verify file integrity.
Note: Calculating a file hash alone does not establish that the file is genuine. The hash must be compared against a trusted reference. The hash of the executable should also be distinguished from the hash of the configuration file.
3. Registry Persistence Detection
3.1 Creating a Registry Run Key
Windows allows applications to configure automatic execution at user logon through registry Run keys. Attackers can abuse this mechanism to establish persistence.
The following command creates a Run key entry that configures Notepad to start automatically when the current user logs on:
New-ItemProperty `
  -Path "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run" `
  -Name "SysmonTest" `
  -Value "C:\Windows\System32\notepad.exe" `
  -PropertyType String
This was performed in a controlled lab environment to test Sysmon's registry monitoring capabilities.
3.2 Comparing Sysmon with Windows Security Logs
After creating the registry entry, Sysmon logs were examined for evidence of the change.
Sysmon recorded an event associated with the registry modification, while no corresponding event was observed in the Windows Security logs during this test, even with process creation auditing enabled.
This demonstrates an important distinction:
•	Windows Security Event ID 4688 records process creation when the relevant auditing policy is enabled.
•	Sysmon registry events can record registry modifications that may not be visible through process creation auditing alone.
Registry changes should therefore be monitored directly rather than inferred only from the processes that execute them.
3.3 Testing Configuration Filters
The Sysmon configuration file was examined to understand how its inclusion and exclusion rules affect event collection.
As an additional test, a registry key was created under HKCU:\Software\SysmonLab.
No corresponding event was observed in Sysmon for this action. This was consistent with the effective configuration not collecting that particular registry operation.
This highlights the importance of reviewing configuration rules carefully. An event that is excluded by the configuration will not be available for investigation through Sysmon.
3.4 Investigating Additional Process Activity
After creating the Run key, the logs showed activity involving sihost.exe.
Further investigation identified sihost.exe as the Windows Shell Infrastructure Host. Its activity coincided with a Windows notification indicating that Notepad had been configured to run at startup.
This demonstrates why investigators should examine related process activity rather than focusing exclusively on the command that initiated a registry change.
The presence of a process after a registry modification does not, by itself, establish a causal relationship. Process details, timestamps, parent-child relationships, and other supporting evidence should be reviewed before drawing conclusions.
4. Uninstalling Sysmon
The official Sysmon uninstall option was reviewed. The following command removes Sysmon from the system:
.\Sysmon64.exe -u
Sysmon should only be removed when required for legitimate administration or testing. Unexpected service removal or termination should be investigated because it may indicate an attempt to disable security monitoring.
5. Sysmon Compared with Windows Security Event ID 4688
Both logging sources provide useful information, but they serve different purposes.
Information	Sysmon	Security Event ID 4688
Process creation	Yes, Event ID 1	Yes
Process image hash	Available in Event ID 1 when supported by configuration	Not normally included
Company and product information	Available in process metadata	Not normally included
File version information	Available in process metadata	Not normally included
Process GUID	Available	Not provided in the same format
Parent process command line	Available in Event ID 1 when collected	Depends on available fields and auditing configuration
Elevated token information	Not provided in the same format as Event ID 4688	Available through the Token Elevation Type field when applicable
Sysmon provides richer process context, which can help investigators identify suspicious executables, correlate activity, and reconstruct process relationships.
Windows Security Event ID 4688 can provide token elevation information useful for determining the security context in which a process was created.
Conclusion: Sysmon and Windows Security auditing should complement each other. Neither source should be treated as a complete replacement for the other.
6. Detection Opportunities
6.1 Unexpected Registry Persistence
Monitor registry modifications involving common startup locations, including:
•	HKCU\Software\Microsoft\Windows\CurrentVersion\Run
•	HKLM\Software\Microsoft\Windows\CurrentVersion\Run
•	Other startup-related registry locations covered by the monitoring configuration.
Investigate changes made by unexpected processes, especially when the configured executable runs from a suspicious location or uses unusual command-line arguments.
6.2 Sysmon Configuration Changes
Sysmon Event ID 16 records Sysmon configuration changes.
Investigate unexpected configuration changes, particularly when the configuration hash changes or the effective rules reduce visibility into important activity.
A hash change is not automatically malicious, as legitimate configuration updates can also change the hash.
6.3 Sysmon Service Stoppage
Monitor for unexpected Sysmon service termination or removal.
When Sysmon stops recording events, subsequent activity may no longer be visible through its event channel. Investigate service status, relevant Windows service-control events, and other available telemetry to determine why logging stopped.
6.4 Process and Registry Correlation
Correlate registry modification events with process creation events to establish:
•	Which process modified the registry.
•	Which user account performed the activity.
•	Whether the process and its parent are expected.
•	Whether the registry value points to a legitimate executable.
•	Whether the change coincides with other suspicious activity.
A registry Run key modification is not inherently malicious. Detection decisions should consider the process, user, registry value, timing, and surrounding activity.
7. Limitations
7.1 Historical Visibility
Sysmon records activity only after it has been installed and started. It cannot retrospectively generate events for activity that occurred before installation.
Windows Security logs and other existing telemetry may provide historical evidence, but their availability depends on the auditing configuration and log retention.
For this reason, Sysmon should be deployed and configured as early as practical when building a Windows server.
7.2 Configuration-Dependent Visibility
Sysmon only records events covered by its effective configuration.
Registry operations that are excluded or not covered by the applicable rules will not be available in Sysmon logs. A lack of recorded events does not necessarily mean that an action did not occur.
Configuration testing should confirm that important event types are actually collected.
7.3 PowerShell Script Visibility
PowerShell Script Block Logging was not tested in this lab.
As a result, the commands or script content executed within PowerShell may not be fully visible through the telemetry examined here.
Additional PowerShell logging, such as Script Block Logging and Module Logging, may be required to improve visibility into script execution. These features should be configured and tested separately.
8. Security Considerations
8.1 Tampering with Sysmon
An attacker with sufficient administrative privileges may attempt to stop Sysmon, uninstall it, or modify its configuration to reduce security visibility.
Defenders should monitor for unexpected service termination, removal, and configuration changes. Where possible, forward security logs to a separately managed central logging system to reduce the risk of local tampering affecting all available evidence.
8.2 Log Access and Confidentiality
Sysmon events may contain sensitive operational information, including usernames, file paths, command-line arguments, and process details.
Access to event logs should be restricted to authorised personnel. Log retention, forwarding, and access controls should be configured to support investigation requirements while protecting sensitive information.
8.3 Configuration Integrity
Sysmon configuration files should be maintained through a controlled change process. Configuration changes should be authorised, documented, and verified.
Monitoring configuration changes helps identify unexpected modifications that could weaken detection coverage.
9. Conclusion
This lab demonstrated how Sysmon can improve Windows security monitoring by recording detailed process and registry activity.
The registry Run key test highlighted the value of monitoring registry changes directly rather than relying solely on process creation logs. The comparison with Windows Security Event ID 4688 also demonstrated that the two sources provide different, complementary information.
Effective detection depends on a tested configuration, reliable log collection, monitoring for tampering, and correlation with other available telemetry. Additional PowerShell logging is also worth investigating to improve visibility into command and script execution.

