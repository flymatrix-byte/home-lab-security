PowerShell Script Block Logging
Objective
PowerShell Script Block Logging addresses a key visibility gap in Windows security monitoring: identifying commands executed within PowerShell.
Previous testing with Get-Date, New-ItemProperty, and -EncodedCommand showed that the commands were not consistently recorded in the available Security and Sysmon logs.
When PowerShell launches an external executable such as reg.exe to modify the registry, process creation can be recorded in Windows Security Event ID 4688 and Sysmon Event ID 1, provided the relevant auditing is enabled. These events may include the command line used.
However, when an attacker uses built-in PowerShell cmdlets, such as New-ItemProperty, the action may occur entirely within the PowerShell process. Without suitable PowerShell logging, the specific command and its arguments may not be recorded.
Our Sysmon configuration also did not monitor the relevant PowerShell logging policy registry keys, creating an additional detection gap.
1. Testing the Existing Logging Configuration
To establish the current logging coverage, we executed the following command in PowerShell:
Get-Date
We then examined the available logs:
•	Windows Security logs: No event recording the specific command was identified.
•	Sysmon: No event recording the specific command was identified.
•	PowerShell Operational logs: No Event ID 4104 was found for this execution.
This confirmed that the existing configuration did not provide the expected visibility into PowerShell command execution.
Finding: Without PowerShell Script Block Logging, defenders may be unable to reconstruct commands executed through built-in cmdlets, even when process creation auditing is enabled.
2. Enabling Script Block Logging
We enabled PowerShell Script Block Logging by setting the relevant registry policy using an elevated PowerShell session:
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" /v EnableScriptBlockLogging /t REG_DWORD /d 1 /f
This sets EnableScriptBlockLogging to 1, enabling the policy for supported PowerShell sessions.
When enabled, PowerShell records script block content in the PowerShell Operational log, primarily under Event ID 4104.
These events can provide visibility into commands and script content that may not be apparent from process creation events alone.
3. Testing the Configuration Change
After enabling the policy, we tested both existing and newly opened PowerShell sessions.
Existing PowerShell sessions
We executed Get-Date in a PowerShell session that had already been running before the policy change.
No corresponding Event ID 4104 was identified for that execution.
This demonstrated that the policy change did not immediately provide the expected logging in the existing session.
New PowerShell sessions
We opened a new PowerShell session and executed Get-Date again.
This time, the command was recorded in the PowerShell Operational log as Event ID 4104.
The result confirmed that Script Block Logging was working in the newly opened session.
Key finding: Logging configuration changes should be tested against both existing and newly created sessions. Starting a new session is a useful validation step after enabling the policy.
4. Unexpected Logging Behaviour
During testing, we also found that commands used to enable Script Block Logging and subsequently check its configuration appeared in the logs with a warning-level classification.
This was unexpected because we initially assumed that commands executed because the settings were not implemented yet.
The result highlights the importance of checking the actual event details rather than relying solely on assumptions about logging behaviour.
The warning-level classification does not, by itself, establish that an action was malicious. Event content, timestamps, event IDs, and execution context must be examined to understand why the events were generated.
5. Detection Gap: PowerShell Logging Policy Tampering
Our Sysmon configuration did not include rules to monitor the PowerShell Script Block Logging policy registry keys.
This creates a potential detection gap.
For example, an administrator-level attacker could attempt to disable Script Block Logging by modifying the relevant registry policy. If the modification is performed through a PowerShell cmdlet such as New-ItemProperty, and the operation is not covered by the active logging configuration, Sysmon may not record the registry change.
Process creation logging alone would not necessarily reveal the specific cmdlet or registry modification.
Recommended improvements:
•	Enable PowerShell Script Block Logging through Group Policy or an equivalent managed configuration.
•	Monitor changes to the PowerShell logging policy registry keys.
•	Review Sysmon registry monitoring rules to ensure relevant modifications are captured.
•	Forward PowerShell Operational and Windows Security logs to a central logging system.
•	Alert on unexpected changes that disable or weaken PowerShell logging.
6. Correlating PowerShell Event ID 4104 with Sysmon Event ID 1
The next step was to correlate PowerShell Script Block Logging events with Sysmon process creation events.
6.1 PowerShell Event ID 4104
The Get-Date execution was recorded in Event ID 4104 with the following identifiers:
•	Process ID: 4572
•	Thread ID: 4888
The same process and thread identifiers appeared in the related 4104 events associated with the execution.
An additional event associated with the PowerShell prompt showed different identifiers:
•	Process ID: 332
•	Thread ID: 1016
This suggested that the prompt-related event was associated with a different execution context. However, the identifiers alone are insufficient to establish the complete process relationship.
6.2 Sysmon Event ID 1
Sysmon Event ID 1 recorded the creation of powershell.exe.
The event contained a command line showing an encoded command, represented in the available logs as:
C:\Windows\...\powershell.exe -EncodedCommand ...
The process identifier was 4572, matching the Process ID observed in the relevant PowerShell Event ID 4104 records.
This provided a useful correlation between the process creation event and the PowerShell script block activity.
6.3 Correlation Findings
Matching the process identifiers helped establish that the Sysmon process creation event and the relevant PowerShell Operational events were associated with the same PowerShell process.
However, process IDs can be reused, and thread IDs identify threads rather than parent-child process relationships. Reliable correlation should also consider:
•	Event timestamps.
•	Process creation time and process lifetime.
•	Sysmon Process GUID.
•	PowerShell process ID and script block content.
•	Parent process information and command-line arguments.
the child’s Sysmon 1 ParentProcessId is 332, so the prompt event came from the parent window.
7. Conclusion
This lab demonstrated the importance of PowerShell Script Block Logging for improving visibility into command execution.
After enabling the policy, a newly opened PowerShell session generated Event ID 4104 for the command Get-Date.
The investigation also demonstrated how PowerShell Event ID 4104 can be correlated with Sysmon Event ID 1 to connect script execution with process creation.
The remaining concern is the ability to modify or disable logging without generating sufficient evidence. Monitoring PowerShell logging policy changes, reviewing Sysmon registry rules, and centralising logs are important steps towards reducing this detection gap.

