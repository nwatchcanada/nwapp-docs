# Installation

1. Start by moving into a writable directory:

        $ cd /tmp

2. Use the curl command and the link from Go to download the tarball:

        $ curl -LO https://dl.google.com/go/go1.12.7.linux-amd64.tar.gz

3. Confirm our download is valid. Run this command and make sure the output equals ``66d83bfb5a9ede000e33c6579a91a29e6b101829ad41fffb5c5bb6c900e109d9``:

        $ sha256sum go1.12*.tar.gz

4. Install go.

        $ sudo tar -C /usr/local -xvzf go1.12.7.linux-amd64.tar.gz

5. Setup our go folders.

        $ mkdir -p ~/go/{bin,pkg,src}

6. Enable `Go` in your paths.

        $ sudo vi /etc/profile.d/path.sh

7. Append the following to the file.

        export PATH=$PATH:/usr/local/go/bin

8. Open this file.

        $ sudo vi ~/.bash_profile

9. Append the following:

        export GOROOT=/usr/local/go
        export GOBIN="$HOME/go/bin"
        export GOPATH="$HOME/go"

10. Apply your changes.

        $ source /etc/profile && source ~/.bash_profile

# Usage

1. Create our new file.

    ```
    $ vi ~/go/src/hello.go
    ```

2. Set the following content:

    ```go
    package main

    import "fmt"

    func main() {
        fmt.Printf("Hello, World!\n")
    }
    ```


3. Run our installation.

    ```
    $ go install $GOPATH/src/hello.go
    ```

4. Run.

    ```
    $ $GOBIN/hello
    ```

# Notes

* The following instructions where adapted from  [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-go-1-7-on-centos-7).

* Also special thanks to [this article](https://linuxize.com/post/how-to-install-go-on-centos-7/).
