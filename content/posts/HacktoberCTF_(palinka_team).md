---
title: "HacktoberCTF_(palinka_team)"
date: 2020-10-19T08:32:45+01:00
draft: false
toc: false
images:
tags:
  - hacktober
  - ctf
  - writeup
  - palinka_team
---
  
{{< image src="/img/hacktober/00_logo.png" alt="hacktober ctf logo" position="center" style="border-radius: 8px;" >}}  

Not long ago, during a regular afternoon, I thought to myself: "I feel like I want to suffer a bit". I quickly checked what [CTF Time](https://ctftime.org/) had on offer and there it was, the answer to all my dreams: [Hacktober CTF](https://ctftime.org/event/1108).  
It was at this point, without pressuring of course... :P , I check to see if anyone wanted to join me in this, probably painful, adventure.  

{{< image src="/img/hacktober/1_rec.png" alt="Recruitment" position="center" style="border-radius: 8px;" >}}  

Still, no pressure.. ..  

{{< image src="/img/hacktober/2_gr0.png" alt="Gyorgy!" position="center" style="border-radius: 8px;" >}}  

After no more than a day or two, the fucking dream team was complete! We were ready to go against stuff we had no idea how to solve, ready to cry fighting against those 59 challenges, ready to learn as much as we could but definitely **not** ready to give up  

{{< image src="/img/hacktober/3_dreamteam.png" alt="dream team" position="center" style="border-radius: 8px;" >}}  

Let me introduce the members of the palinka_team:
- gr0g101 aka [Gyorgy](https://www.linkedin.com/in/gyorgy-antal/)
- kaos aka [Dennis](https://www.linkedin.com/in/dennis-varischetti/)
- fluffiie aka [Maider](https://www.linkedin.com/in/mdrurb/)  
- lapolis aka [Nicola](https://www.linkedin.com/in/nicola-pastres/)

We all helped each other with solving each section, however this is where each team member focused more:  
|User|&nbsp;Category|
|----|----|
|lapolis&nbsp;&nbsp;&nbsp;&nbsp;-->|&nbsp;web, SQL, forensics, Linux|
|kaos&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;-->|&nbsp;programming, forensics|
|fluffiie&nbsp;&nbsp;&nbsp;&nbsp;-->|&nbsp;crypto|
|gr0g101&nbsp;-->|&nbsp;Traffic analysis, stego, forensics|  
  
We really want to mention that the whole CTF was a fun competition with nice and creative challenges. A huge shout out to the organisers!!

The following write-ups are a selection of the most interesting challenges, chosen by the whole team, showing how we solved them and the thought processes behind them.  
  
{{< badge >}}
  

___
## 90s Kids

For this challenge we had to run some queries on a database recovered from a dump.  
Wait.. What?? No SQL injections?? Who the hell ever did queries to a legit db??  
Well, first thing was to set up a new db called `westridge` to recover the dump which required some cleaning.
```bash
unzip shallowgraveu.zip
sed -i 's/utf8mb4/utf8/g' shallowgraveu.sql
sed -i 's/utf8mb4_0900_ai_ci/utf8_general_ci/g' shallowgraveu.sql
sudo mysql -e "create database westridge;"
sudo mysql westridge < shallowgraveu.sql
```

{{< image src="/img/hacktober/5_90s.png" alt="90 kids" position="center" style="border-radius: 8px;" >}}  

Mmmmh.. this requires a regex for sure. No wait, MySQL knows exactly what a date is? Well.. Nice!  
```sql
SELECT count(user_id) FROM users 
WHERE month(dob) = 10 
AND year(dob) 
BETWEEN 1990 AND 1999;
```  
___
## Student Body

{{< image src="/img/hacktober/6_stud.png" alt="student body" position="center" style="border-radius: 8px;" >}}  
  
Well, this one was hard. First we had to find out the instructor's name and surname.

```sql
SELECT e.user_id, 
luc.first, 
luc.last, 
e.term_crs_id, 
c.title, 
tc.instructor, 
concat(prof.first,prof.last) as prof, 
r.role_name  
FROM enrollments e 
JOIN term_courses tc on tc.term_crs_id = e.term_crs_id 
JOIN courses c on tc.course_id = c.course_id 
JOIN users prof on tc.instructor = prof.user_id 
JOIN users luc on luc.user_id = e.user_id 
JOIN roles_assigned ra on ra.user_id = prof.user_id 
JOIN roles r on r.role_id = ra.role_id 
WHERE e.user_id = ( 
  SELECT user_id FROM users WHERE first = 'Lucia' ) 
AND tc.course_id = (
  SELECT course_id FROM courses WHERE title = 'SOCI424');
```  

{{< image src="/img/hacktober/27_proff.png" alt="professor" position="center" style="border-radius: 8px;" >}}  

Finally, we could query the students headcount:
```sql
SELECT count(user_id) 
FROM enrollments 
WHERE term_crs_id in ( 
  SELECT term_crs_id FROM term_courses WHERE instructor = 480 );
```  

{{< image src="/img/hacktober/28_dbflag.png" alt="db flaggg" position="center" style="border-radius: 8px;" >}}  

___
## Jigsaw  

{{< image src="/img/hacktober/7_jig.png" alt="jugsaw" position="center" style="border-radius: 8px;" >}}  

Finally something that we like, regex time!

```sql
SELECT username,last 
FROM users 
WHERE last regexp '^[K,R,I]{2}[^\n][[:alpha:]]{3}[E-N]$';
```  

{{< image src="/img/hacktober/29_regflag.png" alt="regexxxxx" position="center" style="border-radius: 8px;" >}}  

Ok that was quick :(  

___
## Shellcode extraction

{{< image src="/img/hacktober/8_shell.png" alt="shellcode extraction" position="center" style="border-radius: 8px;" >}}  

Our first idea was to convert it to a binary file and try to analyse or disassemble it, then figure out the name of the created file.  
```bash
wget -O shellcode.hex https://tinyurl.com/y2ra3pzj
cat shellcode.hex | xxd -r -p > shellcode.bin
```  
Using xxd we could check for any potential strings, or hints or really *JUST ANYTHING USEFUL*, unfortunately no luck.  
We also tried to disassemble it with [radare2](https://rada.re/n/radare2.html) and make some sense out of it, no success here.  
Then, IDA tried to rescue us; we threw the shellcode into IDA hoping to discover something juicy and we realised that it tried to interact with shell32.dll.  

{{< image src="/img/hacktober/9_shell.png" alt="shellcode" position="center" style="border-radius: 8px;" >}}  

This is a Windows API that contains functions used for opening files or web pages... Mmmmmh... Let's switch to Windows then... Wait! Do you really want to run a piece of shellcode from real malware on your PC!?!? Are you mental? Well... We leave it to you, to decide that... All we knew, was.... It had to be solved at all costs!  
  
After careful researching (banging our heads against the wall and crying) we came across a great tool called [scdbg](http://sandsprite.com/blogs/index.php?uid=7&pid=152). A huge "THANK YOU" TO THE CREATORS of this amazing tool. It is capable of analysing shellcode at runtime and logs every detail, such as API calls and interactive hooks, in a simulated virtual environment. It is even possible to integrate it with IDA.  
  
We need to activate report mode and API table scan to see if any function is called from shell32.dll to download or create a file. If yes, then we smashed it because the full path will be passed as an argument and hopefully it will be the ROAMING folder. So let’s see...  

{{< image src="/img/hacktober/10_dbg.png" alt="debuggggg" position="center" style="border-radius: 8px;" >}}  
  


{{< image src="/img/hacktober/11_done.png" alt="shellcode done" position="center" style="border-radius: 8px;" >}}  
  
And yesssss we won!!! There we have it... The shellcode calls URLDownloadToFileA with 2 arguments. One is the malicious exe's location and the other is the full path that the downloaded file will be written to. The exe file lands inside the ROAMING folder under the current user. It also executes that file using the ShellExecuteA function. This challenge was an emotional roller-coaster, but we managed to learn from it AND discovered an amazing tool.  
  
{{< image src="/img/hacktober/12_memeshelll.png" alt="meme shellcode" position="center" style="border-radius: 8px;" >}}  
  
___
## Red Rum
  
{{< image src="/img/hacktober/13_code.png" alt="redrum" position="center" style="border-radius: 8px;" >}}  
  
The challenge "Red Rum" asked us to create a list of numbers between 1 and 500 and to replace some of them as the following:
- numbers divisible by 3 with "Red"
- numbers divisible by 5 with "Rum"
- numbers divisible by both 3 AND 5 with "RedRum"  

By netcatting env2.hacktober.io we got one more hint: "Your answer should be comma-separated with no spaces".  
  
{{< image src="/img/hacktober/14_hintnc.png" alt="nc hint" position="center" style="border-radius: 8px;" >}}  
  
After some coding... and swearing, we got it.  
  
```py
import socket 

def rum(out=''):
    for i in range (1,501):
        a, b, = i % 3, i % 5
        c = (a + b)
        if   c == 0 : out += 'RedRum,'
        elif a == 0 : out += 'Red,'
        elif b == 0 : out += 'Rum,'
        else: out += f'{str(i)},'

    return f'{out[:-1]}\n'

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(('143.110.147.190', 5000))
s.sendall(rum().encode())
s.shutdown(socket.SHUT_WR)

data = '!!--palinka_team--!!'
while data:
    data = s.recv(1024)
    print (f'{data.decode()}')

s.close()
```  
  
{{< image src="/img/hacktober/15_flagssss.png" alt="flaggggg" position="center" style="border-radius: 8px;" >}}  
  
___
## Boney Boi Breakdance

{{< image src="/img/hacktober/16_bboy.png" alt="b-boy" position="center" style="border-radius: 8px;" >}}  
  
This challenge involved a little bit of digging around and guessing. We tried the usual tools to carve some information out from the picture, but nothing was working.  
  
Combining the hint from the description and the fact that [steghide](http://steghide.sourceforge.net/) expected a pass phrase we thought we had to find something about the picture itself. A little bit of googling directed us to a [Wikipedia page](https://en.wikipedia.org/wiki/Danse_Macabre) containing a description of the picture. Scrolling through that page, right at the bottom, we found an interesting sentence mentioning the title of the picture and the author... Mmmmh... That could be the pass phrase.  
  
{{< image src="/img/hacktober/17_wolgemut.png" alt="pass phrase" position="center" style="border-radius: 8px;" >}}  
  
{{< image src="/img/hacktober/18_passphrase.png" alt="steghide" position="center" style="border-radius: 8px;" >}}  
  
{{< image src="/img/hacktober/19_flag.png" alt="flaaaaagggggg" position="center" style="border-radius: 8px;" >}}  

___
## An evil christmas carol 3
  
{{< image src="/img/hacktober/20_evil.png" alt="evil carol" position="center" style="border-radius: 8px;" >}}  

Ok... there we were, with bleeding eyes, trying to solve the final session of traffic analysis. The task was to find out the type of malware family, so we fired up [Wireshark](https://www.wireshark.org/), used couple of eye drops, a quick Palinka and ready to dive in.  
  
Since the malware was downloaded from a remote source, we should be able to see a clear destination IP and the name of the malicious file.  

{{< image src="/img/hacktober/21_ipssss.png" alt="evil carol" position="center" style="border-radius: 8px;" >}}  
  
Shortly after, we found a nice site which can provide information on specific malware by its name or url. Doing a search for either “july22.dll” or “MwRrN5” throw us all the info we need. The malware type was there waiting for us. The last phase could be solved in a similar fashion.  
  
{{< image src="/img/hacktober/22_mal-flag.png" alt="flags everywhereeeeeeee" position="center" style="border-radius: 8px;" >}}  

___
## Hell Spawn 1

{{< image src="/img/hacktober/23_hell.png" alt="hell spawn" position="center" style="border-radius: 8px;" >}}  
  
Memory dump analysis? WTF is that? Ooooh, is it that magic thing done with [volatily](https://www.volatilityfoundation.org/)? Yeah that's right.  
For this challenge we needed to find out which was the process that spawned the malicious "explorer.exe". First we need to find the the profile used.  
```bash
volatility imageinfo -f ./mem.raw
```  

{{< image src="/img/hacktober/24_prof.png" alt="profile" position="center" style="border-radius: 8px;" >}}  
  
Having the profile, it was possible to make some sense from this bunch of data. Using the "cmdline" function we easily get a list of all the commands.  
```bash
volatility --profile=Win10x64_17134 cmdline -f ./mem.raw 
```  
  
{{< image src="/img/hacktober/25_cmline.png" alt="cmdline" position="center" style="border-radius: 8px;" >}}  
  
In order to confirm which process spawned which process, we used "pstree".  
```bash
volatility --profile=Win10x64_17134 pstree -f ./mem.raw
```  
{{< image src="/img/hacktober/26_pstree.png" alt="pstree" position="center" style="border-radius: 8px;" >}}  
  
