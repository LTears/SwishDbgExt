#SwishDbgExt
===========

SwishDbgExt is a Microsoft WinDbg debugging extension that expands the set of available commands by Microsoft WinDbg, but also fixes and improves existing commands.
This extension has been developed by Matt Suiche (@msuiche) – feel free to reach out on support@comae.io ask for more features, offer to contribute and/or report bugs.

SwishDbgExt aims at making life easier for kernel developers, troubleshooters and security experts with a series of debugging, incident response and memory forensics commands.
Because SwishDbgExt is a WinDbg debugging extension, it means it can be used on local or remote kernel debugging session, live sessions generated by Microsoft LiveKd, but also on Microsoft crash dumps generated to a Blue Screen of Death or hybrid utilities such as Comae DumpIt.

## Installation
You can either copy the WinDbg extension in the corresponding (x86 or x64) WinDbg folder or load it manually using the !load command such as below. Please note you can’t have spaces or quotes in the full path to the target dll to be loaded.
`!load X:\FullPath\SwishDbgExt.dll`

###Example:
```
kd> !load E:\projects\labs\SwishDbgExt\bin\x64\SwishDbgExt.dll;
       SwishDbgExt v0.7.0 (Nov  2 2016) - Incident Response & Digital Forensics Debugging Extension
       SwishDbgExt Copyright (C) 2016 Comae Technologies FZE - http://www.comae.io
       SwishDbgExt Copyright (C) 2014-2016 Matthieu Suiche (@msuiche)

       This program comes with ABSOLUTELY NO WARRANTY; for details type `show w'.
       This is free software, and you are welcome to redistribute it
       under certain conditions; type `show c' for details.
```
	   
If you wish to update your WinDbg template with a more DML-friendly template, you can directly import windbg_template.reg file joined to the package.

## TODO
- [ ] Define structures
- [ ] Define Commands
- [ ] Announce feature contest.

## Commands
### !SwishDbgExt.help
Displays information on available extension commands.

This command will give you the list of all commands if you specify no argument, will give you the list of parameters for an existing command if specified as an argument.

### !ms_callbacks     
Display callback functions

### !ms_checkcodecave 
Look for used code cave

### !ms_consoles      
Display console command's history 

### !ms_credentials
Display user's credentials (based on gentilwiki's mimikatz) 
### !ms_drivers
Display list of drivers.
!ms_drivers will go ahead and display a list of drivers that are currently loaded.
In this example, here’s a few of the drivers loaded at the time of the crash in this kernel-dump:
With this command, we can also view in-depth IRP information regarding a driver:
In the above image we can see the driver-specific I/O stack location within e1cexpress.sys’ IRP. Here we can see function codes such as IRP_MJ_CREATE which opens the target device object, indicating that it is present and available for I/O operations.

### !ms_dump
Dump memory space on disk

### !ms_exqueue
Display Ex queued workers.

`!exqueue` doesn’t work properly on Windows 8, so a working version needed to be implemented. Just like the original command this one dispaly the working threads queue.

### !ms_fixit
Reset segmentation in WinDbg (Fix "16.kd>")

### !ms_gdt
Display GDT.

!ms_gdt displays the Global Descriptor Table. Note on x64 that every selector is flat (0x0000000000000000 to 0xFFFFFFFFFFFFFFFF). This command can be extra helpful to check for any suspected hooking of the GDT, as attempting to do so on x64 will call a bug check. This is because x64 forbids hooking of the GDT.

### !ms_hivelist
Display list of registry hives.

`ms_hivelist` displays a list of registry hives.
We can look directly into a hive (\Registry\Machine\Software for example) to see its subkeys, values, etc:

### !ms_idt
Display IDT.

`!ms_idt` displays the Interrupt descriptor table. Very much like the GDT, if the IDT is hooked on an x64 system, it will call a bug check. This is due to the fact that Microsoft implemented (programmatically) a prevention of hooking the IDT with a kernel-mode driver that would normally intercept calls to the IDT and then add in its own processing. This is why in the above image, there is ‘No’ as far as the eye can see.

### !ms_malscore
Analyze a memory space and returns a Malware Score Index (MSI) - (based on Frank Boldewin's work)

### !ms_mbr
Scan Master Boot Record (MBR)

### !ms_netstat
Display network information (sockets, connections, ...)

### !ms_object
Display list of object

### !ms_process
Display list of processes.
`!ms_process` is an improved version of `!process` and `!dml_proc`..
One of the nice thing as you can notice below is the usage of DML (Debugger Markup Language) with the commands. All the underline commands are in fact links to commands.
As an example below, you can see the output of /vads /scan, to scan VAD (Virtual Address Descriptors). You can notice that one column gives the “Malware Score Index” which can be useful to detect shellcodes or heap-spray.
In the screenshot below, you can see an abnormally high score in several VADs – due to usage of heap spray. Just by clicking on the score it will run the scanning algorithm.
The scanning algorithm is based on Frank Boldewin’s OfficeMalScanner utility.
And returns you information about where the shellcode is:
`/scan` option can also be used on exported functions to know if the EAT (Export Address Table) has been patched or if the prolog of the function modified.

Similar tests are available for the SSDT (`!ms_ssdt`).

### !ms_readkcb
Read key control block

### !ms_readknode
Read key node.
`!reg` WinDbg command has been a frustration for a long time, due to some bugs. This is why SwishDbgExt, has its own registry explorer functions to try to make access to registry data as simple as possible.

### !ms_readkvalue
Read key value

### !ms_scanndishook
Scan and display suspicious NDIS hooks

### !ms_services
Display list of services

### !ms_ssdt
Display service descriptor table (SDT) functions.
`!ms_ssdt` displays the System Service Dispatch Table. This command is extremely helpful in the investigation of suspected rootkit hooks through what is known as Direct Kernel Object Manipulation (DKOM). If you see a low level routine here that is hooked (such as nt!NtEnumerateKey), this can aid you in your analysis regarding a possible rootkit infection.

### !ms_store
Display information related to the Store Manager (ReadyBoost).

The present command allows to list the current ReadyBoost (requires USB 3.0) cache used by the Operating System, but also to display the logs of the memory pages managed by the store manager.
Parameter: /cache

### !ms_timers
Display list of KTIMER.

!ms_timers displays the KTIMER structure, which is an opaque structure that represents and contains various timer objects. This command can be helpful to figure out what drivers created what timer objects, what drivers called what routines, etc.

### !ms_vacbs
Display list of cached VACBs

### !ms_verbose
Turn verbose mode on/off

## Classes
### PEFile
`MsPEImageFile` contains the basic common information used by Windows binaries (PE) and has been derivated into three different classes:

- MsProcessObject
- MsDllObject
- MsDriverObject
