TcbElevation: SeTcbPrivilege Escalation PoC

TcbElevation is a Proof-of-Concept (PoC) tool designed to demonstrate Local Privilege Escalation (LPE) on Windows by abusing the SeTcbPrivilege ("Act as part of the operating system").

If a user account (such as a service account like IIS AppPool or SQL Service) possesses SeTcbPrivilege, this tool leverages SSPI hooking to impersonate the NT AUTHORITY\SYSTEM LUID. This allows the attacker to gain a handle to the Service Control Manager (SCM) with full permissions, enabling the creation of a malicious service that executes arbitrary commands as SYSTEM.

🚀 Features

Privilege Enablement: Automatically enables SeTcbPrivilege in the current process token.

SSPI Hooking: Intercepts AcquireCredentialsHandleW to spoof the authentication LUID.

SCM Manipulation: Opens the Service Control Manager with SC_MANAGER_CREATE_SERVICE rights (normally restricted to Administrators).

Arbitrary Command Execution: Creates and starts a temporary service to execute any command as SYSTEM.

📷 Proof of Concept

As shown below, a user with SeTcbPrivilege cannot normally add users (Access is denied). Using TcbElevation.exe, we bypass this restriction to create a local administrator.

<img width="1071" height="809" alt="seTcb_poc" src="https://github.com/user-attachments/assets/a70e1bda-446b-4626-b259-6fa5fb074d23" />


🛠 Compilation

The project is written in C++ and can be compiled using Visual Studio or the MSVC compiler (cl.exe).

Prerequisites

Windows SDK (for sspi.h, windows.h)

Libraries: Secur32.lib, Advapi32.lib (Already pragma-linked in source)

Build Command

cl.exe /EHsc TcbElevation.cpp


💻 Usage

TcbElevation.exe <ServiceName> <Command>


<ServiceName>: A unique name for the temporary service (e.g., UpdateSvc).

<Command>: The command line to execute (must be a valid service binary or a cmd /c wrapper).

Example: Create a Local Admin

# 1. Add a new user
.\TcbElevation.exe svc_add "cmd /c net user hacker pass123!@# /add"

# 2. Add user to Administrators group
.\TcbElevation.exe svc_group "cmd /c net localgroup administrators hacker /add"


⚠️ Known Behaviors & Errors

Error 1053 (The service did not respond...):

This is normal. When using cmd /c as a service binary, cmd.exe does not communicate back to the Service Control Manager. The SCM will kill the process after a timeout, but your payload will have already executed.

Error 1073 (The specified service already exists):

You are trying to reuse a Service Name that was created in a previous run but not deleted.

Fix: Use sc delete <ServiceName> or simply choose a different name for the service argument.

🧠 Technical Details

Privilege Check: The tool checks for SeTcbPrivilege. This privilege allows a process to assume the identity of any user.

SSPI Hook: We hook the AcquireCredentialsHandleW function in the Security Support Provider Interface.

LUID Spoofing: When OpenSCManager is called, it triggers an authentication check. Our hook intercepts this and replaces the caller's LUID with 0x3E7 (The SYSTEM LUID).

Service Creation: The SCM believes the request is coming from SYSTEM and grants a handle. The tool then registers a new service running cmd /c <payload> as LocalSystem.

📜 Disclaimer

This tool is for educational purposes and authorized security research only. Misuse of this software to attack systems you do not own or have explicit permission to test is illegal. The author is not responsible for any damage caused by the use of this tool.

👏 Credits

Technique originally documented by James Forshaw (@tiraniddo).

Based on research into Windows Access Tokens and the LSA.
