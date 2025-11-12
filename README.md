Consumer ESU Enrollment
=======================

Windows Powershell script to enroll in Windows 10 Consumer Extended Security Updates (ESU) program via the free Backup option, **with** or **without** Microsoft Account.

***UPDATE 2025-11-12:***
------------------------

- Enrollment with Local account is restored as of 2025-11-12
- Acquire license without enrollment still not working

~~***UPDATE 2025-10-12:***~~
------------------------

~~- Enrollment without Microsoft account is not possible anymore as of 2025-10-08~~  
~~- Acquire license without enrollment no longer works as well~~  
~~- Already enrolled devices with Local account or License should not be affected by this change~~

Requirements
------------

- [Consumer ESU Prerequisites](https://www.microsoft.com/windows/extended-security-updates) ([Old page](https://web.archive.org/web/20250727070928/https://support.microsoft.com/en-us/windows/windows-10-consumer-extended-security-updates-esu-program-33e17de9-36b3-43bb-874d-6c53d2e4bf42)).
- Cumulative Update 2025-06 KB5061087 (19045.6036) or later.
- Enabled Consumer ESU feature (see below).
- Administrative account.
- Internet connectivity.
- User Region is not geo-blocked (Russia, Belarus, Iran, Cuba, North Korea, Syria, Sudan, Lebanon, Venezuela).

______________________________

Design
------

By default, the script will run in the following order, if a step failed, the next is executed:

- Enroll using Microsoft account currently logged-in as Windows user.
- Enroll using Microsoft account currently logged-in with Microsoft Store.
- Enroll using current Local account.

______________________________

Usage
-----

- Click on Code > [Download ZIP](https://github.com/abbodi1406/ConsumerESU/archive/refs/heads/master.zip) button at the top to download.
- Extract all files from the ZIP file.
- Run `Consumer_ESU_Enrollment_run.cmd` as administrator.

Advanced Usage
--------------

***Command Prompt:***  
- Click on Code > [Download ZIP](https://github.com/abbodi1406/ConsumerESU/archive/refs/heads/master.zip) button at the top to download.
- Extract all files from the ZIP file.
- Run *`Command Prompt`* as administrator in the same folder where you located the extracted files, or change location to it using `cd /d` command.
- Execute `Consumer_ESU_Enrollment_run.cmd` with the wanted optional parameters
- Examples:  
`Consumer_ESU_Enrollment_run.cmd -Store -Proceed`  
`Consumer_ESU_Enrollment_run.cmd -Local`  
`Consumer_ESU_Enrollment_run.cmd -Remove`  
`Consumer_ESU_Enrollment_run.cmd -Reset`

***Windows Powershell:***  
- Click on Code > [Download ZIP](https://github.com/abbodi1406/ConsumerESU/archive/refs/heads/master.zip) button at the top to download.
- Extract all files from the ZIP file.
- Run *`Windows Powershell`* as administrator in the same folder where you located the extracted files, or change location to it using `cd` command.
- Temporary allow running unsigned scripts:  
`Set-ExecutionPolicy Bypass -Scope Process -Force`
- Execute `Consumer_ESU_Enrollment.ps1` (with optional parameters if wanted)
- Examples:  
`.\Consumer_ESU_Enrollment.ps1`  
`.\Consumer_ESU_Enrollment.ps1 -Store -Proceed`  
`.\Consumer_ESU_Enrollment.ps1 -Local`  
`.\Consumer_ESU_Enrollment.ps1 -Remove`  
`.\Consumer_ESU_Enrollment.ps1 -Reset`

Optional Parameters
-------------------

|Switch    |Effect|
|----------|------|
| -Online  | Only enroll using Microsoft user account token, exit if failed |
| -Store   | Only enroll using Microsoft store account token, exit if failed |
| -Local   | Only enroll using Local user account token, exit if failed |
| -Remove  | Remove Consumer ESU License if exists |
| -Reset   | Reset Consumer ESU features to the default state (if changed by the script) |
| =        | =
| -Proceed | Force running enrollment, even if Eligibility status is already enrolled |

- You must only specify **one** switch of the first three switches.
- Only `-Proceed` switch can be combined with the three enroll switches to re-enroll with a different token.

______________________________

Important Note
--------------

- If you only have one user account, once you successfully got `DeviceEnrolled` status,  
and to avoid changing or reverting the state, specially for EEA or geo-blocked regions,  
it is recommended to disable all related Consumer ESU scheduled tasks.
- To do so, run *`Consumer_ESU_ScheduledTasks.cmd`* as administrator, and press 1 to disable them.
- You can reenable them anytime using 2nd option of the same script.

______________________________

Consumer ESU Feature
--------------------

- If the feature is not broadly enabled yet, the script will try to enable it.

- If the script asked to close the session, then close the whole console window, and run the script again (with same parameters if any).

<details><summary>Manual Reference</summary>


How to enable it manually yourself, this require a reboot to take effect:

- Run *`Command Prompt`* as administrator.
- Execute the following command:  
```
reg.exe add "HKLM\SYSTEM\CurrentControlSet\Policies\Microsoft\FeatureManagement\Overrides" /v 4011992206 /t REG_DWORD /d 2 /f
```
- Run *`Windows Powershell`* as administrator.
- Copy and paste the following commands together as-is, wait for "Task Completed" message:  
```
$TN = "ReconcileFeatures"; $TP = "\Microsoft\Windows\Flighting\FeatureConfig\"; $null = Enable-ScheduledTask $TN $TP
Start-ScheduledTask $TN $TP; while ((Get-ScheduledTask $TN $TP).State.value__ -eq 4) {start-sleep -sec 1}; "Task Completed"
#
$TN = "UsageDataFlushing"; $TP = "\Microsoft\Windows\Flighting\FeatureConfig\"; $null = Enable-ScheduledTask $TN $TP
Start-ScheduledTask $TN $TP; while ((Get-ScheduledTask $TN $TP).State.value__ -eq 4) {start-sleep -sec 1}; "Task Completed"
#
```
- **Restart the system**.
- .
- Run *`Command Prompt`* as administrator.
- Execute the following commands:  
```
cmd /c ClipESUConsumer.exe -evaluateEligibility
reg.exe query "HKCU\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Windows\ConsumerESU"
```
- Verify that the last command shows **ESUEligibility** value as non-zero.  
if so, proceed to run the powershell script as explained above.
- If the value is zero `0x0` or does not exist, then the operation is failed, and you have to wait for official broad availability.
</details>

______________________________

Bypass Region Block
-------------------

<details><summary>Click to expand</summary>


- Temporary change your region to non-blocked country:

Table of Geographical Locations:  
https://learn.microsoft.com/en-us/windows/win32/intl/table-of-geographical-locations

manually:  
`Settings > Time & Language > Region > Country or region`

or run *`Windows Powershell`* and execute:  
`Set-WinHomeLocation -GeoId 244`

- Run the script to enroll as explained above.

- Verify that "ESU Eligibility state" is `DeviceEnrolled / SUCCESS`.

- Run *`Consumer_ESU_ScheduledTasks.cmd`* as administrator, and press 1 to execute this option:

`[1] Disable Consumer ESU scheduled tasks`

- Restore your original region location, manually or using powershell as before.
</details>
