# Installing the Torrust Index
> The Torrust Index requires a running Torrust Tracker.
## Global Prerequisites
- [Git](https://git-scm.com) - Version Control.
- [cURL](https://curl.se/) - Command line tool and library for transferring data with URLs.
- [Rust/Cargo](https://www.rust-lang.org/) - Compiler toolchain & Package Manager (cargo).

## Install Prerequisites
The frontend can't run on its own and needs and external webserver like Apache or NGINX.
In this guide we will be using Nginx.

* Node (version >= 12.0.0)
    * for Debian/Ubuntu:
```bash
curl -fsSL https://deb.nodesource.com/setup_12.x | bash -
apt-get install -y nodejs
```
* NPM
    * for Debian/Ubuntu: ```sudo apt-get install npm```
* NGINX:
    * for Debian/Ubuntu: ```sudo apt install nginx```
    * for other distros: see [Nginx installation tutorial](https://www.nginx.com/resources/wiki/start/topics/tutorials/install/)


## Installing the backend
1\. Create the torrust install directory (if you haven't already) and clone the repo:
```bash
mkdir /opt/torrust
cd /opt/torrust
git clone https://github.com/torrust/torrust.git
```

2\. Change to the backend directory and create a file called: `.env`:
```bash
cd torrust/backend
echo "DATABASE_URL=sqlite://data.db?mode=rwc" > .env
```

3\. Then we have to create the SQLite database and run the migrations. Install the `sqlx-cli` and create the database:
```bash
cargo install sqlx-cli
sqlx db setup
```

4\. Now build the backend:
```bash
cargo build --release
```

5\. Run the backend once to generate the config.toml file:
```bash
cd /opt/torrust/torrust/backend
./target/release/torrust
```

6\. Then edit the `config.toml` and change at least the following keys:
```bash
nano config.toml
```

`[tracker]`

- `url`: Set to a connection string for the tracker. Eg: `udp://TRACKER_IP:6969`.
- `api_url`: Set to tracker api URL. Default: `http://localhost:1212`.
- `token`: Set this to an access token from the Torrust Tracker config.toml.

`[net]`

- `port`: Set to `3000` for this guide. If you choose another port make sure to change the Nginx config as well.

`[auth]`

- `secret_key`: Set to a __SECURE__ randomly generated string.

## Running the backend
1\. Run the backend using:
```bash
cd /opt/torrust/torrust/backend
./target/release/torrust
```

## Running the backend using Tmux
1\. Run the backend using Tmux:
```bash
tmux new -s torrust-index
cd /opt/torrust/torrust/backend
./target/release/torrust
```
> Press `CTRL+B D` to exit the tmux session without killing it.

## Installing the frontend
1\. Start with creating a file called '.env':
> Make sure to change YOUR_DOMAIN.
```bash
cd /opt/torrust/torrust/frontend
echo "VITE_API_BASE_URL=https://YOUR_DOMAIN/api" > .env
```

2\. Build the frontend:
```bash
npm i
npm run build
```
After this command successfully completed, a built version of the frontend is in the `dist` folder.
These files will be served by Nginx in the next steps.

### NGINX Configuration
1\. Create a file in `/etc/nginx/sites-available/` called `torrust.conf` with the following contents:
> Make sure to change YOUR_DOMAIN x2, CERT_PATH and CERT_KEY_PATH.
```nginx
server {
    listen 80;
    server_name YOUR_DOMAIN;
    
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name YOUR_DOMAIN;
    
    ssl_certificate CERT_PATH;
    ssl_certificate_key CERT_KEY_PATH;

    root /opt/torrust/torrust/frontend/dist/;
    location / {
        try_files $uri $uri/ /index.html;
    }
    
    location /api/ {
        proxy_pass https://127.0.0.1:3000/;
    }
}
```

2\. Enable the configuration by making a symlink to the config in the `sites-enabled` directory:
```bash
ln -s /etc/nginx/sites-available/torrust.conf /etc/nginx/sites-enabled/
```

3\. After this you can test the validity of the config by executing `nginx -t`,
if the config is valid you can safely reload Nginx to make the new configuration active:
```bash
sudo systemctl reload nginx
```