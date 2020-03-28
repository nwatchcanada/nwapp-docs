# Setup NWApp Web-Application Frontend
The following instructions will set up the [nwapp-front](https://github.com/over55/nwapp-front) code. Please note, you must complete the [previous article](/2-Virtual-Hosts.md) before proceeding further.

Special thanks to the following article:

* https://hackernoon.com/start-to-finish-deploying-a-react-app-on-digitalocean-bcfae9e6d01b

## Instructions

1. Delete our previously created sample folder.

    ```
    sudo rm -Rf /var/www/nwapp.ws
    sudo rm -Rf /var/www/nwapp.ca
    ```

2. Updated permissions recursively down the folder.

    ```
    sudo chown -R $USER:$USER /var/www
    ```

2. Clone the brochure site from [github.com](https://github.com/mikaponics/mikaponics-brochure). Afterwords rename the file.

    ```
    cd /var/www
    git clone https://github.com/nwatchcanada/nwapp-front.git;
    mv nwapp-front nwapp.ca
    ```

3. Go into our directory and install the dependencies.

    ```
    cd /var/www/nwapp.ca
    sudo npm install
    ```

4. Create our ``.env.local`` file by copying the ``.env`` file.

    ```
    cp /var/www/nwapp.ca/.env /var/www/nwapp.ca/.env.local
    ```

5. Go inside the ``.env.local`` file and adjust the URL to the location of your API web-service.

    ```
    vi /var/www/nwapp.ca/.env.local
    ```

6. Copy and paste the following:


```
# DEPRECATED
REACT_APP_API_HOST=https://nwapp.ws

# Neighbourhood Watch Backend API Web-Service
REACT_APP_API_DOMAIN=nwapp.ws
REACT_APP_API_PROTOCOL=https

# Neigbhourhood Watch Frontend
REACT_APP_WWW_DOMAIN=nwapp.ca
REACT_APP_WWW_PROTOCOL=https
REACT_APP_IMAGE_UPLOAD_MAX_FILESIZE_IN_BYTES=10485760  # Note "1048576" bytes is "1 mega byte".
REACT_APP_IMAGE_UPLOAD_MAX_FILESIZE_ERROR_MESSAGE="File is too large. The maximum size is 10 MB."
```


6. Build our ``react`` app for production.

    ```
    sudo npm run build
    ```

7. Open up front-end ``nginx`` server block file.

    ```
    vi /etc/nginx/sites-available/nwapp.ca.conf
    ```

8. Replace the contents of the file to look as follows:

    ```
    server {
        listen  80;

        server_name nwapp.ca *.nwapp.ca;

        location / {
            add_header 'Access-Control-Allow-Origin' '*'; # Read: https://www.scalescale.com/tips/nginx/allow-cross-domain-ajax-requests-nginx/#
            root  /var/www/nwapp.ca/build;
            index  index.html index.htm;
            try_files $uri /index.html =404;
        }

        error_page  500 502 503 504  /50x.html;
        location = /50x.html {
            root  /usr/share/nginx/html;
        }
    }
    ```

9. Go ahead and restart ``nginx`` by typing:

    ```
    $ sudo systemctl restart nginx
    ```

10. Finally open up your browser and go to your ``http://nwapp.ca``.
