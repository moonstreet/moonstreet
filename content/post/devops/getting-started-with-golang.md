---
title: "Getting started with Golang"
date: 2019-02-09T15:16:45+01:00
draft: false
tags: ["golang"]
categories: ["golang"]
author: "Jacqueline"
---

This is yet another getting started note. This time I am trying to code in Golang.  
Why, would you ask? 

Obviously because of its mascotte:  

![GitHub Logo](https://ih0.redbubble.net/image.324409663.0450/sticker,375x360-bg,ffffff.u4.png)

## Installing Golang

Download Golang here [https://golang.org/dl/](https://golang.org/dl/). I am on Ubuntu, so next step is to simply unpack the executable to `/usr/local`. 

```
sudo tar -C /usr/local -xzf go1.11.5.linux-amd64.tar.gz

```

Then add Go to your path in your profile (in my case it would be in ~/.zshrc)

```
export PATH=$PATH:/usr/local/go/bin
```

After exiting and starting the terminal you should be able to write your first Go code. Start with creating your workspace directory in $HOME/go. If you'd like to use a different directory, you will need to set the GOPATH environment variable.

Make a folder:

```sh
mkdir -p ~/go/src/hello
```

Create a file called hello.go:

```sh
cat > ~/go/src/hello/hello.go <<EOF
package main

import "fmt"

func main() {
	fmt.Printf("hello, world\n")
}
EOF
```

Explanation:

* The package "main" tells the Go compiler that the package should compile as an executable program instead of a shared library. It is our entrypoint. 
* The keyword "import" is used for importing a package. The package "fmt" is used for the function Println. The package “fmt” comes from the Go standard library. 

Now build it:

```sh
cd $HOME/go/src/hello
go build
```

Run it

```
./hello
hello, world
```

You can also run

```sh
go install hello
```

This will create a bin folder in your Go directory with the binary.
If the GOPATH is configured correctly then this will work from any folder in your machine.

So this is proof that Go is correctly installed on your computer.

You can now follow [a tour of Go](https://tour.golang.org/list) to get further aquainted with Go.
