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
http {
        ##
        # RTMP Configuration
        ##   

        rtmp {
            server {
                listen 1935;
                chunk_size 4096;

                application live {
                    live on;
                    record off;
                    allow publish YOUR_IP_ADDRESS;
                    deny publish all;
                    allow play all;
                }
            }
        }      
```

   Replace `YOUR_IP_ADDRESS` with the IP address you'll be streaming from. This restricts publishing to only your IP address.

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