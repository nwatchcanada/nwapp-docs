https://www.scalescale.com/tips/nginx/how-to-configure-nginx-gzip-compression/#
https://www.techrepublic.com/article/how-to-configure-gzip-compression-with-nginx/
vi /etc/nginx/nginx.conf


If you are using Nginx web server, add this code to the nginx.conf configuration file inside the httpd {} block:

    ```
    # enable gzip compression
    gzip on;
    gzip_min_length  1100;
    gzip_buffers  4 32k;
    gzip_types    text/plain application/x-javascript text/xml text/css;
    gzip_vary on;
    # end gzip configuration
    ```


https://nixcp.com/tools/gzip-test/
