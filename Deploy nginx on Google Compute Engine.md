# Deploy nginx on Google Compute Engine

1. create new GCP SDK project with Cloud Run and Secret Manager APIs
2. `gcloud components update` + `gcloud init` + `gcloud auth login`
3. nginx container on vm? prepacked bitnami dokuwiki? sudo apt-get install nginx + nginx start?
4. do last, no use docker and have more control
5. realize that's gonna cost ya (best can do is $4.36/m for prepacked bitnami dokuwiki on 0.6 gb mem)
6. ok nvm get 1 e2-micro free/month
7. create vm new instance in GCE, allow http and https traffic

```
gcloud compute instances list
NAME        ZONE           MACHINE_TYPE  PREEMPTIBLE  INTERNAL_IP  EXTERNAL_IP  STATUS
instance-1  us-central1-a  e2-micro                   some.ip      some.ip (ephemeral)      RUNNING
```
8. `gcloud compute scp` - perms denied on remote /www, so dumped instead to /home/christinazhang2013, then `sudo mv` to /opt/www in ssh
9. update nginx.conf (see below) and scp it to /opt/www
10. add startup script (see below) - to debug: 

```
gcloud compute instances get-serial-port-output instance-1 --zone us-central1-a
```

10. reset vm


```sh
 #! /bin/bash
export HOME=/root

apt update                                           

# Install Nginx and php 7.4 (and php 7.4 fpm) dependencies
apt-get install -y git nginx 

apt-get install -y git php7.4 php7.4-fpm

# Disable the default NGINX configuration
rm /etc/nginx/sites-enabled/nginx.conf

# Overwrite the current default configuration
cp /opt/conf/nginx.conf /etc/nginx/sites-available/nginx.conf
ln -s /etc/nginx/sites-available/nginx.conf /etc/nginx/sites-enabled/nginx.conf
cp /opt/conf/fastcgi_params /etc/nginx/fastcgi_params

# Start the Nginx service
service nginx start
# Start php service
sudo service php7.4-fpm start
```



```conf
server {
    listen 80;
    server_name _;

    root /opt/www;
    index doku.php;

    location / {
        try_files $uri $uri/ @dokuwiki;
    }

    location @dokuwiki {
        rewrite ^/_media/(.*) /lib/exe/fetch.php?media=$1 last;
        rewrite ^/_detail/(.*) /lib/exe/detail.php?media=$1 last;
        rewrite ^/_export/([^/]+)/(.*) /doku.php?do=export_$1&id=$2 last;
        rewrite ^/(.*) /doku.php?id=$1&$args last;
    }

    location ~ /\.ht {
        deny all;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock; # Adjust version if needed
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /(data|conf|bin|inc)/ {
        deny all;
    }

    access_log /var/log/nginx/dokuwiki_access.log;
    error_log /var/log/nginx/dokuwiki_error.log;
}
```

### wonks:

GCP allows only few port accesses by default, so may need to enable port 80 in firewall config list

```
gcloud compute firewall-rules create allow-80 --allow tcp:80
```

only php-7.4 package is supported on the debian image for this vm.

```
nginx: [emerg] bind() to [::]:80 failed (98: Address already in use)
```

use netstat via ssh (don't have to worry about priv issues)

dokuwiki may complain about inaccessible /opt/www/data/*. can give ownership to webserver (and not chmod 777 ha):

```
sudo chown -R www-data:www-data /opt/www
```

![image](https://github.com/christinazhangstudio/casa-del-big-dragon/assets/57076552/472c4a88-f336-46b1-b21a-eade5943240e)


