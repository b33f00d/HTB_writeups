# Hack the Box - Luke
###### tags: `HTB`
---


![](https://i.imgur.com/5S0Tirm.png)

## Introduction
Luke seemed to be a pretty simple box but the only difficult thing was learning how to request the token using curl, but besides that you had to enumerate alot and it seemed to be one of the key things in rooting this box.

## Tools Used

* Nmap (scan)
* curl (token request)
* dirb (directory enumeration)
* gobuster (directory enumeration)
* dirsearch (directory enumeration)
* Wget (file downloads)

## Enumeration
Like every box we commence with a scan: ```nmap -sC -sV -oA luke 10.10.10.137```
![](https://i.imgur.com/MiX4k5g.png)

So we have a couple of ports open: ```21(FTP), 22(ssh), 80(http), 1119 (bnetgame), 3000 (HTTP/Node.js Express Framework), 8000 (HTTP/Ajenti), 80 (HTTP/Apache 2.4.38)```

## Enumerating - Port 80

We go to the luke main page and we snoop around on the page, but nothing to find here just a couple of tabs and not much we can do with this.
![](https://i.imgur.com/f5w5I5k.png)

So on this box I started with a gobuster scan, but for some reason I wasn't getting any good output so just to make sure I ran a dirb and dirsearch scan just to be safe.

### Scans

I started off with a dirsearch scan with ```python3 /opt/dirsearch/dirsearch.py -u http://10.10.10.137 -e php,conf,txt ```

![](https://i.imgur.com/jcuLQCc.png)

Right off the bat I immediately go to take a look at the ```config.php``` just from experience config files sometimes will contain credentials, so I use wget to download to my local machine and voila! I end up finding the first set of credentials.

![](https://i.imgur.com/6DHy7X4.png)

I then take a look around again and start finding a couple of login pages ```/management /login.php ```. I tried using those credentials to maybe login but nothing. I then start to enumerate the other ports and go take a look at 3000.

## Enumerating - Port 21

In our nmap scan we notice that we can login anonymously which means that we can login in without any validation and when I logged in I ended up finding a ```for_Chihiro.txt``` file and some interesting message that was for Chihiro. This was going to be useful later on especially if I needed to spray usernames. ;)

![](https://i.imgur.com/mlj4aij.png)


## Enumerating - Port 3000

So I pull up port 3000 and notice that it is requiring a authentication token, but first thing I do again is start to scan the page again just to see if I can find any directories laying around

![](https://i.imgur.com/Yjr80zX.png)

### Scans

I start again with dirsearch ```python3 /opt/dirsearch/dirsearch.py -u http://10.10.10.137:3000 -e php,conf,txt,js,json``` just cause it has been handy so far and find some interesting directories

![](https://i.imgur.com/PgXk8Nb.png)

So when I went to the login page again I needed a token. I started doing a bit of research and found that I needed a JWT Token, but to understand that I felt I needed to know how the authentication worked

![](https://i.imgur.com/GaANH2A.png)

### cURL

Essentially for you to receive a token from the server, you need to provide it a username and password and in return you get your golden ticket! So I started thinking, why not give it the credentials that I already have and use curl to send our request....Soooo after trial and error I figured out how to send the curl request, but I had the username wrong all along and the funny thing is it was right in my face ```/users/admin```!

``` curl --header "Content-Type: application/json"   --request POST   --data '{"password":"Zk6heYCyv6ZE9Xcg", "username":"admin"}'   http://10.10.10.137:3000/login ```

![](https://i.imgur.com/wF8Vn7f.png)

Finally we got our token! Now the fun starts :D I had mentioned before that we had found some usernames in that text file we had found... So I started off by using curl and my toen to see if I could find any other users and passwords. 

Command 1: Populates users
``` curl -X GET -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNTYzMDMzODAwLCJleHAiOjE1NjMxMjAyMDB9.9KRVstCHGToxvLl9laqojo9PI7etbDh3EH4iDzdv7Wc' http://10.10.10.137:3000/users ```

Command 2: Populates specific username and password
```curl -X GET -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFkbWluIiwiaWF0IjoxNTYzMDMzODAwLCJleHAiOjE1NjMxMjAyMDB9.9KRVstCHGToxvLl9laqojo9PI7etbDh3EH4iDzdv7Wc' http://10.10.10.137:3000/users/{username} ```

![](https://i.imgur.com/IiZVL0h.png)


I assumed that if I use the token for the users directory I would get something and my idea was successful! Now we got a couple of creds time to spray all our login pages...🚿

### Credentials spraying

I first tried ```/login.php``` but that didn't work so I tried the management login page and was able to login with Derry's creds (poor Derry).

![](https://i.imgur.com/GvTGzoV.png)

First thing I did was take a look at the config.json file and end up finding creds for the ajenti page. So I go immediately to take a look to see if these creds work....

![](https://i.imgur.com/OPyUGyu.png)

## Enumerating - Ajenti Port 8000

So with the credentials that I found I went to login on port 8000 and used them to login to Ajenti

![](https://i.imgur.com/AcQeehW.png)

And I manage to login....😈
I notice that there is a file manager as well so I go to investigate that to see whats going on there.

![](https://i.imgur.com/ZsdAekZ.png)

In the file manager im able to go to user Derry's home folder and I got user.txt and I then go to root and pick up root.txt. Game Over!!!

![](https://i.imgur.com/tNaAoy5.png)

![](https://i.imgur.com/B6sZafL.png)

Till next time.....




