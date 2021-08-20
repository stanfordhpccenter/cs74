1. Create an SSH Key Pair

On your local computer, start MacOS Terminal, Windows PowerShell, or your Linux terminal of choice.

Run ssh-keygen.

```
PS c:\Users\User Name> ssh-keygen
```

This output should follow:

```
Generating public/private rsa key pair
Enter file in which to save the key (C:\Users\User Name/.ssh/id_rsa):
```

Press the enter key to choose the default location ~/.ssh

**NOTE:** For the purpose of this course we will not add passphrases to the key pair.

Press the enter key twice to generate the key without a passphrase.

This output should follow:

```
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in C:\Users\User Name/.ssh/id_rsa
Your public key has been saved in C:\Users\User Name/.ssh/id_rsa.pub.
The key fingerprint is:
The key's randomart image is:
+---[RSA 2048]----+
|=OO**o.          |
|.oO=.o .         |
|o+oo. .          |
| + o.  .         |
|. + .   S . .    |
|.. o o   + o     |
|..+.o . .   E    |
|+=oo . .         |
|@+o.o            |
+----[SHA256]-----+
```
