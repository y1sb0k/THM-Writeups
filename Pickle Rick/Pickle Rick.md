# 🥒 Pickle Rick - TryHackMe Room
### 🔗 Link to Room: [Pickle Rick - THM](https://tryhackme.com/room/picklerick)

Welcome to my first ever CTF Writeup! I am writing this writeup to both help develop myself and my skills, whilst hopefully guiding others who are new to the field of Web Application Penetration Testing.
Please forgive any unusual styling or writing choices I make, as I develop my skillset! 
Without further ado, lets dive in the Pickle Rick room by TryHackMe
For reference, I have already completed this THM Room, and I will be re-creating and documenting my steps.

In this room, we are given very little context, only:

This Rick and Morty-themed challenge requires you to exploit a web server and find three ingredients to help Rick make his potion and transform himself back into a human from a pickle.

Deploy the virtual machine on this task and explore the web application: 10.130.170.42

From this small intro, we know three things;
The target IP - 10.130.170.42 (Note that this will be different for you for your own Machine.
We are attacking a web server
There are 3 flags to find.

Let's start with some initial Enumeration.

Using the Nmap will allow us to take a look at the server, specifically identifying ports and services it may be running that may reveal potential vulnerabilities. I have used the command:

nmap -A -v 10.130.170.42

Here the -A flag is an Aggressive Scan, which accumulates multiple nmap flags into one argument, namely; -sV (Services & Version Detection), -o (OS Detection), -sC (Default Script Scanning) and a traceroute function. I have also used the -v (Verbose flag).


As shown in the output, we have two open ports, ssh running on port 22, and http running on port 80.

Lets navigate to the website, and have a look around there, looking at our nmap results, this should be hosted on http://10.130.170.42:80

Nice, it looks like we have a webpage here, initially looking around and attempting to interact with the webpage doesn’t provide any further details, nor does the content of the webpage.


However, lets take a look at the source code of the page.

Right there in the html source code, we have a Username, commented out so it does not display on the webpage, website developers/administrators often use this as a weak way to hide credentials or details for other admins or devs for future reference.

Source Code:

<!DOCTYPE html>
<html lang="en">
<head>
  <title>Rick is sup4r cool</title>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="stylesheet" href="assets/bootstrap.min.css">
  <script src="assets/jquery.min.js"></script>
  <script src="assets/bootstrap.min.js"></script>
  <style>
  .jumbotron {
    background-image: url("assets/rickandmorty.jpeg");
    background-size: cover;
    height: 340px;
  }
  </style>
</head>
<body>

  <div class="container">
    <div class="jumbotron"></div>
    <h1>Help Morty!</h1></br>
    <p>Listen Morty... I need your help, I've turned myself into a pickle again and this time I can't change back!</p></br>
    <p>I need you to <b>*BURRRP*</b>....Morty, logon to my computer and find the last three secret ingredients to finish my pickle-reverse potion. The only problem is,
    I have no idea what the <b>*BURRRRRRRRP*</b>, password was! Help Morty, Help!</p></br>
  </div>

  <!--

    Note to self, remember username!

    Username: R1ckRul3s

  -->

</body>
</html>



Username: R1ckRul3s

The existence of a Username suggests that somewhere hosted on this server, there is a login page, to find it I am going to use the tool Gobuster to search for hidden directories hosted on the server. I am using the command:

gobuster dir -u http://10.130.170.42/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x php,html,txt

Here the dir -u argument tells Gobuster our target, the -w argument informs Gobuster of the wordlist to use, and the -x argument tells it to search for specific file extensions, here .php .html and .txt files.


The output generated has given us 10 directories, we are only interested in the ones that returned with the HTTP status code of 200, as this indicates a successful connection. So we have:
/index.html
/login.php
/robots.txt

Since robots.txt files usually provide web crawlers and search engines about which pages and directories are permitted to be indexed, this is probably a good place to start, to see if there are any directories that Gobuster didn’t find due to our limited wordlist.


Interesting, no visible directories or pages here, just the word “Wubbalubbadubdub”, perhaps this is a clue, a password, or simply something to throw us off the scent.
Lets check out that /login.php website to see if we can use Wubbalubbadubdub as a password.



Nice, it looks like we are in! The page displays a “Rick Portal”, with a command panel, along with some other links along the top menu. Looking through these links, we are greeted by the /denied.php directory, and the message “Only the REAL rick can view this page..”, perhaps there is some privilege escalation required to an administrator account to view these pages..

Let's take a look at the command panel, perhaps we can attempt some command injection here. I am going to start with a simple ls command, to see if we can enumerate any files on the server. 

This shows a file named Sup3rS3cretPickl3Ingred.txt, this potentially contains a flag, lets have a look at it using cat Sup3rS3cretPickl3Ingred.txt.

Hm it would seem that the use of cat is disabled here, trying other tools such as nano and vim also both show the same error. 
Perhaps something simpler would suffice, by putting the file name directly into the URL to see if there is an IDOR vulnerability here. 


Nice! We have the first flag!

Flag 1: mr. meeseek hair

Going back to our Gobuster results, we have the file /clue.txt, let's check that out. 

Looks like we are going to need to do some more digging around the file directory to see what else we can find here.

Back in the Command Panel, by using pwd, I can see that we are in the /var/www/html directory, I want to see what users I can discover on the machine, by using ls -la /home, I should be able to print the home directory to see who is in there.
Note - using the flags -la allows us to add two flags to the ls command, -l for long format which shows us file permissions, owner, and group of the file owner, file size, and the “Last Modified” date.
This shows us there is a “rick” user and an “ubuntu” user.


Lets then check out what is in the /home/rick directory, to see if the second flag is in there using the command ls -la /home/rick

Okay, looks like we have the file for the second ingredient here, however, the same IDOR vulnerability used before won’t work again here without the file extension. 

Since it would appear that most common ways to view this file are being blocked successfully, I am attempting to use inline bash commands, to read through the file and output the file contents, I did have to do some research externally for this, and consulted good old AI agents to ensure my thinking and coding was correct, however, the following command:

while read line; do echo $line; done < /home/rick/second ingredients

This also did not provide an output, doing some further digging I discovered this is an issue caused by the space character, and the use of a backslash must be used to escape the space in the file name, so the command becomes:

while read line; do echo $line; done < /home/rick/second\ ingredients

It worked! Tah-dah!! We finally have our second flag.



Flag 2: 1 jerry tear

You may notice a change of IP address at this point to 10.130.158.202 - writing and hacking simultaneously is hard and slow work!

After looking around in some additional source code of the webpage, and trying some further enumeration of the directories through Gobuster and Ffuf, it became apparent that nothing was else obvious was hidden around here, I decided to check in the root directory, to access this directory we must use the sudo command, so I used the command:

sudo ls -la /root

This shows us a file called 3rd.txt - likely our third flag! I am going to attempt to make a copy of the file in our current directory, and recreate the IDOR vulnerability once this file is moved over, I copy the file to the current working directory using the command:

sudo cp /root/3rd.txt ./3rdFlag.txt

Then by executing ls in the Command Panel, I can see that the file has been copied successfully over to our current working directory, similarly to before, I put this file name into the URL.



And there we have our third flag!

Flag 3: fleeb juice

We have effectively obtained the ingredients list to turn Rick back into a human!

I hope you enjoyed my first ever writeup as much as I have enjoyed writing it!

Check out my website for future writeups - https://callumgibl.in/

-y1sb0k
