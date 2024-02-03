To achieve your goal of setting up an Nginx-RTMP server with a web UI on a Raspberry Pi 4 running Raspberry Pi OS Bookworm 64-bit, you can follow the steps below. Please note that this guide assumes you have basic knowledge of working with the Raspberry Pi and the Linux command line.

### Step 1: Install Nginx with RTMP module

```bash
sudo apt update
sudo apt install nginx-extras libnginx-mod-rtmp
```

### Step 2: Configure Nginx-RTMP

Edit the Nginx configuration file to include the RTMP module settings:

```bash
sudo nano /etc/nginx/nginx.conf
```

Add the following lines at the end of the `http` block:

```nginx
rtmp {
    server {
        listen 1935;
        chunk_size 4000;

        application live {
            live on;
            record off;
        }
    }
}
```

### Step 3: Restart Nginx

```bash
sudo systemctl restart nginx
```

### Step 4: Install Nginx Web UI

Clone the Nginx Web UI repository and install the required dependencies:

```bash
sudo apt install git python3-pip
git clone https://github.com/sergey-dryabzhinsky/nginx-rtmp-module-web-control.git
cd nginx-rtmp-module-web-control
sudo pip3 install -r requirements.txt
```

### Step 5: Run the Web UI

```bash
sudo python3 nginx-rtmp-webui.py
```

The web UI should now be accessible at `http://your_raspberry_pi_ip:8080`.

### Step 6: Generate RTMPS address and Stream Key

Add a new stream in the web UI, and the RTMPS address and stream key will be generated. Note these values for later use.

### Step 7: Configure Firewall

Ensure that the firewall allows traffic on port 1935 and 8080:

```bash
sudo ufw allow 1935
sudo ufw allow 8080
sudo ufw enable
```

### Step 8: Port Forwarding

If you want to access the server publicly, configure port forwarding on your router for ports 1935 and 8080 to point to the Raspberry Pi's local IP address.

### Step 9: Broadcast from GoPro

Configure the GoPro to stream to the RTMPS address with the generated stream key.

### Step 10: VLC and YouTube Streaming

Viewers can use VLC to watch the stream using the following URL:

```
rtmp://your_public_ip:1935/live/stream_key
```

For YouTube streaming, use a streaming software that supports RTMP and configure it with the same RTMPS address and stream key.

### Note:

- Achieving sub-100 milliseconds latency may be challenging due to factors such as network conditions. Ensure a reliable and low-latency internet connection for optimal performance.

- Remember to secure your server by changing default passwords, enabling HTTPS, and implementing other security measures.

- This setup assumes your GoPro can stream to an RTMP server with a custom RTMPS address. Verify GoPro settings and consult GoPro documentation for accurate configuration.

This guide provides a basic setup, and you may need to fine-tune settings based on your specific requirements and network conditions.