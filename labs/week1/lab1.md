**1. Create an SSH Key Pair **

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

**2: Copy the SSH Public Key to the Server - For Windows Users ONLY **

----

**REQUIRED for Windows account names containing whitespace:**

Revise the last string of your public key **before** copying your public key to the remote server.

Example _"User Name"_ ~ The last line of the public key will state:
```
User Name@[laptop-local-address]
```
Revise this adding quotation marks:
```
"User Name@[laptop-local-address]"
```
----

Run the following (in one line), replacing [username] with your HPCC username, which we're providing. This sends a copy of your public key to the remote server:

```
cat ~/.ssh/id_rsa.pub | ssh [username]@hpcc-cluster.stanford.edu "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

After you copy the SSH public key to the server, you will see a warning like this:
```
The authenticity of host 'hpcc-cluster.stanford.edu (171.64.55.130)' can't be established.
ECDSA key fingerprint is SHA256:20rcvjngfjkrjjank45436tkjfhsdkfsdHjosfjJhk.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
```

Type _yes_ and press the enter key.

You'll receive a response like this:
```
Warning: Permanently added 'hpcc-cluster.stanford.edu,171.64.55.130' (ECDSA) to the list of known hosts.
```

You'll be prompted for your HPCC password, which we provide.
Type your password and press enter
**NOTE** Your keystrokes will not appear on the screen.
```
user@hpcc-cluster.stanford.edu's password:
```

You likely won't receive a response after entering the password.

**2: Copy the SSH Public Key to the Server - For MacOS & Linux Users ONLY**

Run the following, replacing [username] with your HPCC username, which we're providing:

_ssh-copy-id [username]@hpcc-cluster.stanford.edu_
```
~$ ssh-copy-id david@hpcc-cluster.stanford.edu
```

You will get a warning like this:
```
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/kali/.ssh/id_rsa.pub"
The authenticity of host 'hpcc-cluster.stanford.edu (171.64.55.130)' can't be established.
ECDSA key fingerprint is SHE2566:20crZ5XWQNxMgW0BZo3mE97j7z2ThNoULDzVeUAVeCg.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```
