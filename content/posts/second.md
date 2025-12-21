+++
date = "2025-12-21"
title = "Windows Forensics: Hidden Execution"
+++

There are various methods one could employ to hide the execution of commands, whether it be via process injection or simply hiding the console window. Luckily for us, there are also many artifacts we could utilize to put the pieces together. To start things off it would be good to look into application shimming.


## Application Shimming (T1546.011)

Application shimming is a Windows feature designed to maintain compatibility for older software.

*Process:* 

`Program starts - Shim is applied and loaded (if needed)` 

Adversaries abuse this future by creating malicious shims and linking them to legitimate programs.Shims are designed to run in user mode so they cannot directly change the Windows Kernel, administrator privileges are required to install a shim.

## Prefetch

Windows Prefetch files store a record of execution whenever a process is run. So, for every process that is executed, there is a .pf (Prefetch) file that is created. Parsing these files can be easily done with the use of Prefetch Explorer (PECmd.exe). These files contain:


`a. Executable name`

`b. Execution time: the first and last seven execution timestamps (total of 8)`

`c. Path of the application`

`d. Execution count`

`e. List of files/DLLs the application interacted with during the first 10 seconds` 

Prefetch files were created for the purpose of increasing the speed of application launches by caching the required resources to decrease the need for disk access. The naming convention of these files consist of the executed binary and a hash.

`CONHOST.EXE-B0F7681B.pf` 

As expected, the oldest files are removed first to make space once the maximum number of Prefetch files is reached.Deleting these files or the entire folder itself, generates more artifacts and is really noisy. The details of the deleted files can be found from system logfiles and is easily detectable by security tools. For this reason, it is generally recommended for the Prefetch directory to remain untouched. 

Specific to Windows XP (*outdated but interesting*):

 *If tampering with .pf files is of interest, stalling an application for more than 10 seconds (due to the mechanism of the cache manager) during startup will cause the captured information to be considerably less.* 

Later versions of Windows do not limit file access tracing to the first 10 seconds, and executables are monitored during the startup phase instead of a fixed duration.

## Shimcache

Shimcache, is part of the Application Compatibility system, which helps applications run on newer systems by storing metadata about executable files. Also known as "AppCompatCache", shimcache is an artifact that's used to track programs that have been executed on a system.

It is useful for learning about programs that were present on a system, the location of the program and the timestamp of the file's last modified time depending on the OS version. Shimcache entries *CANNOT* reliably be used to prove execution, the cache can only serve as an indicator of a program's existence because entries are also created when files are viewed, indexed, scanned, etc.

The cache is binary-encoded, viewing the contents can be done so with Forensic tools. Zimmerman's AppCompatCacheParser, was updated to reflect that execution can *sometimes* be indicated from the presence of an execution flag for some non-native binaries. Further research from nullsec also proves that additional artifacts are needed in order to rely on the execution value.

## Amcache

AmCache is a Windows artifact that stores the metadata of executable files and installed applications, often providing evidence of program execution.The main purpose of the AmCache is to enhance the performance and capability of programs running in different environments.

 AmCache.hve file is located at `C:\Windows\appcompat\Programs\Amcache.hve`



`Sources:`  

[1] nullsec[.]us/windows-10-11-appcompatcache-deep-dive/

[2] nullsec[.]us/appcompatcache-part-3/

[3] 13Cubed - # Let's Talk About Shimcache - The Most Misunderstood Artifact