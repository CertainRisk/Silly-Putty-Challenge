# My First Solo Malware Analysis - Silly-Putty-Challenge

So, here I am, embarking on my first solo malware analysis adventure. Armed with the mighty Cmder, my trusty companion in the world of cybersecurity, I'm on a quest to unravel the secrets hidden within a mysterious binary. But before we dive into the juicy details, let's start with the basics.

## The Hunt for Hashes

The first step in this thrilling journey is to uncover the SHA256 and MD5 hash of our enigmatic binary. These cryptographic signatures are like fingerprints, unique to each file. So, I unleash Cmder to find them:

SHA256: 0c82e654c09c8fd9fdf4899718efa37670974c9eec5a8fc18a167f93cea6ee83
MD5: 334a10500feb0f3444bf2e86ab2e76da

A quick check on [VirusTotal](https://www.virustotal.com/) reveals that our binary is a well-known Trojan Win32.

![VirusTotal](https://github.com/CertainRisk/Silly-Putty-Challenge/assets/141761181/4a9f751d-26f7-4293-b42d-9643239a6fb8)


## String Extravaganza with FLOSS

Next, we put our binary under the spotlight with FLOSS, a tool for extracting strings. There are strings galore. It's a lot but I remember what to look for from previous videos in the course so I will do my best!

Amidst the string salad, I spot numerous references to SSH. And a suspicious URL I shall not link here and cryptic `%s==NULL in terminal.c` catch my eye. 

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
_______________________________________________________________________________________________________________________________________________

# Basic Dynamic Analysis
- This section had a lot of new lessons that I did not do immediately on my own, some things were familiar but most of this I learned during the challenge explanation as I didn't reach as in depth conclusions initially.

## Initial Detonation

- Upon detonation I noted a blue command window pop up for a second and disappear. 
- A PuTTY Configuration box also popped up which I later learned is the legitimate part of the program.

![Putty Program Window](https://github.com/CertainRisk/Silly-Putty-Challenge/assets/141761181/c4e52e32-786f-473d-bd1a-1d89c4800df8)


## Main Payload Identification

The question posed for this part of the challenge was "So what is the main payload initiated at detonation and what tools can you use to identify this?" 

- In Procmon, I set a filter for "Process Name" containing "putty.exe".
- Lo and behold, my putty shows up in Procmon with a PID of 3356.
- Now, I set up another filter for "Parent PID" as "3356." It's important to filter off the process name since Putty won't be its own parent process.\
  
![Putty Procmon PID](https://github.com/CertainRisk/Silly-Putty-Challenge/assets/141761181/d5cb0df3-9cb7-4233-af5a-e166f07e5a1f)

And there it is! Filtering by the PID, we spot a brief appearance of a powershell.exe window. But what's it up to? The whole VM crashed, although it might be more of a VM issue than the malware's doing. The new PID is 9744.

![Huge 1 Line Powershell command](https://github.com/CertainRisk/Silly-Putty-Challenge/assets/141761181/99a402e9-922b-46cf-ab81-02c82f92d25d)


Here I learned that the powershell window opens a hidden window and a non-interactive one. It's also bypassing the execution policy and creating a new object to handle a GzipStream.

## Converted Base64String

Putting the string in the following format on Remnux allows us to conver the format and place it into a file. Currently we don't know what type of file it is so it's left blank. 
![Remnux Echo](https://github.com/CertainRisk/Silly-Putty-Challenge/assets/141761181/7e580467-24d5-4899-8075-7c69212fc223)

We figure out the file type by the following command. 
![Remnux File Out](https://github.com/CertainRisk/Silly-Putty-Challenge/assets/141761181/d720fb6a-9f8d-4cf5-a45a-9a2779ab63f3)

- Once we do that we go to where the file is located, extract it and open it in a text file.

## Deciphering the Payload

It appears to be a powershell one-liner running a base64-encoded and compressed payload.

![Screenshot of Payload](https://github.com/CertainRisk/Silly-Putty-Challenge/assets/141761181/9e6b2ceb-469a-43da-b4f6-3eeaf1703133)

## DNS Query

Now, let's unveil the DNS record queried at detonation. To do this, I launch Wireshark to capture Ethernet traffic on the Flare VM. After exiting all Putty sessions and executing putty.exe again, it's clear that the binary is setting up some remote connection.

![Wireshark Capture](https://github.com/CertainRisk/Silly-Putty-Challenge/assets/141761181/fb594c7e-6939-4903-b07e-377d65a413af)

![TCP View 8443](https://github.com/CertainRisk/Silly-Putty-Challenge/assets/141761181/c107fb34-81e0-4655-a3d1-b83e61150411)


In an attempt to trick the malware, I modify the hosts file so it believes we are the website it's trying to reach.

![Local Hosts Trickery](https://github.com/CertainRisk/Silly-Putty-Challenge/assets/141761181/95d2b847-77b1-488e-ab40-313a6108ef34)


However, things take an unexpected turn. Unlike the previous scenario, this time the malware's response is a confusing mess.

![Ncat Output](https://github.com/CertainRisk/Silly-Putty-Challenge/assets/141761181/dea4cdef-ee5d-4386-aaa6-9f880b6cddd1)


It turns out this specific sample requires a TLS certificate for communication.

In summary, our journey into basic dynamic analysis has unveiled intriguing insights into the initial detonation, payload identification, and an attempt to decipher the mysterious DNS queries. 
