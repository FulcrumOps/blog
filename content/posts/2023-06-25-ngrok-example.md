---
title: Using ngrok to Expose your Local Server to the Internet
date: 2023-06-25
author: Pete Emerson
tags: [development, ngrok]
toc: false
---

# ngrok Example

ngrok is an extremely easy way to expose your HTTP application to the internet. This is useful when doing development or wanting to share something with a colleague or a customer for a period of time. For developers bringing new projects to life, this is free.

This is a simple example of using ngrok with a Golang HTTP server.

This content is also available on my [blog](https://fulcrumops.com/blog).

## Sign up for an ngrok account

If you haven't already done so, sign up for a free [ngrok account](https://ngrok.com/).

## Download ngrok

Download ngrok [here](https://dashboard.ngrok.com/get-started/setup). Unzip it and move it to somewhere in your path.

## Authenticate ngrok

Prior to using ngrok, you'll need to authorize it.

Your authorization token can be retrieved from ngrok's site [here](https://dashboard.ngrok.com/get-started/your-authtoken).

In your terminal, add your auth token to your ngrok configuration:

```
$ ngrok config add-authtoken YOUR_TOKEN_FROM_NGROK_HERE
```

## Build the web server

Clone [my code repository](https://github.com/FulcrumOps/ngrok-example) and `cd` into it.

```
$ git clone https://github.com/FulcrumOps/ngrok-example.git
$ cd ngrok-example
```

In this repository, get the dependencies you need and build the simple server binary.

```
$ go get
$ go build -o bin/server
```

## Run the server

```
$ bin/server
Starting server on port 8000 ...
```

## In a separate terminal, see the server running:

```
$ curl localhost:8000/foo/bar
Endpoint requested: /foo/bar
```

## Now, fire up ngrok and point to the server:

```
$ ngrok http 8000
ngrok                                                                        

Session Status                online
Account                       Pete Emerson (Plan: Free)
Version                       3.3.1
Region                        United States (us)
Latency                       39ms
Web Interface                 http://127.0.0.1:4040
Forwarding                    https://d6b3-208-65-167-238.ngrok-free.app -> http://localhost:8000
```

Note the Forwarding URL (https://d6b3-208-65-167-238.ngrok-free.app). We can now use that endpoint from anywhere on the internet to access your server.

In a fresh terminal you can curl your server via the external endpoint, showing that your server is indeed available to the world:

```
 $ curl -s 'https://d6b3-208-65-167-238.ngrok-free.app/foo/bar'
Endpoint requested: /foo/bar
```
