# Pickle Rick CTF

## Introduction

- This is a Rick and Morty-themed webapp with certain vulnerabilities
- I need to exploit a web server and find three ingredients (flags) to help Rick turn back to human from a pickle.

![image.png](Pickle%20Rick%20CTF%2019310321b95d8098b656c191e4afc0ce/image.png)

## Prerequisites

- Knowledge
    - **Good grasp of Linux commands**
    - **Enumeration techniques such as brute-forcing, research, viewing source code, web server analysis, etc.**
    - **Understanding of reverse shells, when/where/how to use them and remote code execution with netcat**
    - **Use URLs to access directories and files**
- Tools
    - **netcat**
    - [**gobuster**](https://github.com/OJ/gobuster)
    - [**Wappalyzer**](https://www.wappalyzer.com/apps/)
    - [**Reverse Shell Cheat Sheet**](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)

##Write-Up

- Let’s start with a `gobuster` scan to brute-force the directories and use a `dirbuster` wordlist from SecLists.

![image.png](Pickle%20Rick%20CTF%2019310321b95d8098b656c191e4afc0ce/image%201.png)

- Seems like there’s only two directories
- Let’s check the source code of the webpage

![image.png](Pickle%20Rick%20CTF%2019310321b95d8098b656c191e4afc0ce/image%202.png)

- Looks like the dev has stupidly left their username in the source itself!!
- Let’s take note of this.
- There’s only two directories, there may be more files in the top directory.
- Let’s use Wappalyzer to analyse the framework of this web server

![image.png](Pickle%20Rick%20CTF%2019310321b95d8098b656c191e4afc0ce/image%203.png)

- Turns out that the web server is powered by Apache, which usually uses PHP and HTML as backend files.
- So let’s check for `.php` `.html` and `.txt` files in the top directory of this server using `gobuster`

![image.png](Pickle%20Rick%20CTF%2019310321b95d8098b656c191e4afc0ce/image%204.png)

- `robots.txt` and `clue.txt` seem interesting, let’s open them using the URL.
- For `robots.txt` I get this:

![wubba.png](Pickle%20Rick%20CTF%2019310321b95d8098b656c191e4afc0ce/wubba.png)

- For `clue.txt` I get this:

![wubba.png](Pickle%20Rick%20CTF%2019310321b95d8098b656c191e4afc0ce/wubba%201.png)

- According to `gobuster` we can also access the login page, so let’s do this

![image.png](Pickle%20Rick%20CTF%2019310321b95d8098b656c191e4afc0ce/image%205.png)

- We can probably use the username from the source code `R1ckRul3s` and text from `robots.txt` as credentials.

![image.png](Pickle%20Rick%20CTF%2019310321b95d8098b656c191e4afc0ce/image%206.png)

- And it works!
- Let’s enter some commands to ascertain contents of the directory

![image.png](Pickle%20Rick%20CTF%2019310321b95d8098b656c191e4afc0ce/image%207.png)

- `Sup3rS3cretPickl3Ingred.txt` seems useful, let’s read it

![image.png](Pickle%20Rick%20CTF%2019310321b95d8098b656c191e4afc0ce/image%208.png)

- Whoops! The command is disabled.
- After a quick google search, I found out that `nl` works like `cat` but also outputs line numbers too, so let’s try that

![wubba.png](Pickle%20Rick%20CTF%2019310321b95d8098b656c191e4afc0ce/wubba%202.png)

- It worked!
- It seems that to access other pages, I need a “Real Rick” authentication

![image.png](Pickle%20Rick%20CTF%2019310321b95d8098b656c191e4afc0ce/image%209.png)

- To bypass this we can use a reverse shell. A really useful cheat sheet can be found [here](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet).
- First we need to know which backend services the server is using, we can use the `which` command to find appropriate binaries.

![***Found bash***](Pickle%20Rick%20CTF%2019310321b95d8098b656c191e4afc0ce/image%2010.png)

***Found bash***

![***Found PHP***](Pickle%20Rick%20CTF%2019310321b95d8098b656c191e4afc0ce/image%2011.png)

***Found PHP***

![***Found PERL***](Pickle%20Rick%20CTF%2019310321b95d8098b656c191e4afc0ce/image%2012.png)

***Found PERL***

![***Found Python3***](Pickle%20Rick%20CTF%2019310321b95d8098b656c191e4afc0ce/image%2013.png)

***Found Python3***

- Now it’s a matter of trial and error and see which services we can use a reverse shell with.
- Bash didn’t seem to work
- PHP reverse shell actually worked!

![image.png](Pickle%20Rick%20CTF%2019310321b95d8098b656c191e4afc0ce/image%2014.png)

- Let’s check for files

![wubba.png](Pickle%20Rick%20CTF%2019310321b95d8098b656c191e4afc0ce/wubba%203.png)

- After snooping around, I finally found the second ingredient on the `/home/rick` directory
- Let’s look for the last one.
- It might be in `/root` as that’s another valid user to store files.

![image.png](Pickle%20Rick%20CTF%2019310321b95d8098b656c191e4afc0ce/image%2015.png)

- We can’t do this because we lack permissions, so let’s try privilege escalation
- A quick google search told me to check for `NOPASSWD` using `sudo -l`

![image.png](Pickle%20Rick%20CTF%2019310321b95d8098b656c191e4afc0ce/image%2016.png)

- And then as long as it is active and allows all paths to run as `sudo` we can use a PHP privilege escalation exploit which takes an environment variable which executes a binary of an allowed path and the variable is fed into an executable PHP code/command.

```bash
CMD="/bin/sh"
sudo php -r "system('$CMD');"
```

![image.png](Pickle%20Rick%20CTF%2019310321b95d8098b656c191e4afc0ce/image%2017.png)

- We can now enter `/root` and capture the final flag

![wubba.png](Pickle%20Rick%20CTF%2019310321b95d8098b656c191e4afc0ce/wubba%204.png)
