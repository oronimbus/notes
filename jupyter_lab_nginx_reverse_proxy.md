## Setting up an NGINX reverse proxy for Jupyter Lab 1.0.x in Ubuntu 18.04

1) This requires sudo access rights so make sure you have that set up. Install jupyter lab and generate the config. The default path for this is typically `~/.jupyter/jupyter_notebook_config.py` :
```
pip3 install jupyterlab
jupyter notebook --generate-config
```

2) Edit the Jupyer Lab config via `sudo nano ~/.jupyter/jupyter_notebook_config.py`. I'm using the following settings, make sure you're using the correct `notebook_dir` and IP address. The base URL that we're using later for NGINX is given by `base_url`. I'm sticking to default port 8888 however it doesn't matter which one you're using:

```
c.NotebookApp.base_url = u'/lab'
c.NotebookApp.password =  'xxxx' 
c.NotebookApp.port = 8888
c.NotebookApp.port_retries = 50
c.NotebookApp.open_browser = False
c.NotebookApp.allow_origin = '*'
c.NotebookApp.ip = 'xxx.xx.xxx.xx'
c.NotebookApp.notebook_dir = u'/home/claudio/jupyter'
c.NotebookApp.allow_remote_access = True
```

3) The password can be generated via:
```
from notebook.auth import passwd 
passwd()
```
4) Install NGINX:

```
sudo apt-get update
sudo apt-get install nginx
```

5) I'm using the following config under `/etc/nginx/site-enabled/default`, but this depends on your webserver. The important features are the two server blocks. My website is using SSL certification generated via CertBot- this can be seen via the first server block listening on 443. Inside this block I've added the reverse proxy on the URI /lab which is listening to the path 8888. Finally, the second server block is used to reroute any request on port 80 (non-SSL) to 443 for my website:

```
upstream jupyter {
        server 127.0.0.1:8888;

}

server {

        listen 443 ssl;
        listen [::]:443 ssl;

        server_name claudio.work www.claudio.work;

        root /var/www/claudio.work/html;
        index index.html index.htm index.nginx-debian.html;

        ssl_certificate /etc/letsencrypt/live/claudio.work/fullchain.pem; # managed by Certbot
        ssl_certificate_key /etc/letsencrypt/live/claudio.work/privkey.pem; # managed by Certbot

        access_log /var/log/nginx/claudio.work.access.log;
        error_log /var/log/nginx/claudio.work.error.log;

        location / {

                try_files $uri $uri/ =404;
        }

        location /lab {
                 proxy_pass http://localhost:8888;
                 proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                 proxy_set_header X-Real-IP $remote_addr;
                 proxy_set_header Host $http_host;
                 proxy_http_version 1.1;
                 proxy_redirect off;
                 proxy_buffering off;
                 proxy_set_header Upgrade $http_upgrade;
                 proxy_set_header Connection "upgrade";
                 proxy_read_timeout 86400;
        }


}

server {

    if ($host = www.claudio.work) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    if ($host = claudio.work) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


    listen 80;
    listen [::]:80;

    server_name claudio.work  www.claudio.work;

    return 301 https://$server_name$request_uri;

}
```

6) Finally we can restart nginx using:

```
sudo service nginx restart
```

7) Finally, you would want to set up a systemd service to have jupyter lab running at all times. Create a new service using `touch /etc/systemd/system/jupyter.service`. The content can look like the below but it's important to get the `ExecStart`, i.e. the location of the executable right. Once done you can run `sudo systemctl enable jupyter.service` and then `sudo systemctl restart jupyter.service`:

```

[Unit]
Description=Jupyter Notebook

[Service]
Type=simple
PIDFile=/run/jupyter.pid
ExecStart=/usr/local/bin/jupyter-lab --config=/home/claudio/.jupyter/jupyter_notebook_config.py
User=claudio
WorkingDirectory=/home/claudio/jupyter
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```



### References:
- [Installing Jupyter Lab on Ubuntu 18.04](https://www.ceos3c.com/open-source/install-jupyterlab-on-ubuntu-18-04/)
- [Setting Up a NGINX Reverse Proxy](http://www.albertauyeung.com/post/setup-jupyter-nginx-supervisor/)
- [Creating a systemd Service](https://forums.fast.ai/t/run-jupyter-notebook-on-system-boot/749/5)