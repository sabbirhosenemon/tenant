# For use in /etc/nginx/sites-available/default

# This directive redirects all(All is denoted by a dot prefix on the domain) HTTP requests of gscsc.cc and *.gscsc.cc to their HTTPS versions respectively.
server {
  listen 80;
  listen [::]:80;
  server_name .gscsc.cc;

  return 301 https://$server_name$request_uri;
}

# This directive tells Nginx to use HTTP2 and SSL. And also proxy requests of https://gscsc.cc to a local Node.js app running on port 9000
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
  server_name gscsc.cc;

  ssl_certificate /etc/letsencrypt/live/gscsc.cc/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/gscsc.cc/privkey.pem;
  ssl_session_timeout 5m;

  location / {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-NginX-Proxy true;
    proxy_pass http://localhost:9000/;
    proxy_ssl_session_reuse off;
    proxy_set_header Host $http_host;
    proxy_cache_bypass $http_upgrade;
    proxy_redirect off;
  }
}

# This directive tells Nginx to use HTTP2 and SSL. And also proxy requests of wildcard *.gscsc.cc (first level subdomain of gscsc.cc) to a local Node.js app running on port 9000
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
  server_name *.gscsc.cc;

  ssl_certificate /etc/letsencrypt/live/gscsc.cc-0001/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/gscsc.cc-0001/privkey.pem;
  ssl_session_timeout 5m;

  location / {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-NginX-Proxy true;
    proxy_pass http://localhost:9000/;
    proxy_ssl_session_reuse off;
    proxy_set_header Host $http_host;
    proxy_cache_bypass $http_upgrade;
    proxy_redirect off;
  }
}

# after changing the file we have to restart nginx to apply the changes command: `sudo systemctl restart nginx`
# reference: https://blog.logrocket.com/how-to-run-a-node-js-server-with-nginx/
# full setup tutorial: https://blog.logrocket.com/how-to-build-web-app-with-multiple-subdomains-nginx/