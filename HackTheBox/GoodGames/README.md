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
Service Info: Host: goodgames.htb```


**Don't forget to add goodgames.htb to your /etc/hosts !!!**

Well, the scan didn't show much, so we begin to poke around the main site. What is interesting (for hackers, obviously...) is the login form. We need to provide email and password to be logged into our account. At first, I spend some time poking around with JWT tokens - but it was not the correct way of thinking. I gave up on trying to bruteforce/guess admin's password, as for logging we were using email address, which I didn't know (at this point). After looking up other writeups, it turned out that I should have tried the infamous **SQL INJECTION**. Apparently, even though we had to provide email address in loging form, if we used typical SQL injection payload `email=admin' or 1=1 -- -` we still got logged in as admin (sick!). Now that's a surprise! If we were able to exploit the `email` field with our famous payload, we could test it using tool called `sqlmap`. All we had to do was to intercept our burp request during loging, save it as `request.txt` and then use command like `sqlmap -r request.txt`. This results in a dumped database with usernames and password hashes. Bingo! Now we just need to crack the admin's password and we are ready to go!

