This section explains how to create our ``nwatchcanada`` service user account which will be dedicated strictly to running our ``nwapp-back``.

1. Create a services user for the application:

    ```
    $ sudo groupadd --system nwatchcanada;
    $ sudo useradd --system --gid nwatchcanada --shell /bin/bash --home /opt/nwatchcanada nwatchcanada;

2. Create project home inside ``/opt``:

    ```
    $ sudo mkdir /opt/nwatchcanada
    ```

3. Give the permissions to the ``nwatchcanada`` user:

    ```
    $ sudo chown nwatchcanada:nwatchcanada /opt/nwatchcanada
    ```

4. Go into our new user. Note: ``sudo`` is to elevate privilege and ``su`` to switch users.

    ```
    $ sudo su - nwatchcanada
    ```


5. Setup our go folders.

    ```
    $ mkdir -p ~/go/{bin,pkg,src}
    ```

8. Open this file.

    ```
    vi ~/.bash_profile
    ```

9. Append the following:

    ```
    export GOROOT=/usr/local/go
    export GOBIN="$HOME/go/bin"
    export GOPATH="$HOME/go"
    ```

10. Apply your changes.

    ```
    source /etc/profile && source ~/.bash_profile
    ```
