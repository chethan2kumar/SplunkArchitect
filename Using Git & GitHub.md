
# Using Git & GitHub

Notes on using Git
Just the essentials.

***
### GitHub tutorial

<a href="https://guides.github.com/activities/hello-world/" target="_blank">https://guides.github.com/activities/hello-world/</a>
***
## Github online

Go to GitHub.com:

www.github.com
or
https://github.com

Click Sign up
or
Click Sign in
Enter your Username or Email  	packetiq
& password		```<password>```

From the home page:

Click an existing repository, or

Click the green box:    + New repository
Add the repository name  
Click and edit the README file

Everything else is mostly click-able

## Using Git on a Windows PC

Install Git for Windows:
https://git-scm.com/download/win

Accept all the defaults; add the Git Bash icon to your desktop / taskbar

The Git Bash shortcut properties are:
Target:		"C:\Program Files\Git\git-bash.exe" --cd-to-home
Start in:	%HOMEDRIVE%%HOMEPATH%


Using Git:

Click to open the Git Bash shell
A MINGW64 terminal window will open

Run 'git help git' to dipslay the help index
Run 'git help <command>' to display help for specific commands.

James@DESKTOP-6N4UAR1 MINGW64 ~   (or similar)
$

pwd 			shows you the present working directory
cd /c/Dropbox/		change to C:\Dropbox directory
ls -al			list all files in the current directory

First time setup:

git config --global user.name packetiq
git config --global user.email jim.baxter@packetiq.com

## Using Git with GitHub

### Fetch a copy of an existing repository from GitHub.com to a local PC directory:

Change directory to where you'd like to create a copy of the repository on GitHub
(the git clone command will create a new directory from this location with the same name as on GitHub - so don't create the directory you anticipate the files to reside in)

```git clone https://github.com/packetiq/Appalyzer.git```  

(this is the repository structure)

ls -al  	// new repository directory should show up

cd Appalyzer		(the new repository directory)

ls -al	// should see .git & README.md

Use a text editor to add some text to README.md

Get the status:
git status		// indicates that README.md has been modified

Indicate that you want Git to track *all* the files in your local dir:
git add .

You can show the 'handles' for your 'remotes' - whatever that means - but I don't know why at this point:
git remote -v

Commit everyting that has been 'staged' and add a message:
git commit -m "commit latest files"

Push the latest file updates back to the GitHub repository:
git push origin master
username: packetiq
password: ```<password>```

###Authenticating to GitHub

You will have to add a ssh key to git hub - follow the instructions on the link below:
https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/


Generating a new SSH key and adding it to the ssh-agent

MAC WINDOWS LINUX
After you've checked for existing SSH keys, you can generate a new SSH key to use for authentication, then add it to the ssh-agent.

If you don't already have an SSH key, you must generate a new SSH key. If you're unsure whether you already have an SSH key, check for existing keys.

If you don't want to reenter your passphrase every time you use your SSH key, you can add your key to the SSH agent, which manages your SSH keys and remembers your passphrase.

Generating a new SSH key

Open Git Bash.

Paste the text below, substituting in your GitHub email address.

ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

This creates a new ssh key, using the provided email as a label.

Generating public/private rsa key pair.
When you're prompted to "Enter a file in which to save the key," press Enter. This accepts the default file location.

 Enter a file in which to save the key (/c/Users/you/.ssh/id_rsa):[Press enter]
At the prompt, type a secure passphrase. For more information, see "Working with SSH key passphrases".
Enter passphrase (empty for no passphrase): [Type a passphrase]
Enter same passphrase again: [Type passphrase again]
Adding your SSH key to the ssh-agent

Before adding a new SSH key to the ssh-agent to manage your keys, you should have checked for existing SSH keys and generated a new SSH key.

If you have GitHub Desktop installed, you can use it to clone repositories and not deal with SSH keys. It also comes with the Git Bash tool, which is the preferred way of running git commands on Windows.

Ensure the ssh-agent is running:

If you are using the Git Shell that's installed with GitHub Desktop, the ssh-agent should be running.
If you are using another terminal prompt, such as Git for Windows, you can use the "Auto-launching the ssh-agent" instructions in "Working with SSH key passphrases", or start it manually:

### start the ssh-agent in the background

eval $(ssh-agent -s)
Agent pid 59566
Add your SSH key to the ssh-agent. If you are using an existing SSH key rather than generating a new SSH key, you'll need to replace id_rsa in the command with the name of your existing private key file.

$ ssh-add ~/.ssh/id_rsa

Next Step: Add the SSH key to your GitHub account (below)



###Adding a new SSH key to your GitHub account

https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/


MAC WINDOWS LINUX

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
Settings icon in the user barIn the upper-right corner of any page, click your profile photo, then click Settings.

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