# This is the Writeup of Perfect Synchronization challenge on Hack The Box Cyber Apocaplypse 2023
So this challenge begins with 2 given files: [output.txt](https://github.com/mateusz7988/CTF-Writeups/blob/main/CyberApocalypse2023/PerfectSynchronization/output.txt) and [source.py](https://github.com/mateusz7988/CTF-Writeups/blob/main/CyberApocalypse2023/PerfectSynchronization/source.py). In the output.txt we can see around 1500 lines of gibberish and in the source.py 
we are given the source code that this encryption has used. Let's focus on the source.py file:


```python

from os import urandom
from Crypto.Cipher import AES
from secret import MESSAGE

assert all([x.isupper() or x in '{_} ' for x in MESSAGE])


class Cipher:

    def __init__(self):
        self.salt = urandom(15)
        key = urandom(16)
        self.cipher = AES.new(key, AES.MODE_ECB)

    def encrypt(self, message):
        return [self.cipher.encrypt(c.encode() + self.salt) for c in message]


def main():
    cipher = Cipher()
    encrypted = cipher.encrypt(MESSAGE)
    encrypted = "\n".join([c.hex() for c in encrypted])

    with open("output.txt", 'w+') as f:
        f.write(encrypted)


if __name__ == "__main__":
    main()

```


There were 2 things that were interesting for me: **key = urandom(16)** and **self.cipher = AES.new(key, AES.MODE_ECB)**. After searching for a while, I found that
function **urandom(16)** basically returns the 16 bytes long key which is the equivalent of 128 bits which is the equivalent of AES-128 encryption.
AES-128 encryption is a very strong one and would take about **one billion years** to break, so the bruteforcing of this message was not even an option.
So, I focused on **AES.MODE_ECB**. After a short research I was able to find this information:



    In terms of security, ECB is generally a bad choice since identical plain text blocks are encrypted to identical cipher text blocks, 
    which allows a possible attacker to disclose patterns in our ciphered messages


Bingo! This basically means that if we have for example letter "a" it will always be encoded as for example "dfc8a2232dc2487a5455bda9fa2d45a1", the same goes for "b" being for example "c87a7eb9283e59571ad0cb0c89a74379" and so on. If you take a look at the file **output.txt** you will see that there are indeed repeating patterns of characters. Try this combination in SublimeText: **ctrl + alt + H**. It will find all the occurences of the value and change them for the value of your choice. So what we need to do, is to find all occurrences of "dfc8a2232dc2487a5455bda9fa2d45a1" and count them, then find all occurrences of "305d4649e3cb097fb094f8f45abbf0dc" and count them and so on and so on... Of course we will use a simple script that will do this for us (the **output.txt** is 1500 lines long!!). For our purpose I wrote this script in python:



``` python

data = open("output.txt", 'r').readlines()
lista = []
db = {}
for item in data:
    lista.append(item)

while True:
    if len(lista) == 0:
        break
    else:
        i = 0
        number = data.count(lista[i])
        db[lista[i].strip("\n")] = number

        lista.remove(lista[i])

db = sorted(db.items(), key=lambda x:x[1], reverse=True) #sort dictionary in the descending order

print(db)

```

    
**_I KNOW THIS SCRIPT COULD BE WRITTEN BETTER BUT IT WAS JUST A QUICK SOLLUTION IN WHICH I DIDN'T INVEST TOO MUCH TIME - SORRY NOT SORRY PYTHON FREAKS LOL_**

**_ALSO I KNOW THIS SCRIPT THINKS THAT THE "c87a7eb9283e59571ad0cb0c89a74379" OCCURRS ONLY ONCE, WHICH IS OBVIOUSLY AN ERROR, IDK WHY THIS HAPPENED BUT I WAS ABLE TO SUBSTITUTE THE RIGHT LETTERS ANYWAY_**

Now, when the script counted all of the occurrences of encoded values, we can try the letter substitution method later. I found the list of the most common letters in english alphabet which i will substitute for the most common values in **output.txt** later on. I also knew that flag that I need to find is in format **_HTB{flag_that_i_need_to_find}_**, so I knew that in my **output.txt** there should be at least one occurence of "{" character and "}" character. I also knew that 3 letters before the "{" character will be the **"HTB"** letters. This is something to work with! My script found that there was only 1 occurrence of the "fbe86a428051747607a35b44b1a3e9e9" value on line 1099 (which I supposed will be the "{") and only 1 occurrence of the "c53ba24fbbe9e3dbdd6062b3aab7ed1a" on line 1129 of **output.txt** (which I supposed will be the "}"). Hmmm... This looks like a flag! Now I started to substitute the most common occurrences of the values from **output.txt** file with the most common english letters. Inside of the "{...}" I decided to write some "_" characters (which are very typical for the flag format) in the places that looked suspicious. After a while in the lines 1050 - 1250 of **output.txt** I got this:


    TEULPSONSFEISMEROIONRONUICEISICHRNREFASATICCHE**DOW{I_RNKBCA_RPWRONOPONLS_NR_VAIX}**EUITMEOHBAEKIUDNSATHEVATAEGNTROEPRAMENSEVLT
    CMEVITENNEBLRRNWCHEWHEODAEPREITKHRERNREOLMIHEODAEDITMEVLTXELGECAOOATEULPSONSFE
    

Well now it looks unappealing, but after pasting this output to the quipqiup online tool (which was given in the description of the challenge) and giving it hint that **DOW=HTB** (we know this because the 3 letters before the "{" character are "HTB"), quipqiup was able to produce output like this using the solve(dictionary) method:


    ???OUNTIN??AN??STATISTI?AL?ANAL?SIS??ENE?ALL??**HTB{A_SIMPLE_SUBSTITUTION_IS_WEAK}**??A???T?PE?MA?HINE???WE?E??I?ST?USE??IN?WO?L?
    ?WA??II?POSSIBL??B??THE?US?A?M?S?SIS?TO?A??THE?HA???WO?K?O??LETTE???OU
    


Here we can easily see that our flag is: **_HTB{SIMPLE_SUBSTITUTION_IS_WEAK}_**
