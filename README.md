# limited-time-link
A system of limited time link on nginx proxy

# wtf ? 
You want to give someone access for a limited time to a service behind a proxy ?

# get started
add in your nginx server proxy ->
```
server {
  listen *:443;

  server_name name.domain.com;

  ssl    on;
  ssl_certificate   /etc/letsencrypt/live/domain/fullchain.pem;
  ssl_certificate_key    /etc/letsencrypt/live/domain/privkey.pem;
  proxy_redirect          off;
  server_tokens off;
  proxy_set_header Host      $host;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header Upgrade "Websocket";
  proxy_set_header Connection "upgrade";

  # next steps here

}

```


this ->

```
  
  location ~ /1_generate-token_/.* {  rewrite /1_generate-token_/(.*) /$1  break  ; proxy_pass  https://127.0.0.1:9443; }

```

https://127.0.0.1:9443 is your service !


( optional, to get easily your token link ) add this ->
```
  location ~ /getLink/your-secret-address-to-get-time-link-like:26MFP045... {
    add_header Content-Type text/plain; 
    return 200 'your link ( valid until 2am ) \nhttps://name.domain.com/1_generate-token_/';
        
  }
```
( optional, protect your secret link with remote addr ip ) like this ->

```
  location ~ /getLink/your-secret-address-to-get-time-link-like:26MFP045... {
    if ($remote_addr = 999.999.999.999   ) {
      add_header Content-Type text/plain; 
      return 200 'your link ( valid until 2am ) \nhttps://name.domain.com/1_generate-token_/';
    }
  }

```
( very optional, you can redirect your secret link to your disposable link ) like this ->
```
  location ~ /goToLink/your-secret-address-to-get-time-link-like:26MFP045... {   
        return 301 https://name.domain.com/1_generate-token_/ ;  
  }
```

to change the token every day

create this script ->
```
sudo sed -i "s/1_[a-Z0-9]*_/1_$(hexdump -n 16 -v -e '/1 "%02X"' /dev/urandom)_/g" /etc/nginx/conf.d/myconf.conf ; sudo service nginx reload ; 
```
chmod +x token.sh

add in your cron.d/token your script with time desired( here: every day )

```
00 20 * * * debian /mydir/token.sh
```

End , get your disposable link in cron or via your secret link 

later I'll maybe encapsulate the nginx server in a docker container
