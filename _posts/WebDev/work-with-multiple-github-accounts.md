---
layout: default
title: Work with Multiple GitHub Accounts
parent: WebDev
nav_order: 2
---

<style>
  .container{
    display: flex;
    flex-wrap: wrap;
    justify-content: space-around;
  }
</style>

# Work with Multiple GitHub Accounts on a Single Machine

---

_reference : [How to work with multiple Github accounts on a single machine](https://gist.github.com/rahularity/86da20fe3858e6b311de068201d279e3)_

<br>

Let's suppose you need to use two github accounts, https://github.com/aopersonal and https://github.com/aowork on a single machine.

<br>

This setup can be done in 5 easy steps:

<br>

> ## 1. Create SSH keys for all accounts

First make sure our current directory is your .ssh folder.

```
cd ~/.ssh
```

Syntax for generating unique ssh key for ann account is:

```
 ssh-keygen -t rsa -C "your-email-address" -f "github-username"
```

**Now generating SSH keys for my two accounts**

```
ssh-keygen -t rsa -C "my_personal_email@gmail.com" -f "github-aopersonal"
ssh-keygen -t rsa -C "my_office_email@gmail.com" -f "github-aowork"
```

After entering the command the terminal will ask for a passphrase, but you can leave it empty and proceed.

<div class="container">
<img src="https://raw.githubusercontent.com/rahularity/github-essentials/bc3dafc37b36f5fb4ebcffcba63a7689a36fc7ff/screenshots/passphrase.png">
</div>

- Now after adding keys, in your .ssh folder, a public key and a private will get generated.
- The public key will have an extention **.pub** and private key will be there without any extention both having same name which you have passed after -f option in the above command. (in my case _github-aopersonal_ and _github-aowork_)

<div class="container">
<img src="https://raw.githubusercontent.com/rahularity/github-essentials/master/screenshots/ssh_keys_added.png">
</div>

> ## 2. Add SSH keys to SSH Agent

Now we have the keys but it cannot be used until we add them to the SSH Agent.

```
 ssh-add -K ~/.ssh/github-aopersonal
 ssh-add -K ~/.ssh/github-aowork
```

<br>

> ## 3. Add SSH public key to Github

For the next step we need to add our public key (that we have generated in our previous step) and add it to corresponding github accounts.

<br>

### 1) Copy the public key

We can copy the public key either by opening the github-rahul-office.pub file in vim and then copying the content of it.

```
vim ~/.ssh/github-aopersonal.pub
vim ~/.ssh/github-aowork.pub
```

If you don't have vim on your machine, we can directly copy the content of the public key file in the clipboard.

```
pbcopy < ~/.ssh/github-aopersonal.pub
pbcopy < ~/.ssh/github-aowork.pub
```

### 2) Paste the public key to Github

- Sign in to Github Account
- Go to Settings > SSH and GPG keys > New SSH Key
- Paste your copied public key and give it a title of your choice.

<br>

> ## 4. Create a Config file and make Host Entries

The ~/.ssh/config file allows us specify many config options for SSH. If the config file does not exist then create one (make sure you are in ~/.ssh directory)  
 <br>
Now we need to add these lines to the file, each block corresponding to each account we created earlier.

```
#aopersonal account
Host github.com-aopersonal
      HostName github.com
      User git
      IdentityFile ~/.ssh/github-aopersonal

#aowork account
Host github.com-aowork
      HostName github.com
      User git
      IdentityFile ~/.ssh/github-aowork
```

<br>

> ## 5. Cloning GitHub respositories using different accounts.

So we are done with our setups and now its time to see it in action. We will clone a repository using one of the account we have added.

```
git clone git@github.com-{your-username}:{owner-user-name}/{the-repo-name}.git

[e.g.] git clone git@github.com-aopersonal:aopersonal/TestRepo.git
```
