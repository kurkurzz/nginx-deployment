# Deploy Nuxt Project (with SSR Disabled)

Since SSR is disable, this would be a static website. So we just need to point nginx to the path of the index.html (`nuxt-project/.output/public/index.html`)

1. Run `sudo apt-get install nodejs npm`

	Install nodejs and npm in the system, Verify installation by running `node -v` and `npm -v`.

1. Run `git pull`

	Fetch the latest commit from the remote origin.

1. Run `npm install`

	To ensure all dependencies are installed. Not required if there is no new dependencies needed to be installed.

1. Run `npm run generate`

	This will build nuxt project for static hosting (no SSR). It will export the files at `nuxt-project/.output/public/`.

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
			root /path/nuxt-project/.output/public;

			index index.html;

			server_name domain.com;

			location / {
				try_files $uri $uri/ =404;
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

	- Run `sudo certbot certonly --webroot -w /home/project/.output/public -d example.com`
	- Open nginx sites-available directory: `cd /etc/nginx/sites-available`
	- Create a new config file: `sudo nano <project_name>`
	- Insert the configuration and override the previous config. Sample:

		```
		server {
			listen 80;
			server_name yourdomain.com;
			return 301 https://$server_name$request_uri;
		}

		server {
			listen 443 ssl;
			server_name yourdomain.com;
			root /path/nuxt-project/.output/public;

			ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
			ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

			ssl_protocols TLSv1.2 TLSv1.3;
				ssl_ciphers EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH;
				ssl_prefer_server_ciphers on;

				# Logging
				access_log /var/log/nginx/access.log;
				error_log /var/log/nginx/error.log;

				# Index file and error pages
				index index.html;
			
			include /etc/nginx/mime.types;

			location / {
					try_files $uri $uri/ /index.html;
				}
		}
		```
	- Reload/restart nginx: `sudo systemctl restart nginx` or  `sudo systemctl reload nginx`



