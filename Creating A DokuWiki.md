# Creating a DokuWiki

1. download nginx (though any server would do, Apache, IIS, OpenBSD, etc.) and php (FastCGI proxying)
2. download Dokuwiki, unpack tarball, and put in www/ in root folder of nginx
3. put extracted php/ in root folder of nginx
4. download start-nginx.exe/start-nginx.ini for nice Windows GUI so that you don't need to manually run php-cgi.exe and start nginx separately at the same time
5. cook nginx.conf (see below)
6. start-nginx.exe > Start Server
7. print.php (phpinfo()) to debug nginx+php integration
8. browse /install.php (can use MAMP to get done first)
9. fill out username, acl, etc., delete when done
10. enjoy


```conf
worker_processes  1;
# single threaded process. Generally set to be equal to the number of CPUs or cores.

events {
    worker_connections  1024;
    # worker_processes and worker_connections allows you to calculate maxclients value: 
    # max_clients = worker_processes * worker_connections
}

http {
    include       mime.types;
    # anything written in /opt/nginx/conf/mime.types is interpreted as if written inside the http { } block

    default_type  application/octet-stream;

    sendfile        on;
    # If serving locally stored static files, sendfile is essential to speed up the server,
    # But if using as reverse proxy one can deactivate it
    
    keepalive_timeout  65;
    # timeout during which a keep-alive client connection will stay open.

    server {
	listen       80;
        # listen 80; is equivalent to listen *:80;
        
        server_name  localhost;

        root www;
		# --- for document_root to find php files
		
		location / {
			index   index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        #error_page   500 502 503 504  /50x.html;
        #location = /50x.html {
        #    root   html;
        #}
		
		location ~ ^/dokuwiki/.*\.php {     # or simply location ~ \.php to match all
			fastcgi_pass   127.0.0.1:9000;
			fastcgi_index  index.php;
			fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
			fastcgi_param  QUERY_STRING       $query_string;
			include        fastcgi_params;
		}
		
		
		# serve static files
		location ~ ^/dokuwiki/lib/  {	
			expires 30d;
		}
		
		location ~ ^/dokuwiki/conf/ { deny all; }
		location ~ ^/dokuwiki/data/ { deny all; }
		location ~ /\.ht            { deny all; }
        
```
