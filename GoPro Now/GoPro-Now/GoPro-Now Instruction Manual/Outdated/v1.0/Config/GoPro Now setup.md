To set up a secure RTMP server using Nginx on your Raspberry Pi 4 with Raspberry Pi OS (bookworm), follow these steps:

1. **Install updates and upgrades:**

   ```bash
   sudo apt update
   sudo apt upgrade
   sudo reboot
   ```
2. **Configure raspi-config:**
   ```bash
   sudo raspi-config
   ```
When ready naigate to `Finish` and hit `Enter`.
A reboot request window will pop up, now navigate to `Yes` and hit `Enter` again to start rebooting the Raspberry Pi.

3. **Identify your Raspberry Pi's hostname as well as its IP for devices to access it:**

   ```bash
   sudo hostname
   ---

   It will print the name of the Raspberry Pi:
   ```
      gopro-now
   ---

  ```bash
   sudo hostname -I
  ---

  It will print the IP (IPv4) address of the Raspberry Pi:
   ```
      192.168.0.102
   ---

4. **Install Nginx with RTMP module:**

   ```bash
   sudo apt update
   sudo apt install nginx libnginx-mod-rtmp

   sudo systemctl start nginx

   sudo systemctl status nginx

   ```

If the nginx service is running without any similar I should get:

```bash

admin@gopro-now:~ $ sudo systemctl start nginx
admin@gopro-now:~ $ sudo systemctl status nginx
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: active (running) since Tue 2024-01-30 17:48:43 SAST; 36min ago
       Docs: man:nginx(8)
    Process: 2317 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 2318 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 2380 ExecReload=/usr/sbin/nginx -g daemon on; master_process on; -s reload (code=exited, status=0/SUCCESS)
   Main PID: 2346 (nginx)
      Tasks: 5 (limit: 8734)
        CPU: 85ms
     CGroup: /system.slice/nginx.service
             ├─2346 "nginx: master process /usr/sbin/nginx -g daemon on; master_process on;"
             ├─2381 "nginx: worker process"
             ├─2382 "nginx: worker process"
             ├─2383 "nginx: worker process"
             └─2384 "nginx: worker process"

Jan 30 17:48:43 gopro-now systemd[1]: Starting nginx.service - A high performance web server and a reverse proxy server...
Jan 30 17:48:43 gopro-now systemd[1]: Started nginx.service - A high performance web server and a reverse proxy server.
Jan 30 17:48:47 gopro-now systemd[1]: Reloading nginx.service - A high performance web server and a reverse proxy server...
Jan 30 17:48:47 gopro-now systemd[1]: Reloaded nginx.service - A high performance web server and a reverse proxy server.
Jan 30 17:48:47 gopro-now nginx[2380]: 2024/01/30 17:48:47 [notice] 2380#2380: signal process started
admin@gopro-now:~ $


```

5. **Configure Nginx:**
```diff

-______________________________________________________________________________________________

- MAKE A BACK-UP OF THE DEFAULT CONTENTS BEFORE MAKING ANY CHANGES TO NGINX.conf file BEFORE PROCEEDING!!!!!
-_____________________________________________________________________________________________

```

   Open the Nginx configuration file:
   ```bash
   sudo nano /etc/nginx/nginx.conf
   ```
   # Back-up of the original unmodified Nginx.conf file:

   ```nginx

user www-data;
worker_processes auto;
pid /run/nginx.pid;
error_log /var/log/nginx/error.log;
include /etc/nginx/modules-enabled/*.conf;

events {
        worker_connections 768;
        # multi_accept on;
}

http {

        ##
        # Basic Settings
        ##

        sendfile on;
        tcp_nopush on;
        types_hash_max_size 2048;
        # server_tokens off;

        # server_names_hash_bucket_size 64;
        # server_name_in_redirect off;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        ##
        # SSL Settings
        ##

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;

        ##
        # Logging Settings
        ##

        access_log /var/log/nginx/access.log;

        ##
        # Gzip Settings
        ##

        gzip on;

        # gzip_vary on;
        # gzip_proxied any;
        # gzip_comp_level 6;
        # gzip_buffers 16 8k;
        # gzip_http_version 1.1;
        # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

        ##
        # Virtual Host Configs
        ##

        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
}


#mail {
#       # See sample authentication script at:
#       # http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
#
#       # auth_http localhost/auth.php;
#       # pop3_capabilities "TOP" "USER";
#       # imap_capabilities "IMAP4rev1" "UIDPLUS";
#
#       server {
#               listen     localhost:110;
#               protocol   pop3;
#               proxy      on;
#       }
#
#       server {
#               listen     localhost:143;
#               protocol   imap;
#               proxy      on;
#       }
#}


```

   Add the following RTMP block within the `http` block:


```
It should look like this if done correctly:

```nginx

rtmp {
    server {
        listen 1935;
        chunk_size 4096;

        application live {
            live on;
            record off;
            allow publish 192.168.0.0/24;  # Restrict publishing to the local network
            deny publish all;
            allow play all;
            meta copy;
        }
    }
}

