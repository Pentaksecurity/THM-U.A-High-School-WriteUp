# THM-U.A-High-School-WriteUp

# Recon using Zenmap

<img width="973" alt="Screen Shot 2025-03-24 at 7 37 59 PM" src="https://github.com/user-attachments/assets/b1d85a60-15ad-4628-8781-85a0e01f834d" />

I used Zenmap to discover any open ports, services and OS versions running. I discovered open port 80 running HTTP and port 22 running SSH. Other than that nothing really interested came up on Zenmap so I decided to check out the website. Nothing really gave me anything hopefull except the contact page, but after intercepting my "test" request with Burpsuite I realized it went nowhere as well so I decided to start enumerating.

# Enumeration 

I decided to use gobuster for this one and just used the "common.txt" wordlist.

<img width="1121" alt="Screen Shot 2025-03-24 at 7 38 43 PM" src="https://github.com/user-attachments/assets/77c75b01-bc5c-472b-9275-8ea4b1f1f4b3" />

I discovered the "assets" directory which was blank so I decided to see if there was any directories within the "assets" page so I decided to enumerate it.

<img width="1137" alt="Screen Shot 2025-03-24 at 7 39 04 PM" src="https://github.com/user-attachments/assets/3ef64787-fb4a-4db9-b1a2-68d8111c128e" />

Found "images" directory which I didn't have permissions to view and the "index.php" and PHP means commands!

I tried to just run "ls" to see what happens.

<img width="1663" alt="Screen Shot 2025-03-24 at 7 39 25 PM" src="https://github.com/user-attachments/assets/3aa0b49b-c616-4a1c-97d6-d987a66358f7" />

Which gave me a base64 enconded string so I went to Cyberchef to decode, and got the result which was "Images, index.php and styless.css".

<img width="847" alt="Screen Shot 2025-03-24 at 7 40 11 PM" src="https://github.com/user-attachments/assets/22ba468a-3934-44a7-9539-1d7ebcc83255" />

Nothing that I can really do with it but that means the my command worked! So it's time for PHP shell.

# Time to get a shell

<img width="1676" alt="Screen Shot 2025-03-24 at 7 40 27 PM" src="https://github.com/user-attachments/assets/1877f3bc-d5b8-4ca8-9223-dbe4f5e515d6" />

Went to Reverse Shell generator and got this PHP exec shell (make sure you have URL encoding turned on!) 
Started my netcat listener on port 5555 and ran my shell in the browser.

<img width="1677" alt="Screen Shot 2025-03-24 at 7 40 39 PM" src="https://github.com/user-attachments/assets/1d03be58-5e58-4bc7-a4fc-3f52dd7c9a97" />

Got the shell and upgraded to a more stable shell usgin 'python3 -c 'import pty; pty.spawn("/bin/bash")';



<img width="771" alt="Screen Shot 2025-03-24 at 7 41 00 PM" src="https://github.com/user-attachments/assets/009353a6-0cd8-41b3-b5ba-ef8784097d72" />

Went back to '/var/www' directory and found this "Hidden_Content" directory in there! Which had a "passphrase.txt" file with another base64 encoded password.

<img width="783" alt="Screen Shot 2025-03-24 at 7 42 27 PM" src="https://github.com/user-attachments/assets/c4806449-bb11-4f82-b66b-526f6f6846b1" />

Went back to Cyberchef got that decoded and went back to get the images back to my machine to use steghide.

<img width="831" alt="Screen Shot 2025-03-24 at 7 42 44 PM" src="https://github.com/user-attachments/assets/4de6f611-7b16-4c44-992d-0944ecaefb37" />

<img width="1014" alt="Screen Shot 2025-03-24 at 7 43 21 PM" src="https://github.com/user-attachments/assets/e4ae8fed-9e7d-41f8-a6fd-eed86c0e4ae8" />
<img width="720" alt="Screen Shot 2025-03-24 at 7 43 31 PM" src="https://github.com/user-attachments/assets/66a2be92-a6d9-4941-9cf7-934666c0f639" />

So the "oneforall.jpg: was corrupted so I found this tool called magicbyte helps you repair corrupted images.

Link to magicbyte: https://github.com/Haxrein/MagicBytes

<img width="1091" alt="Screen Shot 2025-03-24 at 7 44 39 PM" src="https://github.com/user-attachments/assets/287ecd5c-6e58-433d-b0d9-7e8eb032654a" />

So we repaired the image using magicbyte (python3 magicbyte.py -i oneforall.jpg -m jpg) and used steghide (steghide extract -sf oneforall.jpg) with our "Hidden_Content" passphrase and got the password for deku!


<img width="1663" alt="Screen Shot 2025-03-24 at 7 45 12 PM" src="https://github.com/user-attachments/assets/3cb060d2-a7d1-4dbf-a638-ea5d997df6db" />

Time to SSH into the machine with our deku password and get the user flag!

<img width="718" alt="Screen Shot 2025-03-24 at 7 45 27 PM" src="https://github.com/user-attachments/assets/22b7d5b6-ca9b-4e02-8aea-f7548f348338" />

# Privilege Escalation

Now it's time for the money shot. We gotta figure out to escalate our privileges.

<img width="1334" alt="Screen Shot 2025-03-24 at 7 46 19 PM" src="https://github.com/user-attachments/assets/d720f5a8-a711-4d54-98f5-89f1f48da769" />

sudo -l

And we can see that we can run the /opt/NewComponent/feedback.sh


<img width="1673" alt="Screen Shot 2025-03-24 at 7 46 00 PM" src="https://github.com/user-attachments/assets/d3754701-b714-41b2-a975-0b0e81fc074b" />

I tried to edit the file but it was unwritable, but then I realised that 'eval' might actually execute my command!

So gave it a shot to give me root priveleges.

deku ALL=NOPASSWD: ALL >> /etc/sudoers
And sudo -l again and we got it!


<img width="1440" alt="Screen Shot 2025-03-24 at 7 47 19 PM" src="https://github.com/user-attachments/assets/e0d51d4e-a8a4-4db6-9be0-9002fa0fe15a" />

Now we can get the root flag!


<img width="970" alt="Screen Shot 2025-03-24 at 7 47 30 PM" src="https://github.com/user-attachments/assets/bd66cd83-38d9-46a1-b038-400fb4e4c1f1" />
