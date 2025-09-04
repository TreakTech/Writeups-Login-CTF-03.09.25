In this challenge we are sent to website where we create a user.

<img width="600" height="300" alt="image" src="https://github.com/user-attachments/assets/e10be95a-a90c-43f9-879d-4eee15da833b" />

Then we are taken to this page with a button which says "get flag".

<img width="600" height="300" alt="{C04EBE10-7A33-4D06-8F0A-BD50F007F2AB}" src="https://github.com/user-attachments/assets/9cd5dc52-eedd-47d9-a405-3e5078ced540" />

But when entering the site we learn that we are not admins and can't view the contents.

<img width="500" height="300" alt="{2BAF7466-D548-491B-969C-CF155D06C4AE}" src="https://github.com/user-attachments/assets/e152ee06-7826-4777-966a-5659eb0dc468" />

So we check the cookies and see that we have a Json Web token (JWT).

<img width="600" height="400" alt="image" src="https://github.com/user-attachments/assets/8286119b-0ad8-4ea9-b7b7-91dc3e09f97e" />

By taking this token to jwt.io we can read the contents and as you can see admin is indeed set to false. So our goal is now to change this value to yes.

But this cant just be done because the token is encrypted with a password.

<img width="600" height="300" alt="{6EF4E02E-AEE9-464E-8775-C947B394FF89}" src="https://github.com/user-attachments/assets/b4913773-26f8-4929-ad21-b0a7f8f91ecb" />

So we use Hashcat with the wordlist "rockyou.txt" to discover the password:

<img width="600" height="200" alt="{F2801384-7EE3-4A13-BE18-D6021DB4E81B}" src="https://github.com/user-attachments/assets/0f17dbe4-7b2f-49f0-b705-4b42d3e919b3" />

After running the command we are given the password: duckhunter23.

<img width="1663" height="1064" alt="{A799EE72-32BB-4B20-A63E-DC01C77E2F8B}" src="https://github.com/user-attachments/assets/a11b6a2f-5cec-4806-bcde-262497b3d3e3" />

Now we can take this password back to jwt.io, change from decode to encode and write in the password. Then we edit admin from false to true.

Then copy the new jwt token and return to the site, change the old token with the new one and proceed to the site with the flag.

<img width="600" height="350" alt="{0C3AF003-9B88-43BC-90FC-9CB4EDE54806}" src="https://github.com/user-attachments/assets/ff3374da-5463-4fbf-a138-fb85e2a5da38" />

As you can see we now have access to the site and the flag is revealed:

<img width="699" height="350" alt="{1E305918-BE7B-4D5D-AD7B-619B27628972}" src="https://github.com/user-attachments/assets/26f545a7-55d1-4153-a301-2e5a9c25b2e9" />
