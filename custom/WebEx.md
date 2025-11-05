# Web Exploitation 

## Database Incursion 2.0 
In this challenge, we had to look around a database and try to gain access to admin area which would allow us to find the flag. 

### Solution 
To solve this challenge, I followed the steps as given below: 
- I first opened the google drive to find the link and open the login page. This was pretty similar to a challenge we had solved in Citadel-CTF.
- Using similar reasoning, I logging in without facing much issues.
![Image of Login page with `' OR 1=1 --`](/images/custom/WebEx/Screenshot-2025-11-05-19-40-06.png)
- After logging in, I looked around to see that Drake was telling me that someone from management has the passcode. Again using similar logic that I used in Citadel, I set `OFFSET` to find the person with the password so I could acess Admin Area
![Image of Searching name with `' OR 1=1 LIMIT 4 OFFSET 10 --`](/images/custom/WebEx/Screenshot-2025-11-05-19-41-58.png)
- This gave me the passcode to move onto the next stage. Here is where things got interesting. Now because there were multiple tables, I had to go and read through resources to understand how exactly we can acess tables. Here I stumbled across the `UNION` keyword. I used this to try to get data from `metadata` table. Luckily it worked on my first try.
- However, when I tried accessing data from the `CITADEL_ARCHIVE_2077` it was not allowing me to access the data and showing an error as below:
![Image of columns not matching](/images/custom/WebEx/Screenshot-2025-11-05-19-43-41.png)
- It took me a while to understand what was happening, and I had to read up properly on the `UNION` command. What I learned after some reasearch is that `UNION` needs to have the same number of columns on both sides of the command, but the error was showing that there were not equal columns on each side.
- Hence I tried various methods of trying to find out the column names of the `CITADEL_ARCHIVE_2077` table, without much gain. (More in Notes)
- I came across some article which mentioned that we could also leave the column names as empty. Then it struck me all of a sudden that on reading the `Revenue` column, we had something called `secrets` in it. This could be a column name.
- Using this as an insight I crafted a query as shown to get the flag:
![Image of `' UNION SELECT secrets, NULL, NULL, NULL from CITADEL_ARCHIVE_2077`](/images/custom/WebEx/Screenshot-2025-11-05-19-45-29.png)

### Flag 
`CITADEL{571ll_d0n7_kn0w_1f_175_53qu3l_0r_5ql?}`

### Notes 
1. I tried using `PRAGMA table_info()` but that was just useless.
2. I tried using `DESCRIBE` as well, but same issue as above.
3. I tired using `;` but we could only pass one query at a time, not multiple.

### What I learned
From this challenge, I learned about how to use `UNION` properly to access details from other tables and display it. This is a really fruitful thing to know, espeically when we are sure that we are in a database, but need to find information about other tables. The usage of `NULL` to fill out the extra columns was really amazing, and honestly quite impressive, yet logical. 

### References
- https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master
- https://www.w3schools.com/sql/sql_union.asp
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
