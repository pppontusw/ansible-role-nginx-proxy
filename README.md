# Nginx HTTP, HTTPS & Redirect Proxy

## What does it do?

1. Installs Nginx
2. Creates an SSL certificate and DH parameter store location as defined in {{nginx_proxy_ssl_store_path}}
3. Generates secure DHparams (2048)
4. Copies all the SSL certificates and keys defined in ssl sites to your SSL store
5. Copies the custom Nginx configuration with only secure ciphers
6. Copies all of your virtual site configurations defined in defaults/main or {group,host}_vars to sites-available
7. Symlinks all of the above virtual sites to sites-enabled

## What do you have to do?

1. Take a look at the defaults/main file to look at the available option for virtual sites
2. Move the defaults/main file to your group_vars or host_vars (you can of course edit in place if you wish, but it isn't recommended)
3. Look at the example playbooks and read the rest of the documentation

It is recommended that you put your certificates and keys into a separate vault-encrypted file and only reference them in the regular vars or defaults file, in order to protect them

## How can I use this role?

We can use this role to set up a few things that we may want on a forward facing reverse web proxy:

Type | Variable | Use
--- | --- | ---
***Reverse HTTPS Proxy*** | nginx_proxy_ssl_virtual_sites | By default proxies incoming HTTPS connections to an upstream HTTP or HTTPS server - redirects HTTP connections to HTTPS
***Reverse HTTP Proxy*** | nginx_proxy_virtual_sites | Same as above but proxies regular HTTP connections to an upstream HTTP or HTTPS server
***HTTPS Redirect Proxy*** | nginx_redirect_ssl_virtual_sites | Redirects HTTP and HTTPS requests with a 301 to the link you specify
***HTTP Redirect Proxy*** | nginx_redirect_virtual_sites | Redirects HTTP requests with a 301 to the link you specify

This is a simple playbook that sets up this role using the hosts called "reverse-proxies" with all the virtual sites defined in group_vars/reverse-proxies.yml

```
# example-playbook.yml
# sets up basic account and security configuration
- name: reverse proxy
  hosts: reverse-proxies 
  roles: 
    - nginx-proxy
```

Remember, you probably want to remove or comment out the defaults/main.yml file before running 

``` ansible-playbook example-playbook.yml ```

so that it doesn't try to set up my example sites.


### section needs to be completed

All config directives:

nginx_proxy_ssl_virtual_sites:

Name | Example | Required | Use
--- | --- | --- | ---
site_id | example | Yes | will be the name for upstream server and the sites-available file, must be unique
http_port | 80, | No (default: 80) *ignored if custom_http_block is used* | http port for nginx to listen to
https_port | 443 | No (default: 443) *ignored if custom_https_block is used* | https port for nginx to listen to
server_names | example.com example2.com | Yes | server names to listen for
upstream_name | example | No (default: site_id) *ignored if upstream_server is not set* | name for the upstream connection
upstream_server | 10.10.10.10:80 | No | upstream server/port to connect to
upstream_extras | [keepalive 20] | No  | array of extra config directives for upstream 
proxy_pass | http://example | Yes | proxy pass should be the http-link to pass requests to
ssl_cert | ---BEGIN CERTIFICATE---... | Yes, if https_port is defined | SSL certificate
ssl_key: | ---BEGIN PRIVATE KEY---... | Yes, if https_port is defined | SSL key
ssl_protocols | 'TLSv1 TLSv1.1 TLSv1.2' | No (default: 'TLSv1 TLSv1.1 TLSv1.2') | ssl protocols we should accept for this site 
proxy_redirect | http://10.2.0.100:8080 https://example.com | No | proxy_redirect nginx directive (rewrite host)
custom_http_block: | server { listen 80; etcetc } | No | here we can substitute the standard http->https redirect block with a custom server {} block if we want, will negate all other HTTP options
custom_https_block: | server { listen 443; etcetc } | No | here we can substitute the standard http->https redirect block with a custom server {} block if we want, will negate all other HTTP options
letsencrypt | true/false | No | sets up letsencrypt store for your server (site_id must match servername)

```

Compability
-----

Tested on:

Debian 6, 7 & 8

Ubuntu 12.04 & 14.04

CentOS 7
