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
