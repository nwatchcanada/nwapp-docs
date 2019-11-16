# Setup NWApp Web-Application Frontend
The following instructions will set up the [nwapp-front](https://github.com/nwatchcanada/nwapp-front) code. Please note, you must complete the [previous article](/2-Virtual-Hosts.md) before proceeding further.

Special thanks to the following article:

* https://hackernoon.com/start-to-finish-deploying-a-react-app-on-digitalocean-bcfae9e6d01b

## Instructions

1. Delete our previously created sample folder.

    ```
    rm -Rf /var/www/nwapp.ca
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


    REACT_APP_API_HOST=https://nwapp.ca
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

        server_name *.nwapp.ca;

        location / {
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
