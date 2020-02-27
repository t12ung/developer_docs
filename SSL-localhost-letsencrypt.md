[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=plastic)](https://opensource.org/licenses/MIT)

[![Certbot](https://img.shields.io/static/v1.svg?label=&message=certbot&color=ec1c23&style=plastic)](https://certbot.eff.org/) [![letsencrypt](https://img.shields.io/static/v1.svg?label=&message=%20letsencrypt&color=2C3C69&style=plastic)](https://letsencrypt.org/) [![Cloudflare](https://img.shields.io/static/v1.svg?label=&message=Cloudflare&color=f38020&style=plastic&logo=cloudflare&logoColor=white)](https://dash.cloudflare.com/)
### SSL Certificates for local development or production system
##### PRE-REQUISITE
To generate valid SSL certificates, I use a free open source service provided by [Let’s Encrypt](https://letsencrypt.org/).
These certificates have a validity period of 90 days, but they can be renewed. The only thing you need to be wary of are
their [rate limits](https://letsencrypt.org/docs/rate-limits/). I suggest you test using the option parameter
`--dry-run` to test for validation checks, omitting this when you need to generate the actual certificate to use on your
server. The prerequisite for validation is that the domain you want to use **must be publicly reachable**. There are
multiple [validation methods](https://letsencrypt.org/docs/challenge-types/), but I'll be using DNS as it's the most
clean and easy method (nothing to create and clean up afterwards).

_If you don't have your own domain, you could use a free service like [Heroku](https://www.heroku.com/free) with http validation._

For this guide, it is assumed you are using a development environment that is UNIX/Linux based. If your setup is
different, there are [other clients](https://letsencrypt.org/docs/client-options/#acme-v2-compatible-clients) that may
be compatible with your system. My development environment is...
```
Windows Subsystem for Linux (WSL)
Ubuntu 18.04 Distribution
```
##### Installation
First step is to install certbot, so start up your favourite shell and enter the following commands:
```shell
$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository universe
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt-get update
$ sudo apt-get install certbot python3-certbot-dns-cloudflare
```
On the last command, `python3-certbot-dns-cloudflare` is the DNS plug-in name. My choice is to use Cloudflare, but you
could use any of the other supported [DNS Plugins](https://certbot.eff.org/docs/using.html#dns-plugins).

##### Domain choice
For a production environment, you'll most likely want a single certificate for each site due to security risks. For
development, it makes more sense to have a 'catch-all' domain so we can use the same certificate for all our development
sites. I'd suggest something like, `dev.yourdomain.com` or `local.yourdomain.com`. For our local sites, we then set them
up as sub-domains and modify our hosts file appropriately to point to `127.0.0.1`.

##### Cloudflare API
Create a text file to store your account email and API key and make sure the permissions of the file are something like
`600 root:root` to prevent access from prying eyes. The file should contain something like:
```ini
dns_cloudflare_email = "your.email@example.com"
dns_cloudflare_api_key = "your-api-key"
```

> Run certbot commands with `-q` flag to suppress output, and `-n` for non-interactive mode

##### Generate Certificate
```shell
$ sudo certbot certonly -d "dev.example.com,*.dev.example.com" --dns-cloudflare --dns-cloudflare-credentials <path_to_file>/cloudflare.ini --dns-cloudflare-propagation-seconds 10
```
The above command can also be used to _**renew certificates for specific domains**_. Generated certificate files are not
physically in the `live` directory as the letscrypt output suggests - these are sym links to actual files in 'archive'
directory. The files we are interested in for our site(s) are `cert` and `privkey` pem files. These correlate to
`<domain>.crt` and `<domain>.key` respectively. On a production system, you should also implement the `chain` file for
OCSP Stapling. 

##### Renew All Certificates
```shell
$ sudo certbot renew --dns-cloudflare --dns-cloudflare-credentials <path_to_file>/cloudflare.ini --dns-cloudflare-propagation-seconds 10
```

##### Outro
The Cloudflare dns authentication automatically adds/removes the required dns entries for us, but it is also possible to
use shell scripts as hooks to have a custom process (useful for custom installation of ssl certificates on webserver).
See the [Certbot Documentation](https://certbot.eff.org/docs/using.html#pre-and-post-validation-hooks) for more details
on this.

It may be helpful to set up the renew command with -n (and suppress output with -r if you don't care) in a cron job to
keep your certificates up to date.

### Extra

##### Renewal and certificate management script
Here is a helpful shell script you can use to update certificates for your domain (including sub-domains) and
automatically rename/copy them to the source script's directory.
```shell
#!/bin/bash
 
AUTHFILE=cloudflare.ini
DOMAIN='dev.example.com'
DOMAINS="${DOMAIN},*.${DOMAIN}"
 
# Optionally, set your docker NginX certs directory (without trailing '/') below to also copy the generated certs there
NGINX_CERTDIR=''
 
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
 
# Check for SUDO for access to generated certs
if [[ $(id -un) != 'root' ]]; then
  echo -e "${_BLD}${Red}Error${_RST_}: require ${_BLD}sudo${_RST_} permissions - run: ${_BLD}sudo $0${_RST_}"
  exit 1
fi
 
# Set working directory is the location of this file to ensure relative file paths are correct
SOURCE="${BASH_SOURCE[0]}"
while [ -h "$SOURCE" ]; do # resolve $SOURCE until the file is no longer a symlink
  DIR="$( cd -P "$( dirname "$SOURCE" )" >/dev/null 2>&1 && pwd )"
  SOURCE="$(readlink "$SOURCE")"
  [[ $SOURCE != /* ]] && SOURCE="$DIR/$SOURCE" # if $SOURCE was a relative symlink, we need to resolve it relative to the path where the symlink file was located
done
DIR="$( cd -P "$( dirname "$SOURCE" )" >/dev/null 2>&1 && pwd )"
cd $DIR
 
# Check for authentication details
if [[ ! -f ./${AUTHFILE} ]]; then
  echo -e "${_BLD}${Red}Error${_RST_}: ${_BLD}${AUTHFILE}${_RST_} not found in ${_BLD}${DIR}${_RST_}"
fi
 
certbot certonly -d ${DOMAINS} --dns-cloudflare --dns-cloudflare-credentials ./${AUTHFILE} --dns-cloudflare-propagation-seconds 10
 
OUTPUT_PATH="/etc/letsencrypt/live/${DOMAIN}"
USER=$SUDO_USER
USER_GROUP=$(id -gn $USER)
if [[ $? == 0 ]]; then
  cp $OUTPUT_PATH/cert.pem $DIR/$DOMAIN.crt
  cp $OUTPUT_PATH/privkey.pem $DIR/$DOMAIN.key
  chmod 400 $DIR/$DOMAIN.crt
  chmod 400 $DIR/$DOMAIN.key
  chown $USER:$USER_GROUP $DIR/$DOMAIN.crt
  chown $USER:$USER_GROUP $DIR/$DOMAIN.key
  echo -e "Certificates generated successfully and copied to ${_BLD}${DIR}${_RST_}"
fi
 
# Optional
if [[ ! -z "$NGINX_CERTDIR" ]] && [[ -d "$NGINX_CERTDIR" ]]; then
  cp -f $DIR/$DOMAIN.crt $NGINX_CERTDIR
  cp -f $DIR/$DOMAIN.key $NGINX_CERTDIR
  echo -e "Certificates copied to ${_BLD}${NGINX_CERTDIR}${_RST_}"
fi
```

##### Certificate auto renewal
Using the script above, we can configure this to run when the certificates are older than 60 days, (normal renewals are
permitted once expiry is within 30 days). Modify the commands with the appropriate file paths to existing cert and the
cert renewal script.

If you want to set this up as a server cron job (root user) then:
```shell
$ echo -e "#Renew Certs if older than 60 days\n0 1 * * * test \$(( (\$(date +\%s) - \$(date -r path_to.crt +\%s) + 43200)/86400 )) -gt 60 && path_to_renew_scipt\n" >> /var/spool/cron/root
```

Alternative, you can add it to your shell startup script (`.zshrc`):
```shell
echo -e "\n#Renew Certs if older than 60 days" >> ~/.zshrc
echo -e "test \$(( (\$(date +\%s) - \$(date -r path_to.crt +\%s) + 43200)/86400 )) -gt 60 && echo 'sudo permissions required to renew SSL certificates...'" >> ~/.zshrc
echo -e "test \$(( (\$(date +\%s) - \$(date -r path_to.crt +\%s) + 43200)/86400 )) -gt 60 && sudo path_to_renew_scipt" >> ~/.zshrc
```
