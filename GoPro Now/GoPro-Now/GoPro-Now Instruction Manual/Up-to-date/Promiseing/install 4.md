Setting up an nginx-rtmp server on a Raspberry Pi 4 running Raspberry Pi OS Bookworm 64-bit involves several steps. Follow the guide below to achieve your desired setup:

### 1. Initial Raspberry Pi Setup

#### a. Update and Upgrade
```bash
sudo apt update && sudo apt upgrade -y
```

#### b. Configure Raspberry Pi Settings
```bash
sudo raspi-config
```
- Expand Filesystem
- Change User Password (optional but recommended)
- Set the Hostname
- Set up Localisation Options (Timezone, Keyboard Layout, etc.)
- Enable SSH under Interface Options

#### c. Install Necessary Tools
```bash
sudo apt install vim git
```

### 2. Install Nginx and RTMP Module

#### a. Install Dependencies
```bash
sudo apt install build-essential libpcre3 libpcre3-dev libssl-dev zlib1g-dev
```

#### b. Clone Nginx with RTMP Module
```bash
git clone https://github.com/sergey-dryabzhinsky/nginx-rtmp-module.git
```

#### c. Download and Extract Nginx
```bash
wget https://nginx.org/download/nginx-1.21.5.tar.gz
tar -zxvf nginx-1.21.5.tar.gz
cd nginx-1.21.5
```

#### d. Configure and Compile Nginx
```bash
./configure --add-module=../nginx-rtmp-module --with-http_ssl_module
make
sudo make install
```

### 3. Configure Nginx-RTMP

#### a. Create Configuration Directory
```bash
sudo mkdir /usr/local/nginx
sudo mkdir /usr/local/nginx/conf
```

#### b. Create Nginx Configuration File
```bash
sudo vim /usr/local/nginx/conf/nginx.conf
```
Paste the following configuration:

```nginx
worker_processes 1;

events {
    worker_connections 1024;
}

rtmp {
    server {
        listen 1935;
        chunk_size 4096;

        application live {
            live on;
            record off;
        }
    }
}
```

### 4. Configure Port Forwarding

Set up port forwarding on your router to forward external connections on port 1935 to the internal IP address of your Raspberry Pi.

### 5. Dynamic DNS Setup

Configure a Dynamic DNS service to map your external IP to a domain name. Services like DuckDNS or No-IP can be used for this purpose.

### 6. Start Nginx-RTMP Server

```bash
sudo /usr/local/nginx/sbin/nginx
```

### 7. Test RTMP Streaming

On your GoPro, set up the RTMP streaming with the following URL:
```
rtmp://your_dynamic_dns/live/stream
```

### 8. Client Access

Clients can access the stream using VLC or any other RTMP-compatible player with the URL:
```
rtmp://your_dynamic_dns/live/stream
```

For YouTube streaming, you'll need to set up a custom RTMP server in your YouTube Live settings with the same URL.

### 9. Additional Recommendations

- Ensure that your router has a static external IP or use a service for dynamic DNS.
- Consider securing your server with a firewall and only allow necessary ports.
- Use a wired connection for both the Raspberry Pi and the GoPro for stability.
- Adjust the Nginx configuration for your specific requirements and network conditions.

Please note that achieving zero latency may be challenging due to network conditions, but the steps above should help minimize latency as much as possible.