# This is the Writeup of Perfect Synchronization challenge on Hack The Box Cyber Apocaplypse 2023
So this challenge begins with 2 given files: [output.txt]() and [source.py](). In the output.txt we can see around 1500 lines of gibberish and in the source.py 
we are given the source code that this encryption has used. Let's focus on the source.py file:
'''
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
    
'''
There were 2 things that were interesting for me: **key = urandom(16)** and **self.cipher = AES.new(key, AES.MODE_ECB)**. After searching for a while, I found that
function *urandom(16)* basically returns the 16 bytes long key which is the equivalent of 128 bits which is the equivalent of AES-128 encryption.
AES-128 encryption is a very strong one and would take about **one billion years** to break, so the bruteforcing this message was not even an option
