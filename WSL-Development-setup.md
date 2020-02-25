# WSL for Windows & CLI Tools
This is a guide for Windows 10 (Build 16215 or higher) using Hyper Terminal application for a command line environment.

### 1. Install WSL (Windows Subsystem for Linux)
Go to `Settings -> Update and Security -> For developers` and change `Sideload apps` setting to `Developer mode`.

Run `OptionalFeatures.exe` and enable `Windows Subsystem for Linux` then reboot your PC.

Open `Microsoft Store` application and search for `WSL`. Install Ubuntu 18.04 (or other - you can install mutiple, and
choose which one is active in config). Launch Ubuntu after it is installed from MS Store to finish setup and
configuration. Reboot your PC.

### 2. Install Hyper Terminal

Head over to [HT official](https://hyper.is/) to download and install. It's a good idea to pin this application to the
Windows Task Bar for easy access. Another tip is that you can run multiple tabs `(CTRL-SHIFT-T)`. I'd also suggest to
enable copy/paste via right-click by setting `quickEdit: true`.

### Install ZSH
    
Run HT and enter the following commands to install zsh:
```shell
$ bash
$ sudo apt-get install curl
$ sudo apt-add-repository ppa:git-core/ppa
$ sudo apt-get update
$ sudo apt-get install git
$ sudo apt-get install zsh
```

### 4. Default ZSH for Hyper Terminal
Open the HT menu and go to `Edit > Preferences (CTRL-Comma)` and make the following changes to default HT to open with
WSL bash shell:
```ini
shell: '',
shellArgs: ['--login'],
```
to
```ini
shell: 'C:\\Windows\\System32\\cmd.exe',
shellArgs: ['--login', '-i', '/c wsl'],
```
Quit HT and restart HT to confirm (shell prompt should have some color now!).

To change the default shell from bash to zsh, we need to run:
```shell
$ chsh -s /usr/bin/zsh
```
Quit HT and restart it, then check the default shell with `env | grep SHELL`.

### 5. Configuring ZSH using Oh-My-Zsh
Run the following commands to install a customized Z Shell environment:
```shell
$ curl -L https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh | bash
```
Let's add some plugins to help make life easier in zsh. A list of default plugins included with oh-my-zsh can be found
in [~/.oh-my-zsh/plugins](https://github.com/robbyrussell/oh-my-zsh/tree/master/plugins). Here are some suggested
plugins to use:

<table>
<tr>
<td><a href="https://github.com/robbyrussell/oh-my-zsh/tree/master/plugins/aws">aws</a></td>
<td>auto complete for awscli</td>
</tr>
<tr>
<td><a href="https://github.com/robbyrussell/oh-my-zsh/tree/master/plugins/docker">docker</a></td>
<td>auto complete for docker commands</td>
</tr>
<tr>
<td><a href="https://github.com/robbyrussell/oh-my-zsh/tree/master/plugins/docker-compose">docker-compose</a></td>
<td>aliases for docker-compose commands, e.g. dco, dcb, dcup, dcupd</td>
</tr>
<tr>
<td><a href="https://github.com/robbyrussell/oh-my-zsh/tree/master/plugins/gitignore">gitignore</a>
</td>output gitignore templates from <a href="https://www.gitignore.io/">gitignore.io</a><td>
</td>
</tr>
<tr>
<td><a href="https://github.com/robbyrussell/oh-my-zsh/tree/master/plugins/ssh-agent">ssh-agent</a></td>
<td>auto start ssh-agent - check <a href="https://github.com/robbyrussell/oh-my-zsh/tree/master/plugins/ssh-agent">github page</a> for configuration</td>
</tr>
</table>

To enable any of these, edit the file `~/.zshrc` and amend plugins line as appropriate:
```ini
plugins=(git aws docker docker-compose gitignore ssh-agent)
```

### 6. Notes on Developing with WSL
WSL has it's own filesystem so your typical Windows root folders won't be in the root inside a WSL shell. For example,
the default login directory for WSL is your Windows home directory, but this is mapped to: `/mnt/c/Users/<username>`.
If you are familiar with UNIX/Linux systems, you should know that a user's home directory is `~` and on WSL filesystem,
this is: `/home/<username>`.

WSL root filesystem should be located at
`%LOCALAPPDATA%\Packages\CanonicalGroupLimited.Ubuntu18.04onWindows_79rhkp1fndgsc\LocalState\rootfs` where the
part after `Packages` refers to the WSL package you have installed.

On a similar note; if you have Git for Windows installed, then the root filesystem is the directory that you chose to
install it in.

### 7. Installing docker for CLI
If you've not installed [Docker for Windows](https://hub.docker.com/editions/community/docker-ce-desktop-windows) yet,
do that first and then reboot your PC.

Go to the Taskbar tray App for Docker and open Settings. Enable the checkbox for
**Expose daemon on tcp://localhost:2375 without TLS** from the General tab.

Run the following commands:
```shell
$ sudo apt-get install -y apt-transport-https ca-certificates software-properties-common
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo apt-key fingerprint 0EBFCD88
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
$ sudo apt-get update -y
$ sudo apt-get install -y docker-ce
$ sudo usermod -aG docker $USER
$ sudo apt-get install -y python python-pip
$ sudo pip install docker-compose
```
For docker CLI to be able to find the windows docker daemon, add the following to `~/.zshrc`:
```shell
export DOCKER_HOST=tcp://localhost:2375
```
One final configuration we need to set up is because the Windows docker daemon expects a Windows path. WSL automatically
does the path conversions for us, but the problem when using docker is the location of the `C: drive`. As mentioned
earlier, this is mounted by default as `/mnt/c`. When we run docker commands under WSL, it will look for the daemon
running from path `/c/` so to fix this problem we change the location of the root mount point for the internal HDD.

Create/modify the file `/etc/wsl.conf` and then reboot your PC:
```ini
[automount]
root = /
options = "metadata"
```

### 8. Project Source Code

It would be recommended to create your workspace project folder within the Windows filesystem, e.g. `c:\workspace` or
`c:\Users\<username>\Documents\workspace` so that you can access these files outside of the WSL environment and use it
with other third-party software.

For use in our WSL environment, simply make a symbolic link wherever is convenient for you, e.g.
```shell
$ sudo ln -s /c/workspace /workspace
```

### 9. CLI git and GitHub
Before you start using git, you should configure it before making any commits. Recommended set up:
```shell
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com
$ git config --global core.editor nano
$ git config --global core.autocrlf input
$ git config --global core.filemode false
$ git config --global core.excludesfile ~/.gitignore_global
$ git config --global commit.template ~/.gitmessage.txt
```
If you had enable gitignore plugin for oh-my-zsh, you can generate the ignore file, e.g.
```shell
$ gi windows,composer,visualstudiocode > ~/.gitignore_global
```
Whatever convention/style you choose to make commits for the project; you should define the template in the
`~/.gitmessage.txt` file.

Create your SSH key with the command: 
```shell
$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```
Copy the contents of the public key (default file is `id_rsa.pub`), then go to your Profile settings on GitHub. In
`SSH and GPG keys`, add a `New SSH Key` and give a meaningful Title (e.g. _WSL HyperT_). Paste the contents of the
public key, but beware of any line-breaks that may have been inserted by the application you used to view the public
key. You can test it with the following command:
```shell
$ ssh -T git@github.com
```
You may want to copy the WSL `~/.ssh` directory to your windows home directory to allow other 3rd party apps easy access
to the same ssh keys.

### 10. PHP7
```shell
$ sudo apt-get install php php-xml php-mbstring zip unzip php-curl
```

### 11. composer PHP Package Manager
```shell
$ php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
$ php -r "if (hash_file('sha384', 'composer-setup.php') === 'a5c698ffe4b8e849a443b120cd5ba38043260d5c4023dbf93e1558871f1f07f58274fc6f4c93bcfd858c6bd0775cd8d1') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
$ php composer-setup.php
$ php -r "unlink('composer-setup.php');"
```
After installing, it is recommended that you move the composer file to a general bin folder, e.g.
```shell
$ mv composer.phar /usr/local/bin/composer
```

### 12. NPM JavasScript Package Manager
```shell
$ sudo apt-get install npm
```

### 13. AWS CLI
Ubuntu comes pre-installed with `python` (v2) and `python3`. We will be using python3 to install AWS CLI. You can check
the version with `python3 -V`. First lets update our packages so make sure we're up to date on current versions.
```shell
$ sudo apt-get update
$ sudo apt-get -y upgrade
```
Next we need to install `pip` (Python Package Index) package manager.
```shell
$ sudo apt-get install python3-distutils
$ curl -O https://bootstrap.pypa.io/get-pip.py
$ python3 get-pip.py --user
```
The `--user` option installs pip to `~/.local/bin`, so lets add this to our `PATH` environment variable. It is also a
good idea to install the AWS CLI command completer (If you are not using `zsh`, then load the appropriate script for your
shell).
```shell
$ export PATH=~/.local/bin:$PATH
$ source ~/.local/bin/aws_zsh_completer.sh
```
You can verify successful installation by sourcing the `.zshrc` file and then running `pip3 --version`.

Now that we have the necessary requirements, lets install AWS CLI...
```shell
$ pip3 install awscli --upgrade --user
```
To verify installation, run `aws --version`.

Before we can use the AWS CLI, we need to configure it. First log into the AWS Website Console and go to your Account
Security Credentials. Create an access key to use for the AWS CLI credentials. Next run the following config wizard and
enter your credentials. You will be asked for a default region name - specify the **region** that matches where your
AWS instances are located.
```shell
$ aws configure
```

### 14. Setting the LOCALE for your Terminal
By default, the en_US locale is already installed.
```shell
$ sudo locale-gen en_GB.UTF-8
```
To set the LOCALE, run the following command - you will need to restart your terminal for the changes to take effect.
```shell
$ sudo update-locale LANG=en_GB.UTF-8

or

$ sudo update-locale LANG=en_US.UTF-8
```
To confirm your locale settings are correct, run the command `locale` on both your local terminal and on the remote host
via SSH and check if they match.

### 15. Git & Git Tools for Windows
GUI for git are a lot nicer to work with. My personal favourites being SourceTree(https://www.sourcetreeapp.com/) by
Atlassian. 

Most Git applications for windows either use [Git for Windows](https://git-scm.com/download/win) or a localised git
version. It makes managing the same version of Git easier to install and use this standard version. Window's git has
it's own configuration which is saved in your Windows Profile.

It makes sense to link to the same excludes file we created for WSL, so we'll need to find the file within the WSL
rootfs directory mentioned earlier. Either use a `Git Bash shell` or `Windows Command Prompt` to run additional git
configuration (just essential options) for how you intend to use Windows git tools.
```shell
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com
$ git config --global core.autocrlf input
$ git config --global core.filemode false
$ git config --global core.excludesfile C:\\Users\\<windows_username>\\AppData\\Local\\Packages\\CanonicalGroupLimited.Ubuntu18.04onWindows_79rhkp1fndgsc\\LocalState\\rootfs\\home\\<WSL_username>\\.gitignore_global
```

SourceTree has it own configuration settings so apply the same settings appropriately. Do be sure to **remember** and
configure `core.filemode` setting per repository loading into SourceTree (settings cog in toolbar), `Edit Config File`. 

In General options, set the SSH Client to OpenSSH so you can load your SSH key for github that you created in WSL
rootfs or your windows profile if you made a local copy.

In Git options, `Global Ignore List`, select the git excludes file with the **WSL rootfs path**.

For `Git Version`, ensure that `System` is selected.
