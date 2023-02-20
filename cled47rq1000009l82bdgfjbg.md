# Go Introduction and Installation

Go is an open-source, statically typed, compiled programing language built by Google.
It combines the simplest of both statically typed and dynamically typed languages and provides you with the proper mixture of efficiency and simple programming. 
It is primarily fitted to building fast, efficient, and reliable server-side or system applications.

Following are some noted features of Go -

- Safety
- Concurrency
- Efficient Garbage Collection
- High-speed compilation
- Excellent Tooling support

## Installing GO

Go binary distributions are available for all major operating systems like Linux, Windows, and macOS. 
It’s easy to install Go from the binary distributions or <a href="https://golang.org/doc/install/source" style="color:DodgerBlue" target="_blank">you can try installing Go from source.</a>

### MacOS X

Install GO in MacOS using Homebrew.

```
brew install go
```

### Linux

- Download Go for Linux from Go’s official [download](https://golang.org/dl/) page.

    ```
    tar -xvf go1.18.linux-amd64.tar.gz
    ```

    It will create a directory named `go`.

- Move that `go` dir to `/usr/local` where all other binaries reside.

    ```
    sudo mv go /usr/local
    echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc # bashrc or zshrc depending upon your shell type
    ```

- Custom Installation directory

    Instead of moving `go` dir to `/usr/local` you can choose any other dir.

    ```
    mkdir custom_go # just for demo, you can move to any dir of your choice
    mv go custom_go
    ```

    Then set this custom location to `GOROOT` environment variable.

    ```
    export GOROOT=$HOME/go
    export PATH=$PATH:$GOROOT/bin
    ```
    and then test 

    ```
    go version
    go version go1.18 linux/amd64
    ```

    To make `GOROOT` and `PATH` permanent add them to `.bashrc` or `.zshrc` depending on which shell you are using or else your `GOROOT` will be unset once you end your terminal session or start a new terminal session.

### Windows

Download the Windows installer from Go’s official [download](https://golang.org/dl/) page. Open the installer and follow the on-screen instructions to install Go. 
By default, the installer installs Go in `C:\Go` and add `C:\go\bin` to your path environment variable.

***Thank you for reading this blog, and please give your feedback in the comment section below.***