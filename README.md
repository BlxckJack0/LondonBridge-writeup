 # LondonBridge- This is a tryhackme room medium level, where we exploit an SSRF(Server Side Request Forgery) vulnerbility in a website where it exposes private key of a user, Then getting a foothold and escalate our privileges via kernel exploit using the vulnerbility CVE-2018-18955 subuid_shell.
##
## Background check - SSRF is when a web app takes a URL/path/hostname from user input and the server itself fetches it and you can manipulate that input to make the server send requests somewhere you normally couldn’t reach directly. 
# 1- Reconnaissance


<img width="1843" height="677" alt="Screenshot 2026-11-07 004632" src="https://github.com/user-attachments/assets/829b1108-c223-4bf6-961a-9182926647f1" />


## 

## 2-  Version Detection and basic nse Script

<img width="1781" height="540" alt="Screenshot 2026-11-07 004709" src="https://github.com/user-attachments/assets/d6b17def-b8f3-4908-8a53-e71bd3c1541c" />

##

 ## 3-  Tried using searchsploit to see if there is a vulnerblity that include "gunicorn", But Did not get lucky.


<img width="1760" height="692" alt="image" src="https://github.com/user-attachments/assets/57d3b7bf-8441-4749-80e8-fef44ec5ef27" />


##

## 4- As we Know the machine is running a http-proxy webiste on port 8080, i go and visit and then i used feroxbuster to find hidden Directories

```
feroxbuster -u "target/url:8080"
```
<img width="1915" height="857" alt="image" src="https://github.com/user-attachments/assets/4a1a56d9-2007-4ea9-a22e-0e1ac5de55c1" />
##

## 5 - Then i visited each and every Directory, one of the directories caught my eyes which is /Gallery. it has an upload mechanism. I tried to upload a reverse shell, And then Manipulate the "Content-Type header" by changing it into image/jpeg to bypass the filter but did not get lucky 
<img width="1917" height="1016" alt="image" src="https://github.com/user-attachments/assets/81d3b47c-2898-4f4d-8a43-47b5022a4d42" />


##

## 6 - Then i uploaded a normal jpg image to see and understand how the server respond. I tried every possible bypass filter technique but did not work.

<img width="1917" height="942" alt="image" src="https://github.com/user-attachments/assets/7c109c89-3366-4731-be0a-b3e545e84b72" />


##

## 7- I check the source code of /gallery and i found something Interesting 
<img width="800" height="202" alt="image" src="https://github.com/user-attachments/assets/9b887a6b-44da-42f0-a902-d55e43dd2b21" />


## 8 - I visit the /dejaview Directory and I found Something Interesting which we can Abuse. We can enter an image url and view the image.

<img width="1917" height="998" alt="image" src="https://github.com/user-attachments/assets/01410b2e-6db9-4d43-b27b-daddd9fd1e04" />


## 9 - Then i uploaded an image from /gallery, to see how the server would respond to my request.
<img width="1901" height="966" alt="image" src="https://github.com/user-attachments/assets/414cbf4f-a78a-4b8b-84d3-8ae3d56b3de2" />

##

## 10 - Tried uploading a reverse shell payload but did not work.
<img width="1880" height="995" alt="image" src="https://github.com/user-attachments/assets/b6aa7cb5-4ed8-49cb-8d92-066cd8ae13e5" />



## 11 - Tryhackme gave us a hint by changing the parameter, so we use ffuf to parameter brute Force.
```
ffuf \
  -u http://10.82.135.155:8080/view_image \
  -X POST \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-small-words-lowercase.txt \
  -d "FUZZ=/uploads/tower_bridge.jpg" \
  -fw 226
```
-u - Target URL
#
-X - method


-H - header

-w - Wordlist for the parameter brute forcing 


-d - telling ffuf to use this url and target the Parameter position. (basically setting up the payload position)

-fw - filter 226 words
##

<img width="1825" height="646" alt="image" src="https://github.com/user-attachments/assets/63ad9a54-03a7-426b-ae5d-6c98e670d5d3" />

## 12 - Interesting we found a parameter "www", i started checking the parameter to see if it could expose something 
<img width="1906" height="1010" alt="image" src="https://github.com/user-attachments/assets/2ffa0475-8284-467d-8d31-bed1fcbb612e" />

##


