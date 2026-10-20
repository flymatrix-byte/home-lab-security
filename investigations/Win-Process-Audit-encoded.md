Windows Process Auditing and Encoded Commands
1. Purpose
This document examines the default security settings of Windows Server 2022 Standard Edition to identify gaps in process auditing, explain their security impact, and document how to address them.
The testing focuses on process creation auditing, command-line logging, and identifying potentially suspicious or encoded commands.
2. Configuration Changes
2.1 Enable Process Creation Auditing
By default, process creation auditing was not enabled on the test machine. As a result, process creation events were not being recorded in the Security event log.
To enable successful process creation auditing, run the following command in an elevated Command Prompt or PowerShell session:
auditpol /set /subcategory:"Process Creation" /success:enable
This enables Windows to record successful process creation events as Event ID 4688 in the Security event log.
These events help identify which account started a process and when it was created.
2.2 Enable Command-Line Logging
Enabling process creation auditing records process creation events, but command-line arguments are not included by default.
To address this gap, enable the following registry setting:
Registry path:
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\Audit
Value:
ProcessCreationIncludeCmdLine_Enabled
Type: REG_DWORD
Data: 1
This setting enables command-line recording for process creation events. When available, Event ID 4688 includes the command line used to start the process.
Administrators can review these events in Event Viewer under:
Windows Logs → Security
Filter the log by Event ID 4688 to investigate process creation activity.
3. Testing and Observations
The following table summarises the commands tested and the observed results before and after enabling process creation auditing and command-line logging.
Command tested	Default configuration	After enabling process auditing	After enabling both settings
whoami /groups	No Event ID 4688 recorded	Event ID 4688 recorded	Event ID 4688 recorded with the command line
hostname	No Event ID 4688 recorded	Event ID 4688 recorded	Event ID 4688 recorded
Get-Date (run directly in PowerShell)	No Event ID 4688 recorded	No Event ID 4688 recorded	No Event ID 4688 recorded
Get-Date (run through powershell.exe)	No Event ID 4688 recorded	Event ID 4688 recorded	Event ID 4688 recorded with the command line
3.1 Key Findings
•	Process creation auditing was disabled by default. Enabling the Process Creation audit subcategory allowed successful process creation events to be recorded.
•	Command-line logging provided additional visibility. Enabling the registry setting allowed the command line to be included in applicable Event ID 4688 records.
•	PowerShell commands do not always generate separate process creation events. Commands executed within an existing PowerShell session, such as Get-Date, do not create a new operating system process. Therefore, they do not generate a separate Event ID 4688 event.
•	Launching PowerShell as a new process creates a process creation event. When powershell.exe starts, Windows can record the process and its supplied command-line arguments. This does not mean every command subsequently executed inside that session will be logged by Event ID 4688.
4. Detection Idea: Identifying Encoded Commands
Process creation logs provide useful visibility into how processes are launched and which command-line arguments they receive.
However, attackers may use encoded commands to make their activity harder to understand during an investigation. For example, PowerShell supports the -EncodedCommand parameter, which accepts a Base64-encoded command.
Instead of displaying the original command in readable form, the process creation event may contain the encoded string.
Detection approach
1.	Collect Event ID 4688 from Windows Security logs.
2.	Identify suspicious process executions, particularly PowerShell processes using parameters such as -EncodedCommand or -enc.
3.	Extract and decode the encoded command using the appropriate encoding, commonly Base64 with UTF-16LE for PowerShell's -EncodedCommand parameter.
4.	Review the decoded content to understand the command's purpose and determine whether the activity is legitimate or suspicious.
Important: Encoding alone does not prove malicious activity. Investigations should consider the command's content, parent process, user account, execution context, and other available telemetry.
5. Limitations
This testing was performed on a Windows Server 2022 Standard Edition virtual machine in a home lab. The findings represent the observed behaviour of this test environment and should not be treated as a complete assessment of all Windows logging configurations.
The following limitations apply:
•	Logon IDs and their relationship to user sessions were not fully investigated.
•	Testing focused on selected commands rather than a comprehensive range of process execution scenarios.
•	Event ID 4688 records process creation, not every command executed within an existing shell or scripting session.
•	Encoded commands may require additional detection logic to identify, extract, and decode suspicious content.
Further testing should validate these findings across different execution methods, user privileges, and logging configurations.
6. Security Considerations
Enabling command-line logging improves detection and investigation capabilities, but it introduces a potential exposure of sensitive information.
Command-line arguments may contain credentials, passwords, access tokens, API keys, or other sensitive data in plain text. If these values are included in process creation events, they may be stored in the Windows Security event log and become accessible to users or systems with sufficient log access.
If an attacker gains access to the machine or its logs, this information could be exposed.
Recommended safeguards include:
•	Restrict access to Security event logs.
•	Apply suitable retention and monitoring policies.
•	Avoid passing credentials or other secrets directly through command-line arguments.
•	Monitor suspicious process creation and encoded command execution.
Command-line logging should be enabled as part of a broader logging strategy, with appropriate controls to reduce the risk of exposing sensitive information.

