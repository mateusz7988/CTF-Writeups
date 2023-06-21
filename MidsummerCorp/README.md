## This is the Writeup for The Midsummer Corp Hack CTF on TryHackMe platform.
  
Hi! My name is Mateusz and I wanted to share my writeup of The Midsummer Corp Hack CTF with you (or at least challenges that I managed to solve).
  
  
TryHackMe nick: Warx
  
List of contents:
1. Fern Flower
2. Midsummer Corp
3. Puck
4. Leshy
5. Baba Yaga
6. Boruta
7. Twardowski
8. Popiel
9. End of the journey
  
# 1. Fern Flower
The first challenge doesn't really require any knowledge, so let's just mark it down as completed and move to the next one!
  
# 2. Midsummer Corp
Now in this challenge, the hint says to inspect the Javascript carefully. So let's do this! I started inspecting all the available JS and finally
stumbled upon this line in /apps/user_saml/saml/selectUserBackEnd:  
  
`var _theme={"entity":"Midsummer Corp","name":"Midsummer Corp","productName":"Nextcloud","title":"Midsummer Corp","baseUrl":"https:\/\/files.midsummer.corp.local","syncClientUrl":"https:\/\/nextcloud.com\/install\/#install-clients","docBaseUrl":"https:\/\/docs.nextcloud.com","docPlaceholderUrl":"https:\/\/docs.nextcloud.com\/server\/26\/go.php?to=PLACEHOLDER","slogan":"a safe home for all your flowers","logoClaim":"","folder":""}`

There, we can see key-value pair as `{baseUrl":"https:\/\/files.midsummer.corp.local}`

Bingo! That is our answer!

# 3. Puck
In this challenge, there are some theoretical questions and some practical. Let's now answer all the questions that we can, and the we will think about the rest.
```
What do we call the process of verifying the identity of a user or system? Authentication
How do we call the process of verifying that a user has access to a particular resource? Authorization
```
Aaand that would be the end of the questions that we know the answer for (for now). The next questions is: How long is Puck's password? That probably means
that his password is stored somewhere and we don't need to bruteforce it. I really like to just open the JS and look for key-words like: "default", "password",
"admin", "pass", etc. So that's exactly what we will do! Let's look for "default". After couple of tries, in the same file as before (/apps/user_saml/saml/selectUserBackEnd) we are able to find
key-value pair `{"defaultPassword":"sfLfSNYavTD4PL4Z"}`. Well that's unfortunate! But only for the company that we are trying to hack :)
Now we can try to log in using this password and guess what? It is successful!  
Let's log in and answer the questions:
```
How long is Puck's password? 16
What is the content of the file Fern_flower_ritual_shard1.txt in Puck's account? Midsummer_Corp{W@it_unt!1_m1dn1ght_0n_th3_Summ3r_Solst1c3}
```
To answer the final question, we need to see that at the bottom of our files list is written: `2 folders and 25 files  (including 1 hidden)`. Well we need to find the hidden folder!
The first thing to try is to simply visit the Settings and we are greeted with big square with the name `show hidden files`. Now, let's just click on it
and we will have access to hidden `.mail` folder with `inbox.mbox` file inside. After downloading it and reading through emails, we can see that there 
is the answer for the last question:
```
Who is going on vacation? Please provide their email address. leshy@midsummer.corp.local
```
Don't forget to download the `fernflower_flag1.png` file that we will need later! That's all for challenge #3 :)

# 4. Puck



