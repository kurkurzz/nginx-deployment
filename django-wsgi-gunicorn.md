# Deploy Django WSGI application using Gunicorn and nginx

Gunicorn is used to run the django application itself. While nginx is serves as the proxy to forward request to the gunicorn and also to serves static files.

1. Run `git pull`

	Fetch the latest commit from the remote origin.

1. Create `conf/gunicorn_configuration.py`

	```
	# python env
	command = '/path/to/python/env/bin/gunicorn'
	# project directory
	pythonpath = '/path/to/your/django/project'
	bind = '0.0.0.0:8000'
	# small server spec so only 1
	workers = 1
	# long timeout because need to load model
	timeout = 600
	```
	Good to also include this file in the git.

1. Create python environment

	You can use python env or conda.

1. In the python env, run `pip install -r requirements.txt`

1. Run `pip install gunicorn`

1. Configure Gunicorn

	- Run `sudo nano /etc/systemd/system/gunicorn.service`
	- Insert the configuration. Sample:

		```
		[Unit]
		Description=gunicorn daemon
		After=network.target

		[Service]
		User=www-data
		Group=www-data
		WorkingDirectory=/path/to/your/django/project
		ExecStart=/path/to/python/env/bin/gunicorn -c conf/gunicorn_config.py <django_project>.wsgi

		[Install]
		WantedBy=multi-user.target

		```
		Setup it this way ensures that gunicorn will run this right away when the server reboot. All the configuration is specified in the `conf/gunicorn_config.py`

1. Reload system and enable gunicorn service

	- Run `sudo systemctl daemon-reload`
	- Run `sudo systemctl start gunicorn`
	- Run `sudo systemctl enable gunicorn`

1. Install and setup nginx

	- Install nginx: `sudo apt-get install nginx`
	- Start nginx: `sudo systemctl start nginx`
	- Verify nginx running: `sudo systemctl status nginx`

1. Configure nginx for your website

	- Open nginx sites-available directory: `cd /etc/nginx/sites-available`
	- Create a new config file: `sudo nano <project_name>`
	- Insert the configuration. Sample:

		```
		server {
			listen 80;
			listen [::]:80;

			server_name your_domain.com;

			location /static/ {
				root /path/to/your/django/project/;
			}

			location / {
			proxy_set_header Host $http_host;
				proxy_set_header X-Real-IP $remote_addr;
				proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
				proxy_set_header X-Forwarded-Proto $scheme;
				proxy_connect_timeout   300;
				proxy_send_timeout      300;
				proxy_read_timeout      300;
				include         uwsgi_params;
				proxy_pass http://127.0.0.1:8000;
			}
		}
		```

		Config above will only run the website on `http`.
	- Link the file into `site-enabled/` folder: `sudo ln -s /etc/nginx/sites-available/<project_name> /etc/nginx/sites-enabled/`
	- Ensure the config is correct: `sudo nginx -t`
	- Reload/restart nginx: `sudo systemctl restart nginx` or  `sudo systemctl reload nginx`

1. Configure SSL.

	- Install certbot `sudo snap install --classic certbot`

		This is used to create SSL key for us.

	- Run `sudo certbot certonly --webroot --webroot-path /var/www/html -d example.com`
	- Open nginx sites-available directory: `cd /etc/nginx/sites-available`
	- Create a new config file: `sudo nano <project_name>`
	- Insert the configuration and override the previous config. Sample:

		```
		server {
			listen 80;
			server_name your_domain.com;
			return 301 https://$server_name$request_uri;
		}

		server {
			listen 443 ssl;
			server_name your_domain.com;
			
			ssl_certificate /etc/letsencrypt/live/your_domain.com/fullchain.pem;
			ssl_certificate_key /etc/letsencrypt/live/your_domain.com/privkey.pem;

			ssl_protocols TLSv1.2 TLSv1.3;
			ssl_ciphers EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH;
			ssl_prefer_server_ciphers on;
		
			# Logging
			access_log /var/log/nginx/access.log;
			error_log /var/log/nginx/error.log;
			
			proxy_read_timeout 300;
			proxy_connect_timeout 300;
			proxy_send_timeout 300; 

			location /static/ {
				root /path/to/your/django/project/;
			}

			location / {
			proxy_set_header Host $http_host;
				proxy_set_header X-Real-IP $remote_addr;
				proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
				proxy_set_header X-Forwarded-Proto $scheme;
				proxy_connect_timeout   300;
				proxy_send_timeout      300;
				proxy_read_timeout      300;
				include         uwsgi_params;
				proxy_pass http://127.0.0.1:8000;
			}
		}
		```
	- Reload/restart nginx: `sudo systemctl restart nginx` or  `sudo systemctl reload nginx`



