+++
date = "2025-12-08"
title = "Everything WMI"
+++


Let's get into the goldmine that is WMI...

Windows Management Instrumentation (WMI) is the infrastructure for management data and operations on Windows-based operating systems. It is used by developers and system administrators to manage and interact with system internals. This also means that it's abused by attackers in order to move laterally throughout a system, establish fileless persistence and store payloads in memory.

### Communication

Traditionally, WMI queries used DCOM via RPC (Remote Procedure Call) which utilized random high ports (*Make yourself at home!* ). WinRM (Windows Remote Management) was later introduced, this time with only ports 5985 (HTTP) and 5986 (HTTPS)  being exposed by default.


Remote queries are executed using WinRM as a protocol to transport requests and responses. Local queries are handled via COM (Component Object Model) in order to interact with WMI providers.
## Core Components:

#### WMI Providers:

A WMI provider is a COM object that provides data to WMI or performs actions on behalf of WMI consumers. They are made up of a DLL file and a Managed Object Format (MOF) file, which are stored in %WINDIR%\System32\Wbem. These providers expose a specific set of classes/instances to consumers and communicate with WMI by using the COM based WMI Provider API.

#### WMI Infrastructure:

The WMI infrastructure also known as the WMI service (winmgmt), is made up of the WMI Core and WMI repository. The WMI repository stores WMI class definitions, namespaces, static configuration data while the WMI Core is the runtime engine that receives queries, invokes providers  and manages data.

Although the WMI service creates some namespaces at system startup, most are created by applications and drivers which makes it important to monitor, such that malicious classes are not created under obscure namespaces.

#### WMI Consumers: 

WMI consumers are any scripts/applications that query WMI or receive WMI events. This could be an EDR agent that is interacting with WMI to query data. The data available to consumers is only accessible via providers.

#### WmiPrvSE.exe:

WmiPrvSE.exe is the WMI Provider Host which is part of the WMI component. It is a host process that loads and executes WMI provider DLLs.

This means that unlike winmgmt.exe which receives requests, wmiprvse.exe enumerates processes and executes requests.

The WmiPrvSE.exe file is typically located in the directory `C:\Windows\System32\wbem`. In 32-bit systems, it can also be found in `C:\Windows\SysWOW64\wbem`.


### High CPU Usage:

Oftentimes, WMI gets mistaken for malware because of its high CPU usage from WmiPrvSe.exe,  which is often caused by "chatty" or misconfigured applications that use WMI to gather system data. These applications usually use WMI to pull information like running processes, service status, installed drivers etc

Now this can also mean that there's a malicious script that happens to be repeatedly executing commands but then again it is but only one IOC and should be thoroughly verified.

### Suspicious child processes & abnormal commands

From a detection standpoint, all child processes from WmiPrvSe.exe should be monitored. Due to its nature, DLLs and multiple instances are continuously spawned making it a lucrative target for attackers to take advantage of. It is what many would call a [LOLBIN](https://lolbas-project.github.io/lolbas/Binaries/Wmic/).



**Example:** *Remote Command Execution* 

Running a WMI command such as `Invoke-WebRequest`, causes Powershell to call the WMI service which calls WmiPrvse.exe making it the parent process that is responsible for the execution.

`Powershell.exe -> Wmiprvse.exe -> cmd.exe`

1. Command is initiated via Powershell
2. Powershell interacts with WMI on target machine
3. WMI service receives request + command is executed
4. cmd.exe is launched to execute command
5. WMI service sends back response to local machine

### Persistence via Event Subscriptions

Event subscriptions allow specific events to be monitored and notified by executing actions automatically when specific events occur.

In order to do so, your subscription must be composed of:

1. Event filter - Criteria that must be met for event to trigger
2. Event Consumer - Action that occurs when conditions are met
3. Binding - Ties the filter to the consumer, ensures that the consumer is executed once the filter condition is met.

WMI event subscriptions are not saved to a permanent location on the disk, making it harder for payloads to be detected by security tools. They can however survive reboots, a mechanism which allows subscriptions to triggered right after system start up making it easier for persistence. This could be done using the WMI  `__InstanceCreationEvent`   to run a payload every time a new process/service is created. 

### Recon

Using the Win32_Process for recon would allow attackers to understand all of the processes running on a system.

`Get-CimInstance -ClassName Win32_Process`

To get information about the network for lateral movement they would want to use 
`Get-CimInstance -ClassName Win32_ComputerSystem` 


For user information in AD environments, WMI queries are usually combined with other techniques because WMI is not designed to have access to extensive information from AD and would require domain admin level privileges.

LDAP queries  are better suited for interacting with AD services and getting detailed user information whereas WMI queries are suited for information related to a computers local configuration.








