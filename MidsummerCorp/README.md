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
The first thing to try is to simply visit the File Settings and we will be greeted with the a big square with the name `show hidden files`. Now, let's just click on it and we will have access to hidden `.mail` folder with `inbox.mbox` file inside. After downloading it and reading through emails, we can see that there 
is the answer for the last question:
```
Who is going on vacation? Please provide their email address. leshy@midsummer.corp.local
```
Don't forget to download the `fernflower_flag1.png` file that we will need later! That's all for challenge #3 :)

# 4. Leshy
This part mainly focuses on MFA (multi-factor authentication). We will need to abuse it. In the previous challenge, we got access to leshy's password but when we try to log in to his account, we will need to provide the MFA code. The first thing to do is to play a bit with the MFA options on the account that we already have access to (Puck). When I tried to switch the MFA on, it showed me this: `Your new TOTP secret is: 234567 `. Well, this number looks suspicious, just as it was not randomly generated but the same for all of the company workers! I downloaded the `Authy` app on my phone and paired it with Midsummer Corp app. Now, when I tried to log in as Leshy, I provided the MFA code that my app generated (that was meant for Puck's authentication) and it worked! We are now logged in as Leshy!  
The answers for the questions are listed below:
```
What is the length of the MFA code used in the application? Enter a numeric value in your answer.  6
What is the content of the file Fern_flower_ritual_shard2.txt in Leshy's account? Midsummer_Corp{Fo11ow_Th3_Wi1l_o'_7h3_W1sps}
```
Remember to grab the part of the flag stored in the `fernflower_flag2.png` file!

# 5. Baba Yaga
This challenge relies on "Rate limiting bypass". In the description of the challenge we are given some examples of how we can exploit this vulnerability. We can use Burpsuite to intercept the requests and add one of the following headers:
```
X-Originating-IP: 127.0.0.1 

X-Forwarded-For: 127.0.0.1 

X-Remote-IP: 127.0.0.1 

X-Remote-Addr: 127.0.0.1 

X-Client-IP: 127.0.0.1 

X-Host: 127.0.0.1 

X-Forwared-Host: 127.0.0.1
```
If we don't add one of these headers, our request will not be accepted with "code 429 Too Many Requests". But now, after we used the `X-Forwarded-For: 127.0.0.1` header we get our beautiful "code 200 OK" :)  
![image](https://github.com/mateusz7988/CTF-Writeups/assets/108484575/ffd968ea-1278-4a3d-bc78-7a3b1e33421d)

Here you can see the response with parameters:
```
{
"status":"success",
"success_path":"\/lostpassword\/reset\/form\/\u003CTOKEN\u003E\/\u003CUSER\u003E"
}
```
If we URL-decode the success_path value, we get the answer for another question:

```
Which HTTP header was used to bypass throttling?  X-Forwarded-For
What is the endpoint path for resetting a password? /lostpassword/reset/form/<TOKEN>/<USER>
```
Normally, we would need to know the \<TOKEN\>, BUT! In the challenge description we have this piece of information: `For example, what if the server does not check the validity of the token at all?` Hmmmm... They do not check validity at all? Let's try this!  
To reset password, I replaced the \<TOKEN\> value with `please_give_me_lego` and tried to access this endpoint through browser. Remember to intercept this request and add the `X-Forwarded-For: 127.0.0.1`! Without this your request will not be accepted!  
After reaching this endpoint, we can reset the password and log in as `babayaga` user. Indeed, our \<TOKEN\> value was not checked. Inside the files folder, we can find our flag.
```
What is the content of the Fern_flower_ritual_shard3.txt file in babayaga account? Midsummer_Corp{F1nd_th3_cl34r1ng_w1th_th3_anc13nt_st0n3s}
```
Remember to grab the `fernflower_flag3.png` file :)

# 6. Boruta

