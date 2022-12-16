---
title: HTTPs on localhost with Caddy
slug: https-on-localhost-with-caddy

publish_timestamp: Dec. 6, 2022
url: https://www.codingforentrepreneurs.com/blog/https-on-localhost-with-caddy/

---


Using HTTPs on localhost as well as custom domain names is made incredibly simple using [Caddy](https://caddyserver.com/). If you can run on localhost, you can use Caddy.

For example, if you have a web app that runs on:

`localhost:8080`

You can map Caddy to this address (as a reverse proxy) to enable local TLS/HTTPs such as `https://myapp.localhost`

You can use this with nearly any type of application such as:

- Django
- FastAPI
- Docker
- Express.js
- Ruby on Rails
- Static Websites
- Next.js
- Nginx
- Whatever

Pretty neat right? Let's get started.

## Install Caddy 

### Install Caddy on macOS

Using [homebrew](https://brew.sh), run:

Install caddy
```
brew install caddy
```
- In some tests, Java was required in order for caddy to work with Homebrew (`brew install java nss`)

### Install Caddy on Windows

1. Go to [Caddy's website](https://caddyserver.com/download)
2. Download the correct platform
3. Move the downloaded binary to a user folder
4. Run `.\path\to\my\binary\caddy_download version`


### Install Caddy on Linux

Open terminal and run:

```
sudo apt-get update && sudo apt-get install caddy -y
```
> You may need to install Java in order for this to run correctly.

### Verify Installation
In you command line, run:

```
caddy version
```
Which should respond with:
```
v2.6.2 <version-hash>
```


## Run an App Locally
For this, we'll run a Docker-based Nginx app:

```
docker run -p 8181:80 nginx
```
In this command I am binding my localhost port `8181` to the internal container port of `80` (the nginx default)


## Create Caddyfile

Create a `Caddyfile` with the contents:
```caddyfile

my-nginx.localhost {
    reverse_proxy localhost:8181
}
```
In this case, `my-nginx.localhost` will become `https://my-nginx.localhost` and map directly to our `localhost:8181` server. Pretty neat huh?


Since I'm using Docker, you can create the `Caddyfile` file anywhere. I recommend keeping the `Caddyfile` in the same directory as your project whenever possible.

## Run Caddy Server

If you're in the same working directory as the `Caddyfile` you can run:
```
caddy run 
```

Or just specify the path
```
caddy run --config path/to/Caddyfile
```

To stop the caddy server, just hit `CTRL` + `C` to cancel the program.

## Open Caddy-served Endpoint
In your command line type:
```
open http://my-nginx.localhost
```
Notice that when your webbrowser opens you should see the url being `https://my-nginx.localhost` with _no security issue warnings_!!

Pretty neat huh?
