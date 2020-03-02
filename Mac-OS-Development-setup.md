[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=plastic)](https://opensource.org/licenses/MIT)

# Mac OS Development Environment Set Up Guide

### 1. Install Xcode
Search for it in the App Store. You should also install the CLI Tools:
```shell
$ xcode-select --install
```

### 2. Install iTerm2
Download it from [here](https://www.iterm2.com). After install, set up the follow keyboard shortcuts:
<table>
<tr>
<td>⌘⌥←</td>
<td>Previous Tab</td>
</tr>
<tr>
<td>⌘⌥→</td>
<td>Next Tab</td>
</tr>
<tr>
<td>⌘←</td>
<td>Send Escape Sequence+ OH</td>
</tr>
<tr>
<td>⌘→</td>
<td>Send Escape Sequence+ OF</td>
</tr>
<tr>
<td>⌥←</td>
<td>Send Escape Sequence+ b</td>
</tr>
<tr>
<td>⌥→</td>
<td>Send Escape Sequence+ f</td>
</tr>
<tr>
<td>⌘←Delete</td>
<td>Send Hex Code 0x15</td>
</tr>
<tr>
<td>⌥←Delete</td>
<td>Send Hex Code 0x1B 0x08</td>
</tr>
</table>

### 3. Install Homebrew Package Manager
```shell
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

### 4. Update git to latest version
```shell
$ brew install git
```
After installation finishes, you need to restart your terminal for your shell to pick up new changes to the PATH environment
variable so that the new version of git in `/usr/local/bin` takes precedence over system git in `/usr/bin`.

### 5. Update zsh to latest version
```shell
$ brew install zsh
```
Restart your terminal after installation.

### 6. Install Oh My Zsh
```shell
$ sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```
Add some plugins to help make life easier in zsh. A list of default plugins included with oh-my-zsh can be found
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
<tr>
<td><a href="https://github.com/robbyrussell/oh-my-zsh/tree/master/plugins/alias-finder">alias-finder</a></td>
<td>Command accepts a phrase to search for matching aliases - check <a href="https://github.com/robbyrussell/oh-my-zsh/tree/master/plugins/alias-finder">github page</a> for extra options</td>
</tr>
<tr>
<td><a href="https://github.com/robbyrussell/oh-my-zsh/tree/master/plugins/common-aliases">common-aliases</a></td>
<td>A lot of commonly used ones - check <a href="https://github.com/robbyrussell/oh-my-zsh/tree/master/plugins/common-aliases">github page</a> for a full list</td>
</tr>
</table>

### 7. Install Tree
This command allows you to create an ascii directory/file tree customizable by depth with `-L` option that can be useful
for documentation purposes.
```shell
$ brew install tree
```

### 8. Update Bash version
```shell
$ brew install bash
```

### 9. Install Bash Completion
This brings up shell command completion after a double `Tab` key press.
```shell
$ brew install bash-completion
```

### 10. Update Vim to latest version
```shell
$ brew install vim
```

### 11. Install Ultimate Vim configuration
```shell
git clone --depth=1 https://github.com/amix/vimrc.git ~/.vim_runtime
sh ~/.vim_runtime/install_awesome_vimrc.sh
```

### 12. Update nano to latest version
```shell
$ brew install nano
```

### 13. Install nanorc Syntax Highlighting
https://github.com/scopatz/nanorc

### 14. Install Composer Package Manager
```shell
$ brew install composer
```

### 15. Install Node Package Manager
```shell
$ brew install npm
```

### 16. Others
While the previous suggestions are my own personal selection, you might find others here:
https://sourabhbajaj.com/mac-setup/

