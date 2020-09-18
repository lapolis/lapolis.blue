---
title: "OSCP Survival Techniques 101"
date: 2020-09-15T14:34:16+01:00
draft: true
---
With this short post I want to put together a few very good tips I found on the internet about how to prepare for and how to survive the OSCP exam.
This is **not** a tool-kit since you will just need a basic Kali installation for your OSCP exam.  

___  
## Preparing for the exam
Since the **OSCP** certification is relatively expensive, especially if you're still a student. However, there are simple ways to start preparing even before enrolling in order to save at least a month of paid lab.  

A huge thanks goes to this magnificent list of [HTB/VulnHub machines](https://docs.google.com/spreadsheets/d/1dwSMIAPIam0PuRBkCiDI88pU3yzrqqHkDtBngUHNCw8/edit#gid=1839402159); I have personally tackled just the HTB OSCP-like VMs, excluding a few of the "More challenging than OSCP" ones.  
I strongly recommend that, even after pwning each machine, you watch the walkthrough made by [IppSec](https://www.youtube.com/channel/UCa6eh7gCkpPo5XXUDfygQQA) or, even better, watch all of the HTB videos made by [s4vitar](https://www.youtube.com/c/s4vitar) .. yeah I know, he speaks Spanish, I do not know Spanish myself `(actually, I have learnt few words watching the videos :P)` but s4vitar shows and graphically explains everything step by step making the videos self explanatory.  
Watching other people's walkthrough can help to learn new techniques or just to improve your own one, at least that's what happened to me.  
  
If you still have time, it would be a good idea to do all the easy and medium active machines. If you want to save a bit of time, just pick the one marked as CVE/real-life/enumeration, you do not really need to get mental on a CTF like machine.  
  
If someone tells you `"buy just two months of lab"`, you gotta believe him/her. Obviously I did not, I thought that it would have been good to have 3 months time to pwn the whole lab `(reminder: I was not working during those 3 months)` and I ended up wasting the last month playing HTB.  
___
## The actual exam
**REMEMBER**: you are not looking to find any 0day! If you do your port scan right, you already have all you need to work toward your foothold.  
For the privesc, I would say that you just need to fire up [`linPEAS|winPEAS`](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite), but **very basic** enumeration should be more than enough. I personally ended up not even reading the output of `linPEAS|winPEAS` after running it.  
  
A few days before the exam, I read a citation in a post about OSCP (which I cannot find any more) that explicitly said that during the exam, you will finish all of your ideas before your 24 hours end. With that said, set a `2 hours alarm` and every time it fires off, just take a walk around the block. During those 2 hours, if you did not make any improvements on the machine you are working on, switch to another one.  
  
I thought I would never say what I am going to say: `drink a lot of water`. Your brain will melt during those 24 hours, it needs fluids. About food, try to eat low fat high fibres, the last thing you want is to fight with the sleepiness after a huge lunch.  
  
Do not be nervous `(I wasn't, I swear! :P)`, especially the day before the exam. Just rest, do not do anything related to the OSCP, do something relaxing `(I swore the whole day against a f\*!$@&g Pi)`.  
  
Once you survive the first 24 hours and you got at least 70 points, it is report time. You can either use the [whoisflynn](https://github.com/whoisflynn/OSCP-Exam-Report-Template) template with a little personalization or the magnificent [Offensive Security Exam Report Template in Markdown](https://github.com/noraj/OSCP-Exam-Report-Template-Markdown) by noraj. 
___  
## The Glory
As soon as you get the confirmation of correctly submitting the report, open a beer, get a shower, open a beer, open a beer, open a beer and go to sleep. You deserve it.  
  
Following those few tips, a simple noob like myself can easily pass the OSCP examination. The timeline of my almost 16 hours-long exam:

|Time               |&nbsp;&nbsp;&nbsp;Machine point  |
|----------         |----------     |
|1 hours 30 mins    |&nbsp;&nbsp;&nbsp;bof - 25      |
|+ 6 hours          |&nbsp;&nbsp;&nbsp;25            |
|+ 4 hours 30 mins  |&nbsp;&nbsp;&nbsp;20            |
|+ 2 hours		    |&nbsp;&nbsp;&nbsp;10			|
|+ 1 hour           |&nbsp;&nbsp;&nbsp;user on the 20 pointer|
