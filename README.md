# Silly-Putty-Challenge
Investigating my first Malware without guided help!

# My First Solo Malware Analysis

So, here I am, embarking on my first solo malware analysis adventure. Armed with the mighty Cmder, my trusty companion in the world of cybersecurity, I'm on a quest to unravel the secrets hidden within a mysterious binary. But before we dive into the juicy details, let's start with the basics.

## The Hunt for Hashes

The first step in this thrilling journey is to uncover the SHA256 and MD5 hash of our enigmatic binary. These cryptographic signatures are like fingerprints, unique to each file. So, I unleash Cmder to find them:

SHA256: 0c82e654c09c8fd9fdf4899718efa37670974c9eec5a8fc18a167f93cea6ee83
MD5: 334a10500feb0f3444bf2e86ab2e76da

A quick check on [VirusTotal](https://www.virustotal.com/) reveals that our binary is a well-known Trojan Win32.

![VirusTotal](https://github.com/CertainRisk/Silly-Putty-Challenge/assets/141761181/4a9f751d-26f7-4293-b42d-9643239a6fb8)


## String Extravaganza with FLOSS

Next, we put our binary under the spotlight with FLOSS, a tool for extracting strings. There are strings galore. It's a lot but I remember what to look for from previous videos in the course so I will do my best!

Amidst the string salad, I spot numerous references to SSH. URLs like [chiark.greenend.org.uk/~sgtatham/putty/](https://chiark.greenend.org.uk/~sgtatham/putty/) and cryptic `%s==NULL in terminal.c` catch my eye. 

![Floss Str 1](https://github.com/CertainRisk/Silly-Putty-Challenge/assets/141761181/569002dd-fceb-41c1-af61-507d20c8bede)

![Floss Str 2](https://github.com/CertainRisk/Silly-Putty-Challenge/assets/141761181/3c8cf420-09ef-4417-994c-6f4d63336ec5)

![Floss Str 3](https://github.com/CertainRisk/Silly-Putty-Challenge/assets/141761181/35bbb90c-52c9-4d7c-bb55-a642a6ab83a6)

![Floss Str 4](https://github.com/CertainRisk/Silly-Putty-Challenge/assets/141761181/a56a6332-3869-4a53-a645-47a534bf1106)

## Time Travel with PEView

To continue our quest, I turn to PEView, a tool that helps us examine the binary's inner workings. My target is the Time Date Stamp, hidden within the IMAGE_NT_HEADER > IMAGE_FILE_HEADER. Lo and behold, it reveals itself as "2021/07/10 Sat 09:51:55." This timestamp provides a clue about the binary's origins, or it would but I have no idea what.

Now, I venture into the IMAGE_SECTION_HEADER .text, where I'm on the hunt for the Virtual Size and the Size of Raw Data. Armed with my trusty programming calculator, I convert from HEX to DEC:

- Virtual Size (95F6D) HEX = 614,253 DEC
- Size of Raw Data (96000) HEX = 614,400 DEC

These values are remarkably close, indicating that our sample isn't a packed binary. It's as if the binary wants to make it easy for us to explore its secrets!

![Virtual Size vs Raw Size](https://github.com/CertainRisk/Silly-Putty-Challenge/assets/141761181/ed148eb8-c7e4-46fa-bba8-e06a7cb6a2a8)


## Exploring the IMPORT Address Table (IAT)

As I delve deeper, I explore the IMPORT Address Table or IAT. I make a list of what sticks out:

- ShellExecuteA in SHELL32.dll
- A whole lot of action happening in KERNEL32

![Kernel32 PEView](https://github.com/CertainRisk/Silly-Putty-Challenge/assets/141761181/aad373c5-d5d4-4736-bcb1-7523a5bf9b84)

But then, I stumble upon ADVAPI32.dll. This one's intriguing; it's filled with Registry-related values. ADVAPI32.dll is an advanced Windows32 base API DLL file that supports security and registry calls. It sounds like there's something sinister afoot, so I put it on my radar.

![PEView ADVAPI32](https://github.com/CertainRisk/Silly-Putty-Challenge/assets/141761181/1ff91481-8ae1-42ee-8f29-bf3f287fd94d)

This marks the end of my Basic Static Analysis portion of my first solo malware analysis adventure. Stay tuned for more exciting discoveries as we delve deeper into the enigma of this binary. The adventure has only just begun!
