# Deploy React Capstone on DigitalOcean

```sh
sudo apt-get install pm2 serve json-server
```


## Create Subdomains

1. Log into DigitalOcean.
1. Click on `Networking` in the navigation.
1. Click on your domain.

### Create a Application Subdomain

1. In the `Hostname` field, enter in a subdomain. It is suggested that you make it the name of your capstone application.
1. Click in the `Will Direct To` input field and choose your droplet.
1. Click the `Create Record` button.

### Create a API Subdomain

1. In the `Hostname` field, enter in the name of the subdomain. It is suggested that you enter `api`.
1. Click in the `Will Direct To` input field and choose your droplet.
1. Click the `Create Record` button.

### Build the Application

1. `ssh` to your Droplet.
1. Clone your application repository from Github using HTTPS.
1. Change directory to your application.
1. Run `npm run build` to compile your code. When complete, there will be a new `build` sub-directory that contains the static files.
1. `cd build`

## Configure nginx

### React Application

1. `cd /etc/nginx/sites-available`
1. `sudo vim capstone_app`
1. Copy the text below into Visual Studio Code
    ```nginx
    server {
        server_name mysubdomain.mydomain.com;
        access_log /var/log/nginx/capstone.log;

        location / {
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-NginX-Proxy true;
            include proxy_params;
            proxy_pass http://127.0.0.1:8080;
        }

    }
    ```
1. On the second line, replace `mysubdomain` with the subdomain that you created above. Then replace `mydomain` with the domain name that you purchased.
1. Now copy the entire configuration.
1. Back to the terminal window that is showing the vim editor.
1. Press `i` in vim, and then paste the configuration.
1. Press the escape key.
1. Type `:x` and press enter.
1. Run `sudo ln -s /etc/nginx/sites-available/capstone_app /etc/nginx/sites-enabled`

### Capstone API

If you used `json-server` as the database for your application, follow these steps.

1. `cd /etc/nginx/sites-available`
1. `sudo vim capstone_api`
1. Copy the text below into Visual Studio Code
    ```nginx
        server {
            server_name api.mydomain.com;
            access_log /var/log/nginx/capstoneapi.log;

            location / {
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-NginX-Proxy true;
                include proxy_params;
                proxy_pass http://127.0.0.1:8088;
            }

        }
    ```
1. On the second line, replace `mydomain` with the domain name that you purchased.
1. Now copy the entire configuration.
1. Back to the terminal window that is showing the vim editor.
1. Press `i` in vim, and then paste the configuration.
1. Press the escape key.
1. Type `:x` and press enter.
1. Run `sudo ln -s /etc/nginx/sites-available/capstone_api /etc/nginx/sites-enabled`

## Serving your Application

### Configuration of pm2

Make sure you are still in the `build` directory. Run the following command.

```sh
pm2 startup
```

You should be given a command to run that looks like this.

```sh
[PM2] You have to run this command as root. Execute the following command:
      sudo su -c "env PATH=$PATH:/home/unitech/.nvm/versions/node/v4.3/bin pm2 startup <distribution> -u <user> --hp <home-path>
```

Copy the command from your terminal, not the one above and run it.

### Start the application process

```sh
pm2 start serve --name capstone --log capstone.log -- -l 8080
```

### Start the API

Change to your `api` directory and create a server file.

```sh
cd ../api
touch server.js
echo 'const jsonServer = require('json-server')
const server = jsonServer.create()
const router = jsonServer.router('database.json')
const middlewares = jsonServer.defaults()
const port = 8088

server.use(middlewares)
server.use(router)

server.listen(port, () => {
    console.log('JSON Server is running')
})' >> server.js
```

Then start `json-server` for your API.

```sh
pm2 start server.js --name api --log api.log
```

### Save your startup list

Run the following command to save the list of items you want to run on system startup.

```sh
pm2 save
```