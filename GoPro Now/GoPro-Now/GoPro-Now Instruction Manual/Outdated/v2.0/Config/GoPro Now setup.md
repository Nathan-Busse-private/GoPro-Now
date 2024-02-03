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

admin@gopro-now:~ $ sudo systemctl status nginx
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: active (running) since Thu 2024-02-01 00:09:43 SAST; 11h ago
       Docs: man:nginx(8)
    Process: 1019 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 1035 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
   Main PID: 1038 (nginx)
      Tasks: 5 (limit: 8734)
        CPU: 89ms
     CGroup: /system.slice/nginx.service
             ├─1038 "nginx: master process /usr/sbin/nginx -g daemon on; master_process on;"
             ├─1039 "nginx: worker process"
             ├─1040 "nginx: worker process"
             ├─1041 "nginx: worker process"
             └─1042 "nginx: worker process"

Feb 01 00:09:42 gopro-now systemd[1]: Starting nginx.service - A high performance web server and a reverse proxy server...
Feb 01 00:09:43 gopro-now systemd[1]: Started nginx.service - A high performance web server and a reverse proxy server.
admin@gopro-now:~ $

```

5. **Open and back-up Nginx.conf file:**


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

6. **Configure Nginx:**

After making the back-up, scroll to the very end of the conf file insert the  ```rtmp``` block

```
It should look like this if done correctly:

```nginx

rtmp {
    server {
        listen 1935;
        chunk_size 4096;

        application live {
            live on;
            interleave on;
            record off;
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
        # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text javascript;

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
rtmp {
    server {
        listen 1935;
        chunk_size 4096;

        application live {
            live on;
            record off;
            allow publish rtmp://192.168.0.102/live/127.0.0.1;
            deny publish all;
            allow play all;
            meta copy;
        }
    }
}
        
```

5. **Restart Nginx:**

   ```bash
   sudo service nginx restart

   sudo systemctl restart nginx

   sudo systemctl status nginx
   ```

6. **Generate RTMP address and stream key:**

    You can use a simple script or a program to generate unique RTMP addresses and stream keys. Below is an example using Python:

    ```python
    # Save this script as generate_key.py
    import random
    import string

    def generate_key(length=8):
        characters = string.ascii_letters + string.digits
        return ''.join(random.choice(characters) for i in range(length))

    rtmp_address = "rtmp://your_raspberry_pi_ip/live/"
    stream_key = generate_key()

    print(f"RTMP Address: {rtmp_address}{stream_key}")
    ```

    Run the script:

    ```bash
    python generate_key.py
    ```

    Note the RTMP address and stream key; you will provide these to your broadcasting device.

7. **Configure GoPro Hero 11 Black:**

    Configure your GoPro to stream to the RTMP address obtained from the Raspberry Pi.

8. **View the stream locally (VLC):**

    Open VLC and use the following network stream URL:

    ```bash
    rtmp://your_raspberry_pi_ip/live/stream_key
    ```

9. **Broadcast to YouTube (optional):**

    If you want to broadcast the stream to YouTube, use a tool like `ffmpeg` to push the stream to YouTube's RTMP server. Make sure to replace `your_stream_key` with your actual YouTube stream key.

    ```bash
    ffmpeg -i rtmp://your_raspberry_pi_ip/live/stream_key -c copy -f flv rtmp://a.rtmp.youtube.com/live2/your_stream_key
    ```

    Viewers can then watch the YouTube stream.

Remember to secure your Raspberry Pi, and if you plan to expose it to the internet in the future, implement appropriate security measures like password protection and possibly SSL. Additionally, note that the YouTube RTMP URL may change, so refer to YouTube's documentation for the latest information.

**Let's make some tweaks**
