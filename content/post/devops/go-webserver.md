---
title: "Go Webserver"
date: 2019-02-10T09:43:32+01:00
draft: false
tags: ["golang"]
categories: ["golang"]
---

The last Golang post was a little bit lame so let's do something more interesting now. Let's create a webserver.

To create a simple webserver, we need:

* a function that can serve a page in browser (using http obviously)
* it should listen at a port
* it should respond to incoming requests
* it should be able to write content to the page




First create a new project structure:

```sh
mkdir -p ~/go/src/httpserver
```
Then create this file:

```sh
cat > ~/go/src/httpserver/main.go <<EOF
package main

import "net/http"

func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		w.Write([]byte("<h1>Go webserver</h1>"))
	})
	http.ListenAndServe(":8000", nil)
}
EOF
```

We now execute:

```sh
cd ~/go/src/httpserver
go build main.go
./main
```

Then browse to localhost:8000.

Result:

![](/uploads/gowebserver.png)




