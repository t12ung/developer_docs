[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=plastic)](https://opensource.org/licenses/MIT)

[![Certbot](https://img.shields.io/static/v1.svg?label=&message=certbot&color=ec1c23&style=plastic)](https://certbot.eff.org/) [![letsencrypt](https://img.shields.io/static/v1.svg?label=&message=%20letsencrypt&color=2C3C69&style=plastic)](https://letsencrypt.org/) [![Cloudflare](https://img.shields.io/static/v1.svg?label=&message=Cloudflare&color=f38020&style=plastic&logo=cloudflare&logoColor=white)](https://dash.cloudflare.com/)
### SSL Certificates for local development
##### PRE-REQUISITE
To generate valid SSL certificates, I use a free open source service provided by [Let’s Encrypt](https://letsencrypt.org/). These certificates have a validity period of 90 days, but they can be renewed. The only thing you need to be wary of are their [rate limits](https://letsencrypt.org/docs/rate-limits/). I suggest you test using the option parameter `--dry-run` to test for validation checks, omitting this when you need to generate the actual certificate to use on your server. The prerequisite for validation is that the domain you want to use **must be publicly reachable**. There are multiple [validation methods](https://letsencrypt.org/docs/challenge-types/), but I'll be using DNS as it's the most clean and easy method (nothing to create and clean up afterwards).

_If you don't have your own domain, you could use a free service like [Heroku](https://www.heroku.com/free) with http validation._

Before I start this how-to guide, you should know what development environment I am using...
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
On the last command, `python3-certbot-dns-cloudflare` is the DNS plug-in name. My choice is to use Cloudflare, but you could use any of the other supported [DNS Plugins](https://certbot.eff.org/docs/using.html#dns-plugins).

##### Domain choice
For a production environment, you'll most likely want a single certificate for each site due to security risks. For development, it makes more sense to have a 'catch-all' domain so we can use the same certificate for all our development sites. I'd suggest something like, `dev.yourdomain.com` or `local.yourdomain.com`. For our local sites, we then set them up as sub-domains and modify our hosts file appropriately to point to `127.0.0.1`.

##### Cloudflare API
Create a text file to store your account email and API key and make sure the permissions of the file are something like `600 root:root` to prevent access from prying eyes. The file should contain something like:
```ini
dns_cloudflare_email = "your.email@example.com"
dns_cloudflare_api_key = "your-api-key"
```

> Run certbot commands with `-q` flag to suppress output for non-interactive mode

##### Generate Certificate
```shell
sudo certbot -d "dev.example.com,*.dev.example.com" --dns-cloudflare --dns-cloudflare-credentials <path_to_file>/cloudflare.ini --dns-cloudflare-propagation-seconds 10 certonly
```
Generated certificate files are not physically in the `live` directory as the letscrypt output suggests - these are links to actual files in 'archive' directory. The files we are interested in for our site(s) are `cert` and `privkey` pem files. These correlate to `<domain>.crt` and `<domain>.key` respectively.

##### Renew Certificate
You can renew specific domains with `-d` or omit to renew ALL domains.
```shell
sudo certbot renew --dry-run --manual-auth-hook <path_to_file>/authenticator.sh --manual-cleanup-hook <path_to_file>/cleanup.sh --dns-cloudflare-credentials <path_to_file>/cloudflare.ini
```

> **authenticator.sh**
```shell
#!/bin/bash

# Get your API key from https://www.cloudflare.com/a/account/my-account
API_KEY="your-api-key"
EMAIL="your.email@example.com"

# Strip only the top domain to get the zone id
DOMAIN=$(expr match "$CERTBOT_DOMAIN" '.*\.\(.*\..*\)')

# Get the Cloudflare zone id
ZONE_EXTRA_PARAMS="status=active&page=1&per_page=20&order=status&direction=desc&match=all"
ZONE_ID=$(curl -s -X GET "https://api.cloudflare.com/client/v4/zones?name=$DOMAIN&$ZONE_EXTRA_PARAMS" \
     -H     "X-Auth-Email: $EMAIL" \
     -H     "X-Auth-Key: $API_KEY" \
     -H     "Content-Type: application/json" | python -c "import sys,json;print(json.load(sys.stdin)['result'][0]['id'])")

# Create TXT record
CREATE_DOMAIN="_acme-challenge.$CERTBOT_DOMAIN"
RECORD_ID=$(curl -s -X POST "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/dns_records" \
     -H     "X-Auth-Email: $EMAIL" \
     -H     "X-Auth-Key: $API_KEY" \
     -H     "Content-Type: application/json" \
     --data '{"type":"TXT","name":"'"$CREATE_DOMAIN"'","content":"'"$CERTBOT_VALIDATION"'","ttl":120}' \
             | python -c "import sys,json;print(json.load(sys.stdin)['result']['id'])")
# Save info for cleanup
if [ ! -d /tmp/CERTBOT_$CERTBOT_DOMAIN ];then
        mkdir -m 0700 /tmp/CERTBOT_$CERTBOT_DOMAIN
fi
echo $ZONE_ID > /tmp/CERTBOT_$CERTBOT_DOMAIN/ZONE_ID
echo $RECORD_ID > /tmp/CERTBOT_$CERTBOT_DOMAIN/RECORD_ID

# Sleep to make sure the change has time to propagate over to DNS
sleep 10
```

> **cleanup.sh**
```shell
#!/bin/bash

# Get your API key from https://www.cloudflare.com/a/account/my-account
API_KEY="your-api-key"
EMAIL="your.email@example.com"

if [ -f /tmp/CERTBOT_$CERTBOT_DOMAIN/ZONE_ID ]; then
        ZONE_ID=$(cat /tmp/CERTBOT_$CERTBOT_DOMAIN/ZONE_ID)
        rm -f /tmp/CERTBOT_$CERTBOT_DOMAIN/ZONE_ID
fi

if [ -f /tmp/CERTBOT_$CERTBOT_DOMAIN/RECORD_ID ]; then
        RECORD_ID=$(cat /tmp/CERTBOT_$CERTBOT_DOMAIN/RECORD_ID)
        rm -f /tmp/CERTBOT_$CERTBOT_DOMAIN/RECORD_ID
fi

# Remove the challenge TXT record from the zone
if [ -n "${ZONE_ID}" ]; then
    if [ -n "${RECORD_ID}" ]; then
        curl -s -X DELETE "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/dns_records/$RECORD_ID" \
                -H "X-Auth-Email: $EMAIL" \
                -H "X-Auth-Key: $API_KEY" \
                -H "Content-Type: application/json"
    fi
fi
```

