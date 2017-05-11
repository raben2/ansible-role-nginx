Ansible Role: Nginx
===================

Installs Nginx on RedHat/CentOS or Debian/Ubuntu Linux, or FreeBSD servers.

This role installs and configures the latest version of Nginx from the Nginx yum repository (on RedHat-based systems) or via apt (on Debian-based systems) or pkgng (on FreeBSD systems). You will likely need to do extra setup work after this role has installed Nginx, like adding your own [virtualhost].conf file inside `/etc/nginx/conf.d/`, describing the location and options to use for your particular website.

## Requirements

None.

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

## Virtual Hosts 

```yaml
    nginx_vhosts:
      - listen: "0.0.0.0"
        server_name: "example.com"
        root: "/var/www/example.com"
        index: "index.php index.html index.htm"
        error_page: ""
        port: 80
        access_log: ""
        error_log: ""
        ssl: true
        crt: example.com-cert.pem
        key: example.com-key.pem
```
This will create a vhost using php fpm socket and ssl certificates which are expected in {{ nginx_tls_cert_path }} please note that the default ciphers are hardened for security not compatiblity. 

```yaml 
   nginx_vhosts:
    - listen: "0.0.0.0"
      server_name: "proxy.example.com"
      proxy_zone: "static"
      port: 80
      proxy_pass: "http://localhost:8080/"
      cache_bypass: '$cookie_nocache'
      index: "index.html index.htm"
      root: "/"
      ssl: false
      location:
          - uri: '/mail/'
            regex: none
            modifier: none
            proxy_pass: 'http://localhost:8081/'
```
This example shows a vhost configuration with proxy configuration for two locations

```yaml
   nginx_vhosts:
    - listen: "0.0.0.0"
      server_name: "proxy.example.com"
      proxy_zone: "static"
      port: 80
      proxy_pass: "http://localhost:8080/"
      cache_bypass: '$cookie_nocache'
      index: "index.html index.htm"
      root: "/"
      ssl: false
      location:
       - uri: '^/'
         modifier: '~'
         regex: '(?:build|tests|config|lib|3rdparty|templates|data)/'
         extra_params: 
             - 'deny all'
```
This example creates a vhost with modifier ~ and regex for dening access to several uris

```yaml
   nginx_vhosts:
    - listen: "0.0.0.0"
      server_name: "proxy.example.com"
      proxy_zone: "static"
      port: 80
      proxy_pass: "http://localhost:8080/"
      cache_bypass: '$cookie_nocache'
      index: "index.html index.htm"
      add_header:
        - 'X-Real-IP       $remote_addr'
        - 'X-Forwarded-For $proxy_add_x_forwarded'
      root: "/"
      ssl: false
      location:
         - uri: '^/content/'
           modifier: '~'
           regex: '(?:index|remote|public|cron)\.php(?:$|/)'
           add_header: 
             - 'Cache-Control "public, max-age=7200'
           extra_params:
             - 'fastcgi_split_path_info ^(.+\.php)(/.*)$' 
             - 'include fastcgi_params'
             - 'fastcgi_pass {{ nginx_php_fpm_upstream_name }}'
````
Generate a vhost passing requests to php-fpm using custom headers for the proxy and the /content/ location.

```yaml
    nginx_remove_default_vhost: false
```
Whether to remove the 'default' virtualhost configuration supplied by Nginx. Useful if you want the base `/` URL to be directed at one of your own virtual hosts configured in a separate .conf file.

```yaml
    nginx_install_php_fpm: True
    nginx_php_version: '7.0'
    nginx_php_dependencies:
      - json
      - xml
    nginx_php_socket: "{{ nginx_pid_path }}/php/php{{ nginx_php_version }}-fpm.sock"
    nginx_php_fpm_upstream_name: php-fpm
    nginx_upstreams:
        - name: "{{ nginx_php_fpm_upstream_name }}"
          servers:
        - "unix:{{ nginx_php_socket }}"
```
Install php-fpm and configure a upstream


## Nginx System parameters
```yaml
    nginx_user: "nginx"
```
The user under which Nginx will run. Defaults to `nginx` for RedHat, and `www-data` for Debian.

```yaml
    nginx_worker_processes: "1"
    nginx_worker_connections: "1024"
    nginx_multi_accept: "off"
    nginx_error_log: "/var/log/nginx/error.log warn"
    nginx_access_log: "/var/log/nginx/access.log main buffer=16k"
```
Configuration of the default error and access logs. Set to `off` to disable a log entirely.

```yaml
    nginx_sendfile: "on"
    nginx_tcp_nopush: "on"
    nginx_tcp_nodelay: "on"
    nginx_keepalive_timeout: "65"
    nginx_keepalive_requests: "100"
    nginx_client_max_body_size: "64m"
    nginx_server_names_hash_bucket_size: "64"
    nginx_proxy_cache_path: ""
    nginx_extra_http_options: ""
```
Extra lines to be inserted in the top-level `http` block in `nginx.conf`. The value should be defined literally (as you would insert it directly in the `nginx.conf`, adhering to the Nginx configuration syntax - such as `;` for line termination, etc.), for example:

```yaml
    nginx_ppa_use: false
    nginx_ppa_version: stable
```
(For Ubuntu only) Allows you to use the official Nginx PPA instead of the system's package. You can set the version to `stable` or `development`.

## ToDo
* Add installation of nginx from source
* add passenger upstream for gitlab installations

## Dependencies

None.

## Example Playbook
```yaml
    - hosts: server
      roles:
        - { role: raben2.nginx }
```
## License

LGPL

## Author Information

This role was created in 2017 by [Georg Rabenstein](https://github.com/raben2)