This part gave me the most trouble from them all (or at least from the ones that I solved). Challenge 6 relies on `Mass assingment` vulnerability. This should be fun right? Let's try to add a device in our Midsummer Corp app through Settings in the Security section. If we intercept this request with Burpsuite, we get response like this:
![image](https://github.com/mateusz7988/CTF-Writeups/assets/108484575/03e08f64-25fd-47dc-ad38-607e0d020419)
```
{
"token":"XP8De-mNw6d-sQ4XR-44PoQ-oHE7B",
"loginName":"babayaga",
"deviceToken":{
  "id":27,
  "name":"test",
  "lastActivity":1687365291,
  "type":1,
  "scope":{
  "filesystem":true
},
"canDelete":true,
"canRename":true}
}
```
The thing that looks interesting is the `loginName:babayaga` key-value pair. This is the answer for the first question:
```
 What parameter name assigns the app password to the specific user? loginName
```
But, when I tried to send request with additional parameters like `loginName:boruta`, I got an error like this:
![image](https://github.com/mateusz7988/CTF-Writeups/assets/108484575/d4490c97-694f-4743-95db-c4b96921e3aa)

This is the moment when I decided to inspect the source code of application. To find interesting parts of big source code, I used the error message and `grep` command like this: `grep -r "Blocked by web application firewall." www`. The part that is interesting for me is stored in 
`www/apps/settings/lib/Controller/AuthSettingsController.php`. The code that is responsible for vulnerability looks like this:
```php
public function create($name, $loginName) {
		$ALLOWED_USERS = ['boruta'];

		if (in_array($loginName, $ALLOWED_USERS) && !is_null($loginName)){
			return new JSONResponse([
				'error' => "Blocked by web application firewall.",
			]);
		}

		if (!in_array(rtrim($loginName), $ALLOWED_USERS) && !is_null($loginName)){
			return new JSONResponse([
				'error' => "Account ".$loginName." does not support device tokens.",
			]);
		}

		if ($this->checkAppToken()) {
			return $this->getServiceNotAvailableResponse();
		}

		try {
			$sessionId = $this->session->getId();
		} catch (SessionNotAvailableException $ex) {
			return $this->getServiceNotAvailableResponse();
		}
		if ($this->userSession->getImpersonatingUserID() !== null) {
			return $this->getServiceNotAvailableResponse();
		}

		try {
			$sessionToken = $this->tokenProvider->getToken($sessionId);
			$loginName2 = $sessionToken->getLoginName();
			try {
				$password = $this->tokenProvider->getPassword($sessionToken, $sessionId);
			} catch (PasswordlessTokenException $ex) {
				$password = null;
			}
		} catch (InvalidTokenException $ex) {
			return $this->getServiceNotAvailableResponse();
		}

		if (mb_strlen($name) > 128) {
			$name = mb_substr($name, 0, 120) . '…';
		}

		if(is_null($loginName)) {
			$loginName = $loginName2;
		}
		else {
			$loginName = rtrim($loginName);
		}

		$token = $this->generateRandomDeviceToken();
		$deviceToken = $this->tokenProvider->generateToken($token, $loginName, $loginName, $password, $name, IToken::PERMANENT_TOKEN);
		$tokenData = $deviceToken->jsonSerialize();
		$tokenData['canDelete'] = true;
		$tokenData['canRename'] = true;

		$this->publishActivity(Provider::APP_TOKEN_CREATED, $deviceToken->getId(), ['name' => $deviceToken->getName()]);

		return new JSONResponse([
			'token' => $token,
			'loginName' => $loginName,
			'deviceToken' => $tokenData,
		]);
	}
```

I know, I know, it looks complicated - but don't worry! I will explain everything :)  
Let's focus on this part:
```php
if (in_array($loginName, $ALLOWED_USERS) && !is_null($loginName)){
			return new JSONResponse([
				'error' => "Blocked by web application firewall.",
			]);
		}

		if (!in_array(rtrim($loginName), $ALLOWED_USERS) && !is_null($loginName)){
			return new JSONResponse([
				'error' => "Account ".$loginName." does not support device tokens.",
			]);
		}
```

This code checks if value passed in `loginName` is equal to `boruta` using the `in_array($loginName, $ALLOWED_USERS)` function. But, the next check is made by using the `!in_array(rtrim($loginName))`. The `rtrim` function basically just removes trailing spaces from a string. This means, that if we pass `loginName` with value like: `"boruta    "`, it will return false by the `in_array` function, as `"boruta" != "boruta     "` and then, the next check will also not be fulfilled as `!in_array(rtrim("boruta    "), "boruta")` is false  
(because `!in_array(rtrim("boruta    "), "boruta") ---> !in_array("boruta", "boruta") ---> False`). Then, the value of `loginName` will be assigned normally:
```php
return new JSONResponse([
			'token' => $token,
			'loginName' => $loginName,
			'deviceToken' => $tokenData,
		]);
```
And then, we will be able to log in as `boruta` using device:
![image](https://github.com/mateusz7988/CTF-Writeups/assets/108484575/11ad4b38-c159-4951-997a-809e7bcd8b67)

Now, when we download the `Nextcloud` app, we will be able to log in as device (and not as the browser) using the `token` value.
After connecting with `Nextcloud` app, we get access to all files from the Boruta's account. If we open the `Fern_flower_ritual_shard4.txt` file, we get the answer to the last question:
```
What is the content of the Fern_flower_ritual_shard4.txt file in Boruta's account? Midsummer_Corp{L3ave_an_0ff3r1ng_f0r_th3_spir1ts}
```
Again, don't forget to grab the `fernflower_flag5.png` file as we will need it later!

# 7. Twardowski

Unfortunately, I didn't manage to solve this part. I tried to build my own SAML response including attributes like role (sso), alias (twardowski), etc. To do this, I used online tool https://www.samltool.com. I have spent really long time on this and I think I was really close but I had to finish this writeup before the end of the CTF and I simply run out of time. I will paste the answers to all the questions that I did answer below:
```
What is the name of the XML entity that contains information about user such as their name, email, roles, etc.? Assertion
What is the name of a party that authenticates users and issues SAML assertions? Identity Provider
What should be the value of Content-Type header when sending the SAML Response? application/x-www-form-urlencoded
What should be Destination set to in SAML Response? http://files.midsummer.corp.local/apps/user_saml/saml/acs
What should be Issuer set to in SAML Response? http://idp.midsummer.corp
```
I think, that the problem with my SAML request is that it wasn't signed. I tried to sign it using the tool mentioned before but I couldn't figure it out. I will for sure focus on understanding SAML further and will look forward to other CTFs from the Securing company as this one gave me a lot of fun and a made me learn lot of new skills! 

# 9. End of the journey
Here, I will paste the partial flag that I managed to find across all of the accounts:
```
Midsummer_Corp{Thw33_rf_3rr3 <EMPTY> f@!l <EMPTY> }
```