## 13 - Interesting so there is a filter blocking us, i started to look for 127.0.0.1 alteratives and found this 
<img width="945" height="342" alt="image" src="https://github.com/user-attachments/assets/b6338fba-281e-455c-845e-f8ffe25b54f2" />


##

## 14 - Bingo, its now exposing the main webpage lets abuse this and see if it can expose hidden files from other another server.
<img width="1917" height="1006" alt="image" src="https://github.com/user-attachments/assets/9bf2dc50-f570-4658-ba88-17e67f1a9125" />
##


## 15 - After Trying all type of files like /etc/passwd, and other intersting files i forgot the main one which is ".ssh", then i searched for .ssh and we found something 

<img width="1917" height="957" alt="image" src="https://github.com/user-attachments/assets/8416896c-fd0c-49c6-93d3-06a63064ce39" />

##

## 16 - We found the id_rsa sitting in .ssh directory
<img width="1917" height="1012" alt="image" src="https://github.com/user-attachments/assets/467eede8-81e4-407f-a1d6-eebc560be962" />

##

## 17 - but we don't know the username, in step number 15 it exposes 2 files one is id_rsa other is authorized keys. I check authorized keys and boom we found a username "beth" 

<img width="1917" height="991" alt="image" src="https://github.com/user-attachments/assets/d5fe5c12-2085-4f80-be54-7657550b4e5f" />


## 

## 18 - after that i saved the id_rsa, and used chmod 600 because if we used another option the ssh server wont accept the private key because of bad permission 
<img width="1811" height="467" alt="image" src="https://github.com/user-attachments/assets/e7a0cd63-5f8b-4364-982d-b3592354c48a" />


##

## 19- VOULAA, We are In 
<img width="1835" height="567" alt="image" src="https://github.com/user-attachments/assets/1489e2db-51be-448d-9d66-6ef8d115fc68" />


##

## 20 - I ran the command which is written below to find user.txt and we find it sitting in __pycache__.
```
find /home/beth -name "user.txt"
```
<img width="1835" height="567" alt="image" src="https://github.com/user-attachments/assets/2adc3774-72cd-4985-b896-57579f0ca52a" />


##

## 21 - Now privilege escalation time, I started transfering thr tools into the comprimised machine, like linpeas.sh
<img width="1677" height="637" alt="image" src="https://github.com/user-attachments/assets/c869ecdf-cd74-469b-8b24-2f7d93ea107f" />

<img width="1842" height="808" alt="image" src="https://github.com/user-attachments/assets/e046537e-d9a9-4d06-88b9-c8df3da9aff5" />



##
## 22 - Linpeas Found a Kernel Level vulnerblity, Which is sub_uid shell which is CVE-2018-18955.

<img width="1820" height="791" alt="image" src="https://github.com/user-attachments/assets/e7c675df-bab6-46d3-a9f7-43e301d5c29b" />


##

## 23 - I started seachign for the exploit and came across this one.
```
https://github.com/bcoles/kernel-exploits/tree/master/CVE-2018-18955
```

## 24 - i git clone it and transfer it to the comprimised machine 


<img width="1837" height="775" alt="image" src="https://github.com/user-attachments/assets/42a5fb5c-c7ff-4c14-a497-7b64454b82c5" />
<img width="1837" height="775" alt="image" src="https://github.com/user-attachments/assets/40d2513a-cf8e-4110-a6e2-29d28e2b4cb2" />


##
## 25 - After running the exploit which i s./exploit-dbus.sh, we escalated to root 

<img width="1833" height="800" alt="image" src="https://github.com/user-attachments/assets/802430b5-cb1c-4272-847e-d77cea8e4687" />



##


## 26 - After going to the root directory i did not find root.txt, but then i ran "ls -la" which will list and hidden files with there permissions, and we find it.
<img width="1833" height="800" alt="image" src="https://github.com/user-attachments/assets/b5a02888-85b9-4bc7-a655-59023ea0a705" />



##

## 26 - For Finding charles password we Find a firefox profile in his home directory we transfer it to our main machine, and use the tool which is given below 
```
https://github.com/unode/firefox_decrypt/blob/main/firefox_decrypt.py
```

<img width="1683" height="651" alt="Screenshot 2026-11-07 015132" src="https://github.com/user-attachments/assets/9de665af-91bc-42f1-954e-b145546d5e3c" />



## Please make sure to star the repo if u understood the writeup.
















