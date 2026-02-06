# TcbElevation

**TcbElevation** is a Proof-of-Concept (PoC) tool demonstrating **Local Privilege Escalation (LPE)** on Windows systems where the current user possesses **SeTcbPrivilege** (`Act as part of the operating system`).  
The tool abuses **SSPI hooking** to impersonate the **NT AUTHORITY\SYSTEM** LUID and gain a fully privileged handle to the **Service Control Manager (SCM)**, allowing execution of arbitrary commands as **SYSTEM** via a temporary service.

> ⚠️ **Educational PoC** — Intended strictly for authorized security research and lab environments.

---

## 🚀 Key Features

- **SeTcbPrivilege Enablement**  
  Automatically enables `SeTcbPrivilege` in the process token.

- **SSPI Hooking**  
  Hooks authentication routines to spoof the caller LUID.

- **SYSTEM Impersonation**  
  Uses the SYSTEM LUID (`0x3E7`) during `OpenSCManager` calls.

- **SCM Access**  
  Obtains full permissions on the Service Control Manager.

- **Command Execution**  
  Executes arbitrary commands as `NT AUTHORITY\SYSTEM`.

---

## 📷 Proof of Concept

A user with `SeTcbPrivilege` is normally denied when attempting administrative actions such as adding local users.  
`TcbElevation.exe` bypasses this restriction and successfully performs the action as **SYSTEM**.

<img width="1071" height="809" alt="seTcb_poc" src="https://github.com/user-attachments/assets/e3e1ce46-0748-48de-a593-966587d8e234" />

---

## 🛠 Compilation

Build using **Visual Studio** or **MSVC (cl.exe)**.

### Prerequisites
- Windows SDK  
- Required headers: `windows.h`, `sspi.h`
- Libraries (pragma-linked):  
  - `Secur32.lib`  
  - `Advapi32.lib`

