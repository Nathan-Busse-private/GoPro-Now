Setting up an Nginx-RTMP server with nginx-ui by 0xJacky on a Raspberry Pi 4 running Raspberry Pi OS Bookworm 64-bit requires a few steps. Please note that achieving near-zero latency over the internet might be challenging due to network conditions, but I'll guide you through the setup. Make sure your Raspberry Pi has a good internet connection.

1. **Install Required Packages:**

   ```bash
   sudo apt update
   sudo apt install nginx-extras libnginx-mod-rtmp git
   ```

2. **Install nginx-ui by 0xJacky:**

   ```bash
   git clone https://github.com/0xJacky/nginx-ui.git
   cd nginx-ui
   sudo bash install.sh
   ```

   Follow the instructions provided by the installer.

3. **Configure Nginx-RTMP:**

   Edit the Nginx-RTMP configuration file:

   ```bash
   sudo nano /etc/nginx/nginx.conf
   ```

   Add the following configuration within the `http` block:

   ```nginx
   include /etc/nginx/conf.d/*.conf;
   ```

   Create a new configuration file for Nginx-RTMP:

   ```bash
   sudo nano /etc/nginx/conf.d/rtmp.conf
   ```

   Add the following content:

   ```nginx
   rtmp {
       server {
           listen 1935;
           chunk_size 4096;

           application live {
               live on;
               record off;
               push rtmp://live-push-url/your-stream-key; # Set your streaming destination
           }
       }
   }
   ```

   Save the file.

4. **Generate RTMPS URL and Stream Key:**

   You can use a simple script for this purpose. Create a new file:

   ```bash
   nano ~/generate_rtmps.sh
   ```

   Add the following content:

   ```bash
   #!/bin/bash

   STREAM_KEY=$(openssl rand -hex 32)
   RTMPS_URL="rtmps://your-server-ip:1935/live"

   echo "RTMPS URL: $RTMPS_URL"
   echo "Stream Key: $STREAM_KEY"
   ```

   Save the file and make it executable:

   ```bash
   chmod +x ~/generate_rtmps.sh
   ```

   Run the script to generate the RTMPS URL and Stream Key:

   ```bash
   ~/generate_rtmps.sh
   ```

   Note: Replace `your-server-ip` with your Raspberry Pi's public IP address.

5. **Port Forwarding:**

   Set up port forwarding on your router to forward external traffic on port 1935 to your Raspberry Pi's local IP address.

6. **Access Nginx-UI:**

   Open a web browser and access nginx-ui by navigating to `http://your-server-ip:8080`. You can monitor the streams from here.

7. **Accessing the Stream:**

   - Use VLC to test the RTMP stream:

     ```bash
     vlc rtmp://your-server-ip:1935/live/your-stream-key
     ```

   - For YouTube streaming, set up a custom RTMP server in your YouTube live settings using the RTMPS URL and Stream Key generated earlier.

8. **Reducing Latency:**

   Achieving sub-100ms latency might be challenging due to the inherent latency in internet connections. However, using a low-latency setting in your Nginx-RTMP configuration can help:

   ```nginx
   application live {
       live on;
       record off;
       push rtmp://live-push-url/your-stream-key; # Set your streaming destination
       interleave on;
       wait_video off;
   }
   ```

   This configuration may introduce packet loss, so monitor the stream's quality carefully.

Remember to adapt the configurations to your specific needs, and ensure security measures are in place, especially when dealing with port forwarding and publicly accessible services.