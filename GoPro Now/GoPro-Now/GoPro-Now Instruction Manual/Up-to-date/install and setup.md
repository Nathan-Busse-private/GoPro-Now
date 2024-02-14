rtmp {
    server {
        listen 1935;
        chunk_size 4096;
        interleave on;
        ack_window 2000;

        application live {
            live on;
            allow publish all;
            allow play all;
        }
    }
}
