To create a dedicated RTMP server using nginx and the nginx-rtmp-module on a Raspberry Pi 4 running Raspberry Pi OS Bookworm Desktop 64-bit, you can follow these steps:

1. **Install Required Packages:**
   Make sure your system is up to date and install necessary packages.

```bash
sudo apt update
sudo apt upgrade
sudo apt install build-essential libpcre3 libpcre3-dev libssl-dev unzip
```

2. **Download nginx and nginx-rtmp-module:**
   Download nginx and the nginx-rtmp-module source code.

```bash
mkdir ~/nginx
cd ~/nginx
wget http://nginx.org/download/nginx-1.21.6.tar.gz
tar -zxvf nginx-1.21.6.tar.gz
git clone https://github.com/sergey-dryabzhinsky/nginx-rtmp-module.git
```

3. **Compile nginx with nginx-rtmp-module:**
   Compile nginx with the nginx-rtmp-module.

```bash
cd nginx-1.21.6
./configure --add-module=../nginx-rtmp-module --with-http_ssl_module
make
sudo make install
```

4. **Configure nginx:**
   Create and edit the nginx configuration file.

```bash
sudo nano /usr/local/nginx/conf/nginx.conf
```

   Add the following configuration to the `nginx.conf` file:

```nginx

rtmp {
    server {
        listen 1935;
        chunk_size 4000;

        application mytv {
            live on;
            record all;
            record_path /tmp/av;
            record_max_size 1K;
            record_unique on;
            allow publish 127.0.0.1;
            deny publish all;
        }

        application big {
            live on;
            exec ffmpeg -re -i rtmp://localhost:1935/$app/$name -vcodec flv -acodec copy -s 32x32 -f flv rtmp://localhost:1935/small/${name};
        }

        application small {
            live on;
        }

        application webcam {
            live on;
            exec_static ffmpeg -f video4linux2 -i /dev/video0 -c:v libx264 -an -f flv rtmp://localhost:1935/webcam/mystream;
        }

        application mypush {
            live on;
            push rtmp1.example.com;
            push rtmp2.example.com:1934;
        }

        application mypull {
            live on;
            pull rtmp://rtmp3.example.com pageUrl=www.example.com/index.html;
        }

        application mystaticpull {
            live on;
            pull rtmp://rtmp4.example.com pageUrl=www.example.com/index.html name=mystream static;
        }

        application vod {
            play /var/flvs;
        }

        application vod2 {
            play /var/mp4s;
        }

        application videochat {
            live on;
            on_publish http://localhost:8080/publish;
            on_play http://localhost:8080/play;
            on_done http://localhost:8080/done;
            record keyframes;
            record_path /tmp/vc;
            record_max_frames 10;
            record_interval 2m;
            on_record_done http://localhost:8080/record_done;
        }

        application hls {
            live on;
            hls on;
            hls_path /tmp/hls;
        }

        application dash {
            live on;
            dash on;
            dash_path /tmp/dash;
        }
    }
}

http {
    server {
        listen 8080;

        location /stat {
            rtmp_stat all;
            rtmp_stat_stylesheet stat.xsl;
        }

        location /stat.xsl {
            root /path/to/stat.xsl/;
        }

        location /hls {
            types {
                application/vnd.apple.mpegurl m3u8;
                video/mp2t ts;
            }
            root /tmp;
            add_header Cache-Control no-cache;
        }

        location /dash {
            root /tmp;
            add_header Cache-Control no-cache;
        }
    }
}



```

5. **Start nginx:**
   Start the nginx service.

```bash
sudo /usr/local/nginx/sbin/nginx
```

6. **Configure GoPro Camera:**
   Configure your GoPro camera to stream to `rtmp://<your-raspberry-pi-ip>:1935/live`.

7. **View the Stream:**
   You can view the stream by accessing `http://<your-raspberry-pi-ip>:8080/stat` in a web browser.

This setup should create a dedicated RTMP server on your Raspberry Pi 4 using nginx with the nginx-rtmp-module, allowing GoPro cameras to stream to it. Adjust the configuration as needed for your specific requirements.

##Should look like this##:

