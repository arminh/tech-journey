Hey everyone to my... not sure what this is going to be. Let's call it a blog for now. So how did I get here? Well, I have been working as a software developer for about 18 years now. Starting with internships in 2006, working about a day a week and part time during vacation alongside my studies 2010-2018 and then starting my first full-time job in 2019. 

A week ago I was invited to a job interview as a Senior React developer leading the development of large, really cool greenfield project. Long story short, I wasn't confident that I was up for the task. I was in the same situation 5 years ago. I felt crushed after an interview at a large company were they decided my Angular skills weren't enough for the position. Last week I was in the same position. I've learned so much in the last 5 years and still I am not confident enough to stand there and say "This is what i know, this is my experience. Does that fit what you are searching for?" Instead, I collapsed into myself and wasn't sure if I know single thing about web development anymore. I still feel a bit down, but i don't want the same thing to happen again in the next interview. 

That's why I decided to start this project. I want to take a structured approach of learning new and re-visiting old skills in web development. I want to start from a simple HTML page managed on Github and hosted on GCP and go from there. And I want to document my learnings here and see where it leads me. So welcome to the ride (No I am asking myself who I am talking to?^^)

# Setting up a Github repository
I decided to start by setting up a repository on my [Github account](https://github.com/arminh). With a little help from ChatGPT for naming the repository the [tech-journey](https://github.com/arminh/tech-journey) project is born. Next thing is to set up SSH to connect to Github and clone my repository.

## Creating an ssh key-pair:
```
$ ssh-keygen
Generating public/private rsa key pair.
-Enter file in which to save the key (/home/ahutzler/.ssh/id_rsa): /home/ahutzler/.ssh/id_rsa_github
Enter passphrase (empty for no passphrase):
Enter same passphrase again: 
Your identification has been saved in /home/ahutzler/.ssh/id_rsa_github
Your public key has been saved in /home/ahutzler/.ssh/id_rsa_github.pub
The key fingerprint is:
SHA256:e747btos7YiyCHmO7czDIj8ufy5IDRXwntw/bCvTKCM ahutzler@ahutzler-Precision-5530
The key's randomart image is:
+---[RSA 3072]----+
| ....            |
|  ..             |
|  ..             |
| .o o            |
|  o+ .  S        |
| o .  o  .       |
|+o.   o=...      |
|=E=+o+.o+*+      |
|oB%*=o+.o*O+     |
+----[SHA256]-----+

```

This already lead to some questions. What are the **key fingerprint** and the **key's randomart image**?

### key fingerprint ###
The key fingerprint `SHA256:e747btos7YiyCHmO7czDIj8ufy5IDRXwntw/bCvTKCM ahutzler@ahutzler-Precision-5530` consists of three parts
* Hashing alorightm used: `SHA256`
* Encoded public key: `e747btos7YiyCHmO7czDIj8ufy5IDRXwntw/bCvTKCM`
* Host name: `ahutzler@ahutzler-Precision-5530`

This [SSH documentation](https://docs.ssh-mitm.at/user_guide/fingerprint.html) offers a good description:
> The fingerprint ensures that you do not connect to a wrong server. One of the most common reasons for unknown fingerprints is the reinstallation of a system where new keys are generated.
> However, it can also be a Man in the Middle attack, where the connection was redirected to another server.
> For this reason, the fingerprint must always be compared against a trusted source.

### key's randomart image ###


## Establish an SSH connection with Github
Let's add the public key of our newly created key-pair to my Github account:
1. Navigate to https://github.com/settings/keys
2. Add the publick key using the "New SSH key" dialog

It should look something like this:
![Github SSH Key](./github-ssh-key.png)


Now let's use the example in the [SSH documentation](https://docs.ssh-mitm.at/user_guide/fingerprint.html) to connect to github:
```
$ ssh github.com
The authenticity of host 'github.com (140.82.121.4)' can't be established.
ED25519 key fingerprint is SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

Before confirming with yes, let's make sure the fingerprint is valid by checking the [Github's SSH key fingerprints](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints) or executing
```
$ ssh-keyscan github.com  >github.ssh-keyscan
$ cat github.ssh-keyscan
```

And indeed the ED25519 fingerprints match. So it is safe to connect. Let's try again:
```
$ ssh github.com
The authenticity of host 'github.com (140.82.121.3)' can't be established.
ED25519 key fingerprint is SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'github.com' (ED25519) to the list of known hosts.
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0644 for '/home/ahutzler/.ssh/rlt-deployment' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "/home/ahutzler/.ssh/rlt-deployment": bad permissions
```

Okay this is harder than expected. I don't remember using this SSH key, so let's delete it and try again:
```
$ ssh github.com
no such identity: /home/ahutzler/.ssh/rlt-deployment: No such file or directory
ahutzler@github.com: Permission denied (publickey).
```

Okay it still trys to use the deleted key. Let's check the ssg config. And yep, there it is:

```
Host *
    AddKeysToAgent yes
    IdentityFile ~/.ssh/id_rsa
    IdentityFile ~/.ssh/rlt-deployment
```

Let's remove the rlt-deployment entry and add `id_rsa_github` to the list. And... try again:
```
$ ssh github.com
ahutzler@github.com: Permission denied (publickey).
```

Oh man... still not quite there yet. Github docs to the rescue: [Always use the "git" user](https://docs.github.com/en/authentication/troubleshooting-ssh/error-permission-denied-publickey#always-use-the-git-user)

```
$ ssh git@github.com
PTY allocation request failed on channel 0
Hi arminh! You've successfully authenticated, but GitHub does not provide shell access.
Connection to github.com closed.
```

Success!!! Oh man, that took way longer than expected. 

## Clone the Github repository and add the new content

Let's try to clone the project repository using ssh:
```
$ git clone git@github.com:arminh/tech-journey.git
Cloning into 'tech-journey'...
remote: Enumerating objects: 4, done.
remote: Counting objects: 100% (4/4), done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 4 (delta 0), reused 0 (delta 0), pack-reused 0
Receiving objects: 100% (4/4), done.
```

Nice! That did it. Let's add this markdown file to the repository and create a commit. After looking up how to move multiple files with the `mv` command:

```
$ mv -t tech-journey tech-journey.md github-ssh-key.png
```

Before adding the files I want to get one more thing done. I want to track my progress using Stories. Let's start easy and use the [Github issue tracking](https://github.com/arminh/tech-journey/issues) and create the first issue [Create a Github repository](https://github.com/arminh/tech-journey/issues/1). Github offers the option to open a feature branch for the issue right away. So let's do that.

Back in the shell, let's check out the new feature branch:
```
$ git fetch origin
From github.com:arminh/tech-journey
 * [new branch]      1-create-github-repository -> origin/1-create-github-repository

$ git checkout 1-create-github-repository 
A	github-ssh-key.png
A	tech-journey.md
Branch '1-create-github-repository' set up to track remote branch '1-create-github-repository' from 'origin'.
Switched to a new branch '1-create-github-repository'
```

Let's add the new files
```
$ git add .
git status
On branch 1-create-github-repository
Your branch is up to date with 'origin/1-create-github-repository'.

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	new file:   github-ssh-key.png
	new file:   tech-journey.md
	
$ git commit -m "1: Add Markdown file to repository"
[1-create-github-repository 5a9880e] 1: Add Markdown file to repository
 2 files changed, 131 insertions(+)
 create mode 100644 github-ssh-key.png
 create mode 100644 tech-journey.md

$ git push
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 20 threads
Compressing objects: 100% (4/4), done.
Writing objects: 100% (4/4), 41.52 KiB | 10.38 MiB/s, done.
Total 4 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com:arminh/tech-journey.git
   61fbf7a..5a9880e  1-create-github-repository -> 1-create-github-repository
```

And done! I will create a Pull request in Github and merge it. I think that's enough for today. 
I feel like I got a lot done and also nothing :D. Let's continue next time.
