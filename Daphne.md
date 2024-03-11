`apt install daphne`

### daphne.socket
```cd /etc/systemd/system/```
```sudo nano daphne.socket```
Add the following to this file:

```js
[Unit]
Description=daphne socket

[Socket]
ListenStream=/run/daphne.sock

[Install]
WantedBy=sockets.target
```

### daphne.service
```cd /etc/systemd/system/```
```sudo nano daphne.service```

The name of the file must be same as the .socket file except the file extension which should be '.service'.
Add the following content to this file:

```js
[Unit]
Description=daphne daemon
Requires=daphne.socket
After=network.target

[Service]
Type=simple
User=django
WorkingDirectory={project-root-directory}
ExecStart={path-to-virtual-environment}/bin/daphne -b 0.0.0.0 -p 8001 myproject.asgi:application

[Install]
WantedBy=multi-user.target
```

### NGINX

```js
upstream channels-backend {server localhost:8001;}

server {
    server_name domain.com www.domain.com;

    location = /favicon.ico { access_log off; log_not_found off; }

    location /static/ {
        root {path-to-peoject-folder};
    }

    location /media/ {
        root {path-to-peoject-folder};
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/run/gunicorn.sock;
    }

    location /ws/chat/ {
        proxy_pass http://channels-backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_redirect off;
    }
}
```

Some important things to keep in mind:
1. Make sure the ports in .service file and nginx file are the same. In this case, we have `8001`.
2. `/ws/chat/` is the path on which the websockets are hosted.

### Commands
Run the following commands whenever any of the daphne files are updated.
```
sudo systemctl daemon-reload
sudo systemctl restart daphne
```

To check the status of the service, run `sudo systemctl status daphne`

#### Related Articles:
1. https://okbaboularaoui.medium.com/how-to-set-up-django-with-postgres-nginx-and-daphne-django-channels-on-ubuntu-20-04-b0d24dcc7da9
