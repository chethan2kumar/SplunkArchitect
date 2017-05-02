
# Using Git & GitHub <a name="top"></a>

Notes on using Git and GitHub. Just the essentials.

1. [GitHub tutorial](#tutorial)
2. [GitHub online](#online)
3. [Using Git on a Windows PC](#git_on_windows)
	* [Using the Git shell](#git_shell)
4. [Using Git with GitHub](#git_w_github)
	* [Fork an existing repository](#fork)
	* [Push to the repository](#push)
5. [Configuring Git Authentication](#authentication)  

***
## GitHub tutorial <a name="tutorial"></a>

<a href="https://guides.github.com/activities/hello-world/" target="_blank">https://guides.github.com/activities/hello-world/</a>
***
## GitHub online <a name="online"></a>

<a href="https://github.disney.com" target="_blank">https://github.disney.com</a>  
or  
<a href="https://github.com/" target="_blank">https://github.com</a>  (generic)  

Click Sign up  or  Sign in
Enter your Username or Email & password

__From the home page:__

1. Click an existing repository, or
Click the green box:    ```+ New repository```  

2. Add the repository name  

3. Click and edit the README file

Other GitHub features are mostly click-able - navigate and explore as desired.  

[Top](#top)
***
## Using Git on a Windows PC <a name="git_on_windows"></a>

Install Git for Windows:
<a href="https://git-scm.com/download/win" target="_blank">https://git-scm.com</a>

Accept all the defaults; add the Git Bash icon to your desktop / taskbar

The Git Bash shortcut properties are:
```
Target:		"C:\Program Files\Git\git-bash.exe" --cd-to-home
Start in:	%HOMEDRIVE%%HOMEPATH%
```

### Using the Git shell <a name="git_shell"></a>

1. Click the icon to open the Git Bash shell
	A MINGW64 terminal window will open and display a prompt similar to:  
	```James@DESKTOP-6N4UAR1 MINGW64 ~/<path to your current directory>
	$
	```
	
__Common Git Commands__
The Git shell is like a ssh environment:  

```pwd```	- shows you the present working directory
	
```cd /c/Dropbox/``` - change to C:\Dropbox directory  
	
```ls -al``` - list all files in the current directory  
	
```git remote -v``` - show the 'handles' for your remote connections - v = verbose  
	
```git help git``` - dipslay the help index  
	
```git help <command>``` - display help for specific commands  

__First time setup__
The first time you use the git command shell, run these commands to save these to the environment so you don't have to do it again:  
```
git config --global user.name <your username>
git config --global user.email <your email>
```
[Top](#top)
***
## Using Git with GitHub <a name="git_w_github"></a>

__Note:__ If this is your first time using Git, you may need to set up authentication:
[Configuring Git Authentication](#authentication)

### Fork an existing repository <a name="fork"></a>

This is the process of 'forking' (getting a copy of) an existing repository from GitHub.com to a local PC directory. After you have a local copy on your PC, you can edit/add/delete files and then 'push' them back to the repository on GitHub.  

1. Start the Git Bash shell  

2. Create and/or change directory to where you'd like to create a copy of the repository on GitHub
__Note!__ The git clone command will create a new directory *inside this folder* with the same name as the repository on GitHub - so don't create and CD to the directory you anticipate the files to reside in.

3. Open a new browser window pointed to the GitHub repository:  

	A generic GitHub login:  
	<a href="https://github.com" target="_blank">https://github.com</a>

	or the Disney Enterprise Monitoring repository:  
	<a href="https://github.disney.com/WDPR-EnterpriseMonitoring" target="_blank">https://github.disney.com/WDPR-EnterpriseMonitoring</a>

	or if you want to use a personal GitHub account (substitute your LANID):  
	<a href="https://github.disney.com/BAXTJ018/" target="_blank">https://github.disney.com/BAXTJ018/</a>
	
4. Click to navigate to the repository you want to 'fork' (download a copy of)

5. Click the Green 'Clone or download' button and copy the URL to your clipboard - for example: 

		```git clone https://github.disney.com/WDPR-EnterpriseMonitoring/SplunkOnAWS.git```  
	(this is the repository structure on GitHub)

6. Paste this entire ```git clone https://...``` command into the Git shell on your PC.

	If you have the proper authentication set up, you should see messages that indicate a copy of the repository is being created on your PC.

7. ```ls -al``` - the new repository directory should show up

8. Change directory to the new repository directory ```cd /<new folder>```

9. ```ls -al``` - you should see .git and a README.md and other repository files

### Push to the repository <a name="push"></a>

After you have edited / added / deleted files in your local PC directory, you can update the files in the GitHub repository.

1. Get the local Git status:  
```git status``` - creates a list of files that have been modified/added/deleted

2. Indicate that you want Git to track *all* the files in your local directory:  
```git add .```
	You could also indicate specific files only: ```git add <filename>```

3. Commit everything that has been 'staged' and add a message (mandatory):  
```git commit -m "latest updates"```

4. Push the file updates to the GitHub repository:  
```git push origin master```

You may be prompted for your username and password

[Top](#top)
***

## Configuring Git Authentication <a name="authentication"></a>

You will have to add a ssh key to git hub - follow the instructions on the link below:
https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/


Generating a new SSH key and adding it to the ssh-agent

__MAC WINDOWS LINUX__

After you've checked for existing SSH keys, you can generate a new SSH key to use for authentication, then add it to the ssh-agent.

If you don't already have an SSH key, you must generate a new SSH key. If you're unsure whether you already have an SSH key, check for existing keys.

If you don't want to reenter your passphrase every time you use your SSH key, you can add your key to the SSH agent, which manages your SSH keys and remembers your passphrase.

__Generating a new SSH key__

Open Git Bash.

Paste the text below, substituting in your GitHub email address.

ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

This creates a new ssh key, using the provided email as a label.

__Generating public/private rsa key pair__

When you're prompted to "Enter a file in which to save the key," press Enter. This accepts the default file location.

 Enter a file in which to save the key (/c/Users/you/.ssh/id_rsa):[Press enter]  
 
At the prompt, type a secure passphrase. For more information, see "Working with SSH key passphrases". 

Enter passphrase (empty for no passphrase): [Type a passphrase]
Enter same passphrase again: [Type passphrase again]

__Adding your SSH key to the ssh-agent__

Before adding a new SSH key to the ssh-agent to manage your keys, you should have checked for existing SSH keys and generated a new SSH key.

If you have GitHub Desktop installed, you can use it to clone repositories and not deal with SSH keys. It also comes with the Git Bash tool, which is the preferred way of running git commands on Windows.

Ensure the ssh-agent is running:

If you are using the Git Shell that's installed with GitHub Desktop, the ssh-agent should be running.
If you are using another terminal prompt, such as Git for Windows, you can use the "Auto-launching the ssh-agent" instructions in "Working with SSH key passphrases", or start it manually:

__start the ssh-agent in the background__

```eval $(ssh-agent -s)```
Agent pid 59566

Add your SSH key to the ssh-agent. If you are using an existing SSH key rather than generating a new SSH key, you'll need to replace id_rsa in the command with the name of your existing private key file.

```$ ssh-add ~/.ssh/id_rsa```

Next Step: Add the SSH key to your GitHub account

###Adding a new SSH key to your GitHub account

https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/

__MAC WINDOWS LINUX__

To configure your GitHub account to use your new (or existing) SSH key, you'll also need to add it to your GitHub account.

Before adding a new SSH key to your GitHub account, you should have:

Checked for existing SSH keys
Generated a new SSH key and added it to the ssh-agent
Note: DSA keys were deprecated in OpenSSH 7.0. If your operating system uses OpenSSH, you'll need to use an alternate type of key when setting up SSH, such as an RSA key. For instance, if your operating system is MacOS Sierra, you can set up SSH using an RSA key.

Copy the SSH key to your clipboard.

If your SSH key file has a different name than the example code, modify the filename to match your current setup. When copying your key, don't add any newlines or whitespace.

```$ clip < ~/.ssh/id_rsa.pub```

Copies the contents of the id_rsa.pub file to your clipboard  

Tip: If clip isn't working, you can locate the hidden .ssh folder, open the file in your favorite text editor, and copy it to your clipboard.  

Settings icon in the user bar in the upper-right corner of any page, click your profile photo, then click Settings.

Authentication keysIn the user settings sidebar, click SSH and GPG keys.

SSH Key buttonClick New SSH key or Add SSH key.

In the "Title" field, add a descriptive label for the new key. For example, if you're using a personal Mac, you might call this key "Personal MacBook Air".
The key fieldPaste your key into the "Key" field.
The Add key buttonClick Add SSH key.

Sudo mode dialog  

If prompted, confirm your GitHub password.

$ git clone git@github.disney.com:BAXTJ018/SplunkOnAWS.git

is not the same repository as

$ git clone git@github.disney.com:WDPR-EnterpriseMonitoring/SplunkOnAWS.git


*******************************
Getting it to work
*******************************

1. Start Git Bash
2. cd to directory containing slave files
3. git status
4. Edit / Add / Delete files

> Written with [StackEdit](https://stackedit.io/).