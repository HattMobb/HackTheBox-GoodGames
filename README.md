# HackTheBox-GoodGames
A walkthrough/ write-up of the "GoodGames" box following the CREST pentesting pathway, featuring SQL injection, SSTI attacks and privesc via poor permissions management. (work in progress)

## Enumeration

Web server:

![image](https://github.com/HattMobb/HackTheBox-GoodGames/assets/134090089/e001736e-c883-4b5e-a452-886967ec4ed5)


Hosts some type of game store/blog. A couple of opportunities for user input:

![image](https://github.com/HattMobb/HackTheBox-GoodGames/assets/134090089/410335cc-bc50-4ea4-b2ca-30ed6e2ea4e2)

Checking the login page and feeding the request to SQLmap:

![image](https://github.com/HattMobb/HackTheBox-GoodGames/assets/134090089/65260cf8-57a5-4147-9698-e1d7b8fa9086)


Login form is vulnerable:
![image](https://github.com/HattMobb/HackTheBox-GoodGames/assets/134090089/cd101053-b155-4d4b-bc36-fe01564d90e7)

Further enumeration of the database with the `--banner --current-user --current-db --is-dba` flags:

![image](https://github.com/HattMobb/HackTheBox-GoodGames/assets/134090089/e89eb693-0cbb-438a-bf56-d7bd8e5ec0b9)


and 3 tables :

![image](https://github.com/HattMobb/HackTheBox-GoodGames/assets/134090089/a962fc61-7aad-496a-ab96-92700e53f2e0)

I dumped the credentials within the table via `sqlmap -r  request.txt --dump -T user -D main` :

![image](https://github.com/HattMobb/HackTheBox-GoodGames/assets/134090089/2ed2ea05-4fe5-4782-ae5b-bb87c9b502b7)

Cracking the hash allows to log in to the site as admin:

![image](https://github.com/HattMobb/HackTheBox-GoodGames/assets/134090089/85f9caf8-2a2e-4a72-a06f-d80009aa69c1)

After poking around I found a Flask admin log in page:

![image](https://github.com/HattMobb/HackTheBox-GoodGames/assets/134090089/8e2dfad4-e952-4559-b921-6e96a37b46a1)

Luckily, due to credential re-use, gaining access was trivial and I was greeted with the internal Dashboard:

![image](https://github.com/HattMobb/HackTheBox-GoodGames/assets/134090089/34a65c2e-e765-4247-a8a1-39d139914846)

After playing with the settings, I came across a Server Side Template Injection vulnerability in the "username" input. After entering `{{2+2}}` the username changed to `4`:
![image](https://github.com/HattMobb/HackTheBox-GoodGames/assets/134090089/1062a1a2-db1a-4812-85f4-3be1e5ccf104)

Knowing this, the template engine being used was Jinja2 (as the site is using the Flask framework).
Just to confirm, I ran the `id` command with a successful response:

![image](https://github.com/HattMobb/HackTheBox-GoodGames/assets/134090089/456c39e8-8519-4023-8075-83fc85565c2e)


Backend shell:

![image](https://github.com/HattMobb/HackTheBox-GoodGames/assets/134090089/f5b8ed51-656f-47a8-b4fe-1ebc8e11cc49)


Judging by contents of the folder, Docker contatiners being used:

![image](https://github.com/HattMobb/HackTheBox-GoodGames/assets/134090089/2ad0214c-631a-4026-9033-39f1ab58ad8f)

Ping sweep of local network:

![image](https://github.com/HattMobb/HackTheBox-GoodGames/assets/134090089/1af025ea-9ea8-4c47-9a35-cea8ba857069)

User enumeration:

![image](https://github.com/HattMobb/HackTheBox-GoodGames/assets/134090089/12f66698-847e-4d4f-837b-9ed497a11871)

Password reuse allows SSH log in:

![image](https://github.com/HattMobb/HackTheBox-GoodGames/assets/134090089/58d8c7fe-57dc-4cd9-aac9-fe7c762d1cb8)

Copying bash binary:

![image](https://github.com/HattMobb/HackTheBox-GoodGames/assets/134090089/d5252d2a-c750-4c53-9f3e-82ac3b1d687e)


Change bash permissions:

![image](https://github.com/HattMobb/HackTheBox-GoodGames/assets/134090089/b8d24e4e-f086-45ab-b0a8-b26bb2d9758e)


Final root:

![image](https://github.com/HattMobb/HackTheBox-GoodGames/assets/134090089/74899a20-cb42-4307-b3c2-42d623f042c2)








