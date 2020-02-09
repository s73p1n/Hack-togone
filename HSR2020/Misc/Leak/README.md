# Leak : 125 points

Unfortunatly, I don't have the challenge description but it was something like : 
"We heard that someone have found a new way of exchanging informations. Find what have been leaked."
and the file hsleak.wav was given.

Knowing that it was a WAVE audio, I launched the Sonic Visualiser and opened the hsleak.wav file to see the audio spectogram.

![Image](./Images/hsrleak_spectogram.png)

We could clear see a pastebin link (hxxps://pastebin.com/VRAKUxT9) which contains some facebook account emails and passwords.
The pastebin lastely modified in 2017. 

So it seems like those passwords could be helpful for something. I went back to the audio file to see all contained strings.

```shell
[CTF] (~/HSR/) $ strings -a hsrleak.wav
...
flag.pngUT
...
```
We can suppose that it's a compressed file, let's try to unzip the file

```shell
[CTF] (~/HSR/) $ unzip hsrleak.wav 
Archive:  hsrleak.wav
warning [hsrleak.wav]:  2645804 extra bytes at beginning or within zipfile
  (attempting to process anyway)
[hsrleak.wav] flag.png password: 
   skipping: flag.png                incorrect password
```

It's a protected zip.

Let's make a dictionary attack to try the unprotect the zip.


```shell
[CTF] (~/HSR/) $ zip2john hsrleak.wav > breame
```

I tried to break that zip password with the rockyou dictionary.

```shell
[CTF] (~/HSR/) $ john --wordlist=/usr/shar/wordlists/rockyou.txt breakme
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
0g 0:00:00:02 DONE (2020-02-09 11:58) 0g/s 6639Kp/s 6639Kc/s 6639KC/s !!rebound!!..*7¡Vamos!
Session completed
```

The password is not contained in rockyou, let's try to break the protected zip with pastebin passwords.

```python
[CTF] (~/HSR/) $ cat extract_pwd.py 
file_content = open("pastbin_content.txt", 'r').read().split()
dico = open("dico.txt", 'w')
for usr_pwd in file_content :
    print(usr_pwd)
    if ':' in usr_pwd :
        dico.write(usr_pwd.split(':')[1]+'\n')
    else :
        try :
            dico.write(usr_pwd.split('.com')[1]+'\n')
        except :
            dico.write(usr_pwd+'\n')
```

```shell
[CTF] (~/HSR/) $ john --wordlist=dico.txt breakme
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
3yalh42hl        (hsrleak.wav/flag.png)
1g 0:00:00:00 DONE (2020-02-09 12:13) 33.33g/s 23266p/s 23266c/s 23266C/s joyce1717..khender494
Use the "--show" option to display all of the cracked passwords reliably
Session completed

[CTF] (~/HSR/) $ unzip -P 3yalh42hl hsrleak.wav 
Archive:  hsrleak.wav
warning [hsrleak.wav]:  2645804 extra bytes at beginning or within zipfile
  (attempting to process anyway)
  inflating: flag.png 
```

![Image](./Images/flag.png) 
