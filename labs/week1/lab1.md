**1. Create an SSH Key Pair**

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

**2: Copy the SSH Public Key to the Server - For Windows Users ONLY**

**REQUIRED only for Windows account names containing whitespace:**

Revise the last string of your public key **before** copying your public key to the remote server.

For example, if your user name is _"User Name"_, the last line of the public key will state:
```
User Name@[laptop-local-address]
```
Revise this by adding quotation marks:
```
"User Name@[laptop-local-address]"
```

Run the following (in one line), replacing [username] with your HPCC username, which we're providing. This sends a copy of your public key to the remote server:

```
cat ~/.ssh/id_rsa.pub | ssh [username]@hpcc-cluster.stanford.edu "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

After you copy the SSH public key to the server, you will see a warning like this:
```
The authenticity of host 'hpcc-cluster (171.64.55.130)' can't be established.
ECDSA key fingerprint is SHA256:20rcvjngfjkrjjank45436tkjfhsdkfsdHjosfjJhk.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
```

Type _yes_ and press the enter key.

You'll receive a response like this:
```
Warning: Permanently added 'hpcc-cluster,171.64.55.130' (ECDSA) to list of hosts
```

You'll be prompted for your HPCC password, which we're providing.
Type your password and press enter.
**NOTE** Your keystrokes will not appear on the screen.
```
user@hpcc-cluster.stanford.edu's password:
```

You likely won't receive a response after entering the password.

**2: Copy the SSH Public Key to the Server - For MacOS & Linux Users ONLY**

Run the following, replacing [username] with your HPCC username, which we're providing:

ssh-copy-id [username]@hpcc-cluster.stanford.edu
```
~$ ssh-copy-id david@hpcc-cluster.stanford.edu
```

You will get a warning like this. If you entered the server address correctly, you're fine.
Type **yes** and press the enter key.

```
/usr/bin/ssh-copy-id: INFO: Source of key(s): "/home/kali/.ssh/id_rsa.pub"
The authenticity of host 'hpcc-cluster (171.64.55.130)' can't be established.
ECDSA key fingerprint is SHA256:20crZ5XWQNxMgW0BZo3mE97j7z2ThNoULDzVeUAVeCg.
Are you sure you want to continue connecting (yes/no/[fingerprint]) yes?
```

You'll receive a response like this:
```
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter
out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are
prompted now it is to install new keys
```

You'll be prompted for your HPCC password, which we're providing.
Type your password and press enter.
Note: Your keystrokes will not appear on the screen.
```
davidron@hpcc-cluster.stanford.edu's password:
```

You'll receive a response like this:
```
Number of key(s) added: 1
```

**3: Test SSH Key Authentication**

Run the following, replacing _[your username]_ with your HPCC username, which we're providing:

```
ssh [your username]@hpcc-cluster.stanford.edu
```

If SSH key authentication worked, you should get a response like this:
```
Last login: Thu Jul 15 20:11:39 2021 from dn2lk5ehf.stanford.edu
[davidron@hpcc-cluster ~]$
```
