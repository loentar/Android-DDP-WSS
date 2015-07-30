# Android-DDP-WSS
A fork of https://github.com/delight-im/Android-DDP to provide WSS support for meteor.

This project includes code from https://github.com/palmerc/SecureWebSockets.

## Usage

The Android-DDP's interface mailny leaved unchanged in exception of just one new constructor to set custom web socket options:

  	public Meteor(final Context context, final String serverUri, String protocolVersion, 
  	    WebSocketOptions webSocketOptions) {


To migrate your app to WSS you just need to:
  
  1. replace `Android-DDP.jar` with `Android-DDP-WSS.jar` in your app;
  
  2. change `ws://` to `wss://` in server URL.
  
Plain WS is supported by this library as well, of course.

## Run meteor with WSS

One of methods to start Meteor with WSS is to run it under reverse HTTPS proxy.

Example of nginx configuration to establish reverse proxy to Meteor:


    http {
      server {
      
        listen 8443;
        server_name localhost;
        
        ssl on;
        ssl_certificate /etc/ssl/certs/ssl-cert-snakeoil.pem;
        ssl_certificate_key /etc/ssl/private/ssl-cert-snakeoil.key;
        
        location / {
        
          # websockets
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection "upgrade";
          
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header Host $http_host;
          proxy_set_header X-NginX-Proxy true;
          
          proxy_pass http://localhost:3000/;
          proxy_redirect off;
        }
      }
    }


## How to use custom CA in your app:

1. Export CA from server
    openssl x509 -outform der -in /etc/ssl/certs/ssl-cert-snakeoil.pem -out ca.crt

2. Place generated `ca.crt` into `app/src/main/res/raw/ca.crt`

3. Create `Meteor` instance with custom options:

        String serverUrl = "wss://server:8443/websocket";
        
        WebSocketOptions options = null;
        try {
            InputStream streamCA = context.getResources().openRawResource(R.raw.ca);
            SocketFactory socketFactory = CertificateHelper.getSocketFactoryWithCustomCA(streamCA);
            options = new WebSocketOptions();
            options.setSSLCertificateSocketFactory(socketFactory);
        } catch (Exception e) {
            e.printStackTrace();
        }

        mMeteor = new Meteor(context, serverUrl, null, options);
