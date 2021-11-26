**CVEID**: pending

**Bluestacks Advisory ID**: BS-2021-002

**Name of the affected product(s) and version(s)**: BlueStacks for Windows (all versions prior to 5.1.100.1020)

**Vulnerability type**:  CWE-552: Files or Directories Accessible to External Parties

---

**Summary**

BlueStacks is an Android emulator which runs the guest Android system within a virtual machine. Because BlueStacks
configures the current Windows user's 'My Documents' folder as a shared directory, it is possible for a malicious
Android application to execute code within the host system by creating or modifying a Powershell profile.

**Description**
 
BlueStacks for Windows (all versions before 5.1.100.1020) configures the current user's 'My Documents' folder as
a shared directory between host (Windows) and guest (Android). On the Android system, the directory shows up as
```/mnt/windows/Documents``` with the following DAC and MAC permissions:
```
drwxrwxr-x  1 system sdcard_rw u:object_r:unlabeled:s0    24576 2021-06-07 10:53 Documents
```
As the ```sdcard_rw``` group consists of all applications with the common ```WRITE_EXTERNAL_STORAGE``` permission,
this means that every application which is granted it will be able to read and write to the current Windows user's
'My Documents'.

A malicious application could exploit this by writing arbitrary PowerShell code to
```/mnt/windows/Documents/WindowsPowershell/profile.ps1```. This will overwrite the current user's PowerShell
profile, which is a script that gets executed every time said user runs PowerShell - in effect, achieving code
execution on host with the next PowerShell invocation.

Exploitation of this issue is mitigated by the fact that it requires PowerShell's execution policy to allow running
scripts. By default, Windows 10 uses Restricted policy which disallows it, while Windows Server systems use the
RemoteSigned policy which makes this attack possible.
 
**Reproduction**

- set up the victim environment:
  - run a PowerShell instance with Administrator privileges
  - ```Set-ExecutionPolicy RemoteSigned```
  - Input ```Y``` or ```A``` when prompted to confirm
- set up the attacker environment:
  - start a BlueStacks instance, configured to allow ADB connections (ADB is used for demonstration purposes here;
    this attack can also be executed by an installed application)
  - connect ADB to BlueStacks
- attacker: ```adb shell "mkdir -p /mnt/windows/Documents/WindowsPowershell && echo calc.exe > /mnt/windows/Documents/WindowsPowershell/profile.ps1"```
- victim: run PowerShell
- calculator should open along with PowerShell

**Remedy**

Install a newer version of BlueStacks.
