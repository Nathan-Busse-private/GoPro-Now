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
#	# See sample authentication script at:
#	# http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
#
#	# auth_http localhost/auth.php;
#	# pop3_capabilities "TOP" "USER";
#	# imap_capabilities "IMAP4rev1" "UIDPLUS";
#
#	server {
#		listen     localhost:110;
#		protocol   pop3;
#		proxy      on;
#	}
#
#	server {
#		listen     localhost:143;
#		protocol   imap;
#		proxy      on;
#	}
#}


```




To find the IPv6 address of your Raspberry Pi, you can use the `ifconfig` command or the newer `ip` command. Here are the steps using both methods:

### Method 1: Using `ifconfig`

Open a terminal and type:

```bash
ifconfig
```

Look for the section related to your network interface. Typically, it might be something like `eth0` for Ethernet or `wlan0` for Wi-Fi. The IPv6 address will be listed under the `inet6` section. For example:

```bash
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 2001:db8::1  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::c0ff:eeff:feff:abcd  prefixlen 64  scopeid 0x20<link>
        ...
```

In this example, `2001:db8::1` is the IPv6 address.

### Method 2: Using `ip`

Open a terminal and type:

```bash
ip -6 address
```

Look for the section related to your network interface, similar to the `ifconfig` method. The IPv6 address will be listed under the `inet6` section. For example:

```bash
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qlen 1000
    inet6 2001:db8::1/64 scope global
       valid_lft forever preferred_lft forever
    inet6 fe80::c0ff:eeff:feff:abcd/64 scope link
       valid_lft forever preferred_lft forever
...
```

In this example, `2001:db8::1` is the IPv6 address.

Choose the appropriate method based on your preference or the commands available on your Raspberry Pi. The IPv6 address will typically be listed in the output of either `ifconfig` or `ip` under the `inet6` section.




To set up an Nginx-RTMP server on your Raspberry Pi 4 running Raspberry Pi OS Bookworm 64-bit and enable broadcasting from a GoPro Hero 11 Black to clients outside the network, follow these steps:

1. **Install Nginx with RTMP module:**

```bash
sudo apt update
sudo apt install nginx-extras libnginx-mod-rtmp
```

2. **Configure Nginx:**

Edit the Nginx configuration file:

```bash
sudo nano /etc/nginx/nginx.conf
```

Add the following configuration block inside the `http` block:

```nginx
rtmp {
    server {
        listen 1935;
        chunk_size 4096;

        application live {
            live on;
            allow publish all;
            allow play all;
            push rtmp://gopro-now.theworkpc.com/live;
        }
    }
}
```

Replace `your_dynamic_dns` with your actual dynamic DNS hostname.

3. **Restart Nginx:**

```bash
sudo service nginx restart
```

4. **Set up port forwarding:**

In your router settings, forward port 1935 (RTMP default port) to the local IP address of your Raspberry Pi.

5. **Viewing the stream on VLC:**

On a client device, use VLC to view the stream. Open VLC and navigate to "Media" -> "Open Network Stream" and enter the following URL:

```
rtmp://your_dynamic_dns/live
```

Replace `your_dynamic_dns` with your actual dynamic DNS hostname.

6. **Broadcasting from GoPro:**

On your GoPro, go to the RTMP streaming settings and enter the RTMP server URL as follows:

```
rtmp://your_dynamic_dns/live
```

Replace `your_dynamic_dns` with your actual dynamic DNS hostname.

7. **Viewing on YouTube (optional):**

If you want to stream to YouTube, you'll need to configure YouTube Live Streaming settings and use the YouTube RTMP URL as the push destination in your Nginx configuration.

Note: Achieving "zero latency" is challenging, and some latency may still be present due to network conditions. Using a low-latency protocol like WebRTC might be more suitable for your requirements, but implementing it on a Raspberry Pi can be complex. Consider the trade-offs between latency and ease of setup.

