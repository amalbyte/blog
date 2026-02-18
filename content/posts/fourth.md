+++
date = "2026-02-01"
title = "Mimikatz & the Art of Exploitation"
+++

Mimikatz is a post-exploitation credential dumping tool that is used to extract credentials from the Windows Operating System. The tool was originally created to experiment with the security of authentication protocols and has since been used by security professionals and threat actors to bypass security controls.

### Credential dumping

*LSASS*

To perform credential dumping mimikatz leverages the Local Security Authority Subsystem Service (LSASS) which is responsible for the authentication of users and security policies. LSASS loads the appropriate authentication packages when a user authenticates into a system and retains it in memory to facilitate access to additional resources. Dependent on the protocol, this includes Kerberos granting tickets (TGTs), service tickets, NT password hashes, etc.

Mimikatz parses the LSASS memory structures for cached tickets, usernames and domains once privileged access is obtained via administrative or SYSTEM level privileges. The `sekurlsa` module is used to extract credentials stored in memory structures maintained by LSASS, this is either done with a dump of the LSASS memory (`sekurlsa::minidump <file>`) or directly.

*SAM*

The lsadump container module is used to target the Security Accounts Manager (SAM) windows registry hive that stores NT password hashes. `lsadump::sam`, reads data from the SAM and SYSTEM registry hives and extracts local account password hashes.

*DCSync* 

Mimikatz also provides the ability to abuse Active Directory replication mechanisms (`lsadump::dcsync`). Using the Directory Replication Service Remote Protocol (MS-DRSR), domain controllers synchronized data, replicated password hashes and request updates for directory objects.The tool uses an account that has the rights to replicate directory changes, authenticates to a domain controller, binds it to the DRSUAPI interface and then sends a request specifying target objects (accounts). Once successfully authenticated, the DC provides the requested data which can include Kerberos keys and NT hashes.

*ntds.dit*

The ntds.dit file is the core Active Directory database stored on domain controllers, it contains directory objects such as users, groups and the credentials that are used for authentication.To obtain access to the file, adversaries often create Volume Shadow Copies and copy the SYSTEM file (`_C:\Windows\System32\config\SYSTEM_`)from the registry to obtain the Boot Key that is needed to decrypt the ntds.dit file. 

##### Golden Ticket Attack

1. KRBTGT password hash extracted (`lsadump::lsa /inject /name:krbtgt`)
2. Custom TGT created

A golden ticket attack is a technique that is used by threat actors to forge custom TGTs after the extraction of a KRBTGT account from a DC. The KRBTGT account is a service account that is used by Kerberos to sign and encrypt every TGT in the domain. By compromising the KRBTGT hash, TGTs can be forged for any user.

##### Pass the Hash

This technique relies on the use of stolen NT hashes to authenticate into a system. NTLM authentication, simply relies on proof of knowledge of a hash for authentication without further verification. This allows the client computer to compute a response with the stolen hash to any challenge the server presents for authentication.

### Detection

*PowerShell logging*

1. `Invoke-Mimikatz`
2. `sekurlsa::logonpasswords`
3. `Invoke-Expression` (iex)
4. `Invoke-Kerberoast` 
5. `Add-Type` + `Base64` encoded string
6. `Start-Process <file path> WindowStyle Hidden` 


*Sysmon*

1. `Sysmon Event ID 10` (Process access) - monitor for unusual processes accessing LSASS memory
2. `Sysmon Event ID 1` (Process creation) - monitor for tools like mimikatz.exe, procdump.exe
3. `Sysmon Event ID 11` (File Creation) - the creation of .dmp files 
4. `Sysmon Event ID 8` (CreateRemoteThread) - mimikatz injection
5. `Sysmon Event ID 6` (Driver loaded) - unsigned or unusual drivers
6. `Sysmon Event ID 7` (Image loaded) - injected modules into LSASS, suspicious DLLs




Sources:

[1] hxxps://tools[.]thehacker.recipes/mimikatz/modules/sekurlsa/minidump

[2] hxxps://tools.thehacker.recipes/mimikatz/modules/lsadump/dcsync

[3] hxxps://jumpcloud.com/it-index/what-is-the-directory-replication-service-remote-protocol

[4] hxxps://book.hacktricks.wiki/en/windows-hardening/stealing-credentials/index.html

[5] hxxps://learn.microsoft.com/en-us/windows-server/security/windows-authentication/credentials-processes-in-windows-authentication

[6] hxxps://learn.microsoft.com/en-us/troubleshoot/windows-server/windows-security/ntlm-user-authentication

