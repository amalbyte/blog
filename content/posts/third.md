+++
date = "2026-01-05"
title = "Abusing the ELF Format"
+++


### ELF (Executable and Linkable Format):

The ELF format defines the structure for executables, shared libraries and core dumps in Unix-like systems. From a detection-engineering perspective, it's important to understand not just how ELF files are _supposed_ to look, but also how malware authors commonly manipulate them to evade static analysis.

ELF files consist of:

1. ELF Header
2. Program Header Table (segments)
3. Section Header Table (sections)
4. Data

ELF file headers contain information about the file like the location of the program, type, entry point address etc. This helps the OS to understand how to load and execute the file.

Oftentimes, attackers tamper with the section header table, the section header defines all the sections contained in the file. Tampering with the section header does not break the malware because the data is not needed by the Linux loader in order to run the program, instead it makes it harder for analysts to understand the file structure because the tools are no longer able to classify the sections of data (functions, strings). At runtime, the OS loads segments (which consist of zero or more sections), defines which parts of the file are mapped into memory and at which virtual addresses, along with its permissions.

*Example:* 

Let's look at a YARA rule that detects binaries packed with UPX. UPX is an advanced executable file compressor that reduces the file size of programs and DLLs, although used for legitimate purposes it's also used for malicious purposes.

Because UPX is a packer (packing - a technique where contents of a binary file are compressed or encrypted to make the file smaller or to hide the real code until runtime) it's used by malware authors to obfuscate their binaries.

rule Detect_UPX_Packed_ELF

{
   
    `meta:`

        `description = "Detects ELF binaries packed with UPX"`

    `condition:`

        `uint32(0) == 0x7F454C46 and  // ELF magic`
        `for any s in (".upx0", ".upx1") : ( section.name == s )`
}


UPX packed binaries often have section names '.upx0' and '.upx1', because of this threat actors often remove the section headers or rename these sections to evade detection. So, if there are detection rules that are specifically focused on detecting these binaries via section names, they will ultimately pass through without being detected (weak detection, but it's fine for example purposes!)

`ELF section headers -> (tampered with)-> detection rule fails`

### Memory Mappings

The loader reads the program header table and maps segments into memory, therefore if the section headers and the metadata is tampered with we can investigate every running process to identify whether there are memory mappings which don't align with program headers. By inspecting `/proc/<pid>/maps` and comparing it with the live memory view with `readelf -l <binary>` we can observe whether the memory regions correspond with each other as dispensaries usually suggest runtime injection, reflective loading etc.


##### memfd_create Abuse:

The `memfd_create` system call creates an anonymous (no persistent file system entry and directory linkage), file object that behaves like a regular file that supports standard file operations. This allows the ELF loader to parse it normally and bypass file integrity monitoring while evading disk-based scanning.

1. Download ELF payload into memory
2. Write it to `memfd_create`
3. Replace current process image with new program (`execve()`)







*Sources:*

[1] hxxps://wiki.osdev[.]org/ELF

[2] hxxps://linux-audit[.]com/elf-binaries-on-linux-understanding-and-analysis/

[3] hxxps://nu11busters[.]github[.]io/rust-maldev-course/evasion/fileless-execution/

[4] hxxps://www[.]man7[.]org/linux/man-pages/man2/execve.2.html