```

   Replace `YOUR_IP_ADDRESS` with the IP address you'll be streaming from. This restricts publishing to only your IP address.

   Before saving double check if there are no duplicates, no syntax errors and no-miss-alligned code as the structure is unison throughout the entire file this includes `Comments`, `perenthisis`,  `blocks` and its `children` with each other which will brake the Nginx Agent and ruin your entire days work if one ignored the red `WARNING` message found at the very beginning of `Step 5`. Don't have a shitfit and be thankful for making a mistake. 
   
   `"The greatest teacher, failure is."` 

   If done correctly the Nginx.conf file should look like this:

   ```nginx
user www-data;
worker_processes auto;
pid /run/nginx.pid;
error_log /var/log/nginx/error.log;
include /etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 768;
    # multi_accept on;
}

http {

    ##
    # Basic Settings
    ##

    sendfile on;
    tcp_nopush on;
    types_hash_max_size 2048;
    # server_tokens off;

    # server_names_hash_bucket_size 64;
    # server_name_in_redirect off;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    ##
    # SSL Settings
    ##

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
    ssl_prefer_server_ciphers on;

    ##
    # Logging Settings
    ##

    access_log /var/log/nginx/access.log;

    ##
    # Gzip Settings
    ##

    gzip on;

    # gzip_vary on;
    # gzip_proxied any;
    # gzip_comp_level 6;
    # gzip_buffers 16 8k;
    # gzip_http_version 1.1;
    # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;


    ##
    # RTMP Settings
    ##

    rtmp {
        server {
            listen 1935;
            chunk_size 4096;

            application live {
                live on;
                record off;
                allow publish 192.168.0.102;
                deny publish all;
                allow play all;
            }
        }
    }

    ##
    # Virtual Host Configs
    ##

    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}

#mail {
#       # See sample authentication script at:
#       # http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
#
#       # auth_http localhost/auth.php;
#       # pop3_capabilities "TOP" "USER";
#       # imap_capabilities "IMAP4rev1" "UIDPLUS";
#
#       server {
#               listen     localhost:110;
#               protocol   pop3;
#               proxy      on;
#       }
#
#       server {
#               listen     localhost:143;
#               protocol   imap;
#               proxy      on;
#       }
#}





   
   
   
   ```

5. **Restart Nginx:**

   ```bash
   sudo systemctl restart nginx
   ```

6. **Start streaming:**

   You can now stream to your Raspberry Pi's RTMP server. Use the following RTMP URL in your streaming software:

   ```
   rtmp://YOUR_RASPBERRY_PI_IP/live
   ```

   Replace `YOUR_RASPBERRY_PI_IP` with the IP address of your Raspberry Pi.

7. **Test your RTMP server:**

   Use a client like VLC to connect to your RTMP server and verify that you can stream to it.

Remember, this setup assumes your Raspberry Pi is directly accessible from the internet. Once you set up port forwarding via VPN and Dynamic DNS, ensure you update the IP address in the Nginx configuration accordingly. Also, consider additional security measures like setting up a firewall and using strong passwords.



Certainly! Below is a simplified version of a README file that outlines the process for setting up the GoPro Hero 11 with GoPro Lab firmware to stream to a Raspberry Pi running an Nginx RTMP server. Please note that this is a basic template, and you may want to customize it based on your specific details or add additional information.

---

# GoPro Hero 11 Streaming to Raspberry Pi with Nginx RTMP Server

## Overview

This guide provides step-by-step instructions for setting up a GoPro Hero 11 with GoPro Lab firmware to stream its video feed to a Raspberry Pi running an Nginx RTMP server. The Nginx RTMP server will then distribute the live stream to viewers' devices.

## Requirements

- GoPro Hero 11 with GoPro Lab firmware installed
- Raspberry Pi 4 running Raspberry Pi OS Bookworm 64-bit
- VLC or another RTMP-compatible client for viewing the stream

## Steps

### 1. Install Nginx and libnginx-mod-rtmp on Raspberry Pi

```bash
sudo apt update
sudo apt install nginx libnginx-mod-rtmp
```

### 2. Configure Nginx RTMP Server

Edit the Nginx configuration file:

```bash
sudo nano /etc/nginx/nginx.conf
```

Add the following lines at the end of the `http` block:

```nginx
rtmp {
    server {
        listen 1935;
        application live {
            live on;
            meta copy;
        }
    }
}
```

Restart Nginx:

```bash
sudo service nginx restart
```

### 3. Configure GoPro Hero 11

- Ensure GoPro Hero 11 is running GoPro Lab firmware.
- Access the GoPro's settings to enable streaming.
- Configure the GoPro to stream to the local IP address of the Raspberry Pi and specify a unique stream key.

### 4. Start Streaming

- Power on the GoPro Hero 11 and start the streaming feature.
- Verify that the GoPro is successfully streaming to the Raspberry Pi.

### 5. View the Stream Locally

- Open VLC or an RTMP-compatible client.
- Enter the following URL:

```bash
rtmp://<Raspberry_Pi_IP>/live/<Stream_Key>
```

Replace `<Raspberry_Pi_IP>` with the actual IP address of your Raspberry Pi and `<Stream_Key>` with the configured stream key on the GoPro.

## Additional Notes

- For security reasons, consider securing your RTMP server and network, especially if you plan to expose it to the internet.
- In the future, if you plan to allow external access, set up port forwarding on your router to forward port 1935 to the local IP address of your Raspberry Pi.

---

Feel free to customize this README based on your specific details or additional information you want to include. This is a basic template to help you get started.