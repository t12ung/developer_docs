[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=plastic)](https://opensource.org/licenses/MIT)

[![Poweredby: docker](https://img.shields.io/badge/docker-v18.09.6-lightgrey.svg?style=plastic&logo=docker&logoColor=white&labelColor=2496ed)](https://www.docker.com/) [![Poweredby: nginx](https://img.shields.io/badge/nginx-v1.17.6-lightgrey.svg?style=plastic&logo=nginx&logoColor=white&labelColor=047832)](https://github.com/nginxinc/docker-nginx) [![Poweredby: nginx](https://img.shields.io/badge/docker--gen-v0.7.3-lightgrey.svg?style=plastic&logo=nginx&logoColor=white&labelColor=047832)](https://github.com/jwilder/docker-gen)
# How to run Multiple Websites (Apache/MySQL) in your local Docker development environment
To achieve this, we will be using a reverse proxy called Nginx.
> Nginx (pronounced "engine-x") is an open source reverse proxy server for HTTP, HTTPS, SMTP, POP3, and IMAP protocols, as well as a load balancer, HTTP cache, and a web server (origin server).

We will also be utilizing a companion docker container to automate Nginx configuration, called docker-gen. This container can run alongside the offcial nginx container and *watch* for any changes to our development setup. It is also available bundled together in a single container with Nginx, referred to as [nginx-proxy](https://github.com/jwilder/nginx-proxy). Once we've got our proxy set up, we need to tell our website containers to communicate with the proxy.

![Reverse Proxy UML](http://plantuml.com:80/plantuml/svg/5OqxZW8n40NpFSLoW74BGgAv2JBsquo5FrRtQxW-9ggWfAhT69NUwj-bz5GzmxN-d-IqkuZ6JpWAJt-wGTFeH6T8emclD--7PBnHnuG-yaWcBHgXOffa9QLvPTcMUK4tAuI5-Lljw7dn2m00)

### Proxy Container Set Up
Create a directory to contain the files for your proxy container. Let's use `docker-nginx-proxy` for this guide. Inside this, create directories for `certs`, `conf.d` and `vhost.d`. Using your favourite terminal shell and change into our proxy container directory.

```shell
curl "https://raw.githubusercontent.com/jwilder/nginx-proxy/master/nginx.tmpl" > nginx.tmpl
```

If you've created SSL certificates for your sites, put them in the `certs` directory. [See this guide if you want to create these](./SSL-localhost-letsencrypt.md).

Create a file called `proxy-settings.conf` inside the container directory. Place inside here any additional proxy-wide nginx settings you require.

For dev/debug purposes, the following settings can be useful:
```
proxy_connect_timeout       5m;
proxy_send_timeout          5m;
proxy_read_timeout          60m;
send_timeout                5m;
```

Create a file called `docker-compose.yml` inside the container directory. The following is for a single docker container. If you want to run nginx and docker-gen separately, then check [jwilder/docker](https://github.com/jwilder/docker-gen#separate-container-install) for details.

> **`docker-compose.yml`** for docker-nginx-proxy
```ini
version: "3.7"

services:
  nginx-proxy:
    image: jwilder/nginx-proxy
    container_name: 'nginx-proxy'
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./proxy-settings.conf:/etc/nginx/conf.d/proxy-settings.conf
      - ./vhost.d:/etc/nginx/vhost.d
      - ./certs:/etc/nginx/certs
      - ./nginx.tmpl:/etc/docker-gen/templates/nginx.tmpl
      - /var/run/docker.sock:/tmp/docker.sock:ro

networks:
  default:
    external:
      name: nginx-proxy
```
Before we can build our docker container for the proxy, we need to create the external network that will be used for the project containers to communicate with the proxy container. To do this, run the command `docker network create nginx-proxy`.

Start up the container with `docker-compose up --build`. If everything runs successfully, we should have our configuration file built automatically in `conf.d` directory.

### Website Container Set Up
I won't provide a full `docker-compose.yml` for the websites, check out other guide for how to run docker containers for websites. Instead, I will just emphasize the differences needed to have these containers communicate with the nginx-proxy container.

> **`docker-compose.yml`** for website
```dockerfile
version: "3.7"

webserver:
  expose:
    - 80
  environment:
    - VIRTUAL_HOST=website.dev.tslombard.com
    - CERT_NAME=<name> # without extension
    - HTTPS_METHOD=redirect # values: nohttp, nohttps, noredirect, redirect (default)
    
networks:
  default:
    external:
      name: nginx-proxy
```

We don't publish the ports here as you would without running the reverse proxy - these are published in `nginx-proxy` service. Instead we only expose the port to other containers in the network. You should also notice that we haven't exposed `port 443` for our webserver, this is handled automatically by Nginx so it's not needed to specify it here. The network specified also **matches** that which is specified in `docker-nginx-proxy`, otherwise the two containers would not be able to communicate with each other. (If your docker-compose files are using a different version, refer to [Reference documentation](https://docs.docker.com/compose/compose-file/) for network configuration settings.

For the `environment` variables, the minimum is to specify `VIRTUAL_HOST`, otherwise Nginx won't know which container to route a request to. The same value should be added to your machine's host(s) file and point to `127.0.0.1`. Nginx automatically applies the SSL certificates matching the `VIRTUAL_HOST` variable (down to the most specific level in domain tree), if the certficates exist in the `certs` directory. We can optionally point to a specific certificate using the `CERT_NAME` variable. The `HTTPS_METHOD` is also an optional setting.

### Running multiple MySQL database containers

While it's possible to configure Nginx to proxy TCP port 3306 (MySQL default port) - nginx-proxy does not support this, currently (Check the discussion here for more information). Instead, it's easier to just publish each MySQL container on different ports to the localhost. For example:
```dockerfile
ports:
  - "3307:3306"
```
With this setting, we can access externally on port 3307, (i.e. MySQL client using `localhost:3307`), whilst our docker containers connect internally on the `nginx-proxy` network on port 3306 (i.e. `<docker_container_name>:3306`).