```nginx

I Used the example from the ReadME.md file:


rtmp {

    server {

        listen 1935;

        chunk_size 4000;

        # TV mode: one publisher, many subscribers
        application mytv {

            # enable live streaming
            live on;

            # record first 1K of stream
            record all;
            record_path /tmp/av;
            record_max_size 1K;

            # append current timestamp to each flv
            record_unique on;

            # publish only from localhost
            allow publish 127.0.0.1;
            deny publish all;

            #allow play all;
        }

        # Transcoding (ffmpeg needed)
        application big {
            live on;

            # On every published stream run this command (ffmpeg)
            # with substitutions: $app/${app}, $name/${name} for application & stream name.
            #
            # This ffmpeg call receives stream from this application &
            # reduces the resolution down to 32x32. The stream is the published to
            # 'small' application (see below) under the same name.
            #
            # ffmpeg can do anything with the stream like video/audio
            # transcoding, resizing, altering container/codec params etc
            #
            # Multiple exec lines can be specified.

            exec ffmpeg -re -i rtmp://localhost:1935/$app/$name -vcodec flv -acodec copy -s 32x32
                        -f flv rtmp://localhost:1935/small/${name};
        }

        application small {
            live on;
            # Video with reduced resolution comes here from ffmpeg
        }

        application webcam {
            live on;

            # Stream from local webcam
            exec_static ffmpeg -f video4linux2 -i /dev/video0 -c:v libx264 -an
                               -f flv rtmp://localhost:1935/webcam/mystream;
        }

        application mypush {
            live on;

            # Every stream published here
            # is automatically pushed to
            # these two machines
            push rtmp1.example.com;
            push rtmp2.example.com:1934;
        }

        application mypull {
            live on;

            # Pull all streams from remote machine
            # and play locally
            pull rtmp://rtmp3.example.com pageUrl=www.example.com/index.html;
        }

        application mystaticpull {
            live on;

            # Static pull is started at nginx start
            pull rtmp://rtmp4.example.com pageUrl=www.example.com/index.html name=mystream static;
        }

        # video on demand
        application vod {
            play /var/flvs;
        }

        application vod2 {
            play /var/mp4s;
        }

        # Many publishers, many subscribers
        # no checks, no recording
        application videochat {

            live on;

            # The following notifications receive all
            # the session variables as well as
            # particular call arguments in HTTP POST
            # request

            # Make HTTP request & use HTTP retcode
            # to decide whether to allow publishing
            # from this connection or not
            on_publish http://localhost:8080/publish;

            # Same with playing
            on_play http://localhost:8080/play;

            # Publish/play end (repeats on disconnect)
            on_done http://localhost:8080/done;

            # All above mentioned notifications receive
            # standard connect() arguments as well as
            # play/publish ones. If any arguments are sent
            # with GET-style syntax to play & publish
            # these are also included.
            # Example URL:
            #   rtmp://localhost/myapp/mystream?a=b&c=d

            # record 10 video keyframes (no audio) every 2 minutes
            record keyframes;
            record_path /tmp/vc;
            record_max_frames 10;
            record_interval 2m;

            # Async notify about an flv recorded
            on_record_done http://localhost:8080/record_done;

        }


        # HLS

        # For HLS to work please create a directory in tmpfs (/tmp/hls here)
        # for the fragments. The directory contents is served via HTTP (see
        # http{} section in config)
        #
        # Incoming stream must be in H264/AAC. For iPhones use baseline H264
        # profile (see ffmpeg example).
        # This example creates RTMP stream from movie ready for HLS:
        #
        # ffmpeg -loglevel verbose -re -i movie.avi  -vcodec libx264
        #    -vprofile baseline -acodec libmp3lame -ar 44100 -ac 1
        #    -f flv rtmp://localhost:1935/hls/movie
        #
        # If you need to transcode live stream use 'exec' feature.
        #
        application hls {
            live on;
            hls on;
            hls_path /tmp/hls;
        }

        # MPEG-DASH is similar to HLS

        application dash {
            live on;
            dash on;
            dash_path /tmp/dash;
        }
    }
}

# HTTP can be used for accessing RTMP stats

http {

    server {

        listen 8080;

        # This URL provides RTMP statistics in XML
        location /stat {
            rtmp_stat all;

            # Use this stylesheet to view XML as web page
            # in browser
            rtmp_stat_stylesheet stat.xsl;
        }

        location /stat.xsl {
            # XML stylesheet to view RTMP stats.
            # Copy stat.xsl wherever you want
            # and put the full directory path here
            root /path/to/stat.xsl/;
        }

        location /hls {
            # Serve HLS fragments
            types {
                application/vnd.apple.mpegurl m3u8;
                video/mp2t ts;
            }
            root /tmp;
            add_header Cache-Control no-cache;
        }

        location /dash {
            # Serve DASH fragments
            root /tmp;
            add_header Cache-Control no-cache;
        }
    }
}




Modify it to achieve the best fast moving action captured in real-time by my gopro camera. The myself and my viewers must have the best gopro video streaming experience with very little latency from both ends. 

```

I get this:

```nginx

rtmp {

    server {

        listen 1935;

        # Increase chunk size for better performance
        chunk_size 4096;

        # Optimize for low latency and fast-moving action
        application mytv {
            live on;

            # Disable recording to reduce latency
            record off;

            # Optimize for low latency
            interleave on;

            # Increase keyframe interval for lower latency
            keyframe_interval 1s;

            # Optimize for fast-moving action
            sync 1ms;

            # Adjust video codec parameters for performance
            hls_nested on;
            hls_sync 100ms;
            hls_fragment 1s;
            hls_playlist_length 2s;
            hls_continuous on;
        }
    }
}

http {

    server {

        listen      8080;

        location /stat {
            rtmp_stat all;
            rtmp_stat_stylesheet stat.xsl;
        }

        location /stat.xsl {
            root /path/to/stat.xsl/;
        }

        location /hls {
            types {
                application/vnd.apple.mpegurl m3u8;
                video/mp2t ts;
            }
            root /tmp;
            add_header Cache-Control no-cache;
        }
    }
}

```