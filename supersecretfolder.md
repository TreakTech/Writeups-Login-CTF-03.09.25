For this challenge we are given a zip file which contains the flag, the problem is that this zip file has a password which we dont know.

<img width="671" height="247" alt="{7950A47D-5AC1-4F97-B6BC-9C4A36DA863B}" src="https://github.com/user-attachments/assets/82e5027c-bc41-44fd-b02f-2acaf0efe087" />

The description tells u that we need a wordlist.
CTFkom mostly just uses the "rockyou.txt" wordlist if one is needed.

So we can use the linux tool fcrackzip to discover the password with "rockyou.txt"

<img width="1096" height="208" alt="{BFBDA65A-446B-4224-B322-747815A706E0}" src="https://github.com/user-attachments/assets/0b6933e8-4fba-4856-85d0-41e41e35963a" />

Finally we can take this password, open the zip file and open the text file to find the flag:

<img width="831" height="564" alt="{E1C9DC5D-85A3-4421-902C-F510A9118251}" src="https://github.com/user-attachments/assets/f7c5c53e-69c2-44dd-8ad1-684c33ca256a" />
