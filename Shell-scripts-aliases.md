[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=plastic)](https://opensource.org/licenses/MIT)

##### Git - Reset files based on status attribute
```shell
#!/usr/bin/env bash

# Color vars
_RST_='\033[0m'  #reset
_BLD='\033[1m'   #bold

#Normal Colors
Black='\033[38;5;0m'
Red='\033[38;5;1m'
Green='\033[38;5;2m'
Yellow='\033[38;5;3m'
Blue='\033[38;5;4m'
Magenta='\033[38;5;5m'
Cyan='\033[38;5;6m'
White='\033[38;5;7m'

# High Intensty
IBlack='\033[38;5;8m'
IRed='\033[38;5;9m'
IGreen='\033[38;5;10m'
IYellow='\033[38;5;11m'
IBlue='\033[38;5;12m'
IMagenta='\033[38;5;13m'
ICyan='\033[38;5;14m'
IWhite='\033[38;5;15m'

params=$*
if [[ -z $params ]]; then
    echo "You must specify status codes - use 'git status -s'"
    exit 1
fi

files=$(git status -s|grep -E "^${*}\s+"|sed -r "s/^${*}[[:space:]]+//")
echo -e "\n${Green}The following files will be GIT ${_BLD}${Red}reset${_RST_}${Green}:${_RST_}"
echo -e "${files}\n${Yellow}"

answer=""
while [[ -z $answer ]]; do
    read -rp 'Do you want to continue? [y/n]' input

    case $input in
        [Yy][Ee][Ss]|[Yy])
            answer=true
        ;;
        [Nn][Oo]|[Nn])
            exit 0
        ;;
    esac
done

echo "${files}" | while read value; do
    git checkout ${value}
done
```

##### Git - Apply stash by Stash ID
```shell
#!/bin/bash

git stash apply stash@{${1}}
```

##### Git - Show current branch's parent
```shell
#!/usr/bin/env zsh

git show-branch -a \
| grep '\*' \
| grep -v `git rev-parse --abbrev-ref HEAD` \
| head -n1 \
| sed 's/.*\[\(.*\)\].*/\1/' \
| sed 's/[\^~].*//'
```

##### Rsync
```shell
#!/usr/bin/env bash

# Color vars
_RST_='\033[0m'  #reset
_BLD='\033[1m'   #bold
_UND='\033[4m' #underline

# Normal Colors
Black='\033[38;5;0m'
Red='\033[38;5;1m'
Green='\033[38;5;2m'
Yellow='\033[38;5;3m'
Blue='\033[38;5;4m'
Magenta='\033[38;5;5m'
Cyan='\033[38;5;6m'
White='\033[38;5;7m'

# High Intensty
IBlack='\033[38;5;8m'
IRed='\033[38;5;9m'
IGreen='\033[38;5;10m'
IYellow='\033[38;5;11m'
IBlue='\033[38;5;12m'
IMagenta='\033[38;5;13m'
ICyan='\033[38;5;14m'
IWhite='\033[38;5;15m'

HOSTIP="123.123.123.123"
REMOTE_USER="remote_user"
PROJ_ROOT_DIR="my-proj-root"
REQUIRED_FILE='./local_project_filepath_that_must_exist'
DEST="/remote/path"
DIR="$(pwd|sed 's/.*\///')"
SSHKEY="${HOME}/.ssh/ssl_cert.pem"
EXCL="./.rsync_excludes"
EXCL_FILE='project_config_file_to_exclude'
SOURCE="$(pwd)/"

echo -e "${_BLD}${Yellow}RSYNC to Server: ${HOSTIP}${_RST_}"
echo "(from) $SOURCE"
echo "(to) $DEST"

if [[ "$DIR" != "$PROJ_ROOT_DIR" ]] || [[ ! -e $REQUIRED_FILE ]]; then
        echo -e "${_BLD}${Red}Error${_RST_}: Are you sure you are in the project's ROOT directory?";
        exit 1
fi

if [[ ! -e $SSHKEY ]] || [[ ! -r $SSHKEY ]]; then
        echo -e "${_BLD}${Red}Error${_RST_}: Could not find/read SSH Key in $SSHKEY"
        exit 1
fi

answer=''
RSDEL=''
while [[ -z "$answer" ]]
do
    echo -en "${_BLD}${Red}DELETE${_RST_} files on remote that don't exist on local? (excludes not deleted)"
    read -rp ' [y|n]: ' input

    case $input in
        [Yy][Ee][Ss]|[Yy])
            RSDDEL='--delete-after'
            answer=yes
        ;;
        [Nn][Oo]|[Nn])
            answer=no
        ;;
    esac
done

answer=''
RSDRY=''
while [[ -z "$answer" ]]
do
    echo -en "Do you want to use DRY MODE?"
    read -rp ' [y|n]: ' input

    case $input in
        [Yy][Ee][Ss]|[Yy])
            RSDRY='n'
            answer=yes
        ;;
        [Nn][Oo]|[Nn])
            answer=no
        ;;
    esac
done

rsync -zarvhP$RSDRY --exclude-from="$EXCL" --exclude="$EXCL_FILE" $RSDEL -e "ssh -i $SSHKEY" --rsync-path='sudo rsync' --chown=apache:apache ${SOURCE} ${REMOTE_USER}@${HOSTIP}:${DEST}
```

##### Start-up script
```shell
export DOCKER_HOST=tcp://localhost:2375
```

##### Aliases
```shell
# Docker
alias dstop='docker stop $(docker ps -a -q)'
alias drm='docker rm $(docker ps -aq)'
alias drmi='docker rmi -f $(docker images -q)'
alias dprune='docker system prune'
alias dnet='docker network create nginx-proxy'
alias dex="docker exec -it"
alias dps="docker ps"

# Git Tag
function gtagdel() {
  git push origin :refs/tags/$*
}
function gtagf() {
  git tag -fa $*
}
function gtagp() {
  git push origin $*
}

# List files in profile bin directory
alias lslocbin="echo \"~/.local/bin\" && ls -l ~/.local/bin"
```

#####
