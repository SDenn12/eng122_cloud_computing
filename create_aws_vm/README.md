## How to Setup AWS Application

![](image/diagram.png)

### 1. Create a VM on AWS

1. Select the Operating System
![](image/selectvm.PNG)

2. Select the Processing Power
![](image/select_instance.PNG)

3. Configure instance details
![](image/select_confinstane3.PNG)

4. Select the storage
![](image/selectstore4.PNG)

5. Attach tags
![](image/selecttag5.PNG)

6. Select which ports should be allowed to interact with the VM
![](image/selectfirewall6.PNG)

### 2. SSH Connect to VM

![](image/ssh_connect.PNG)

- In the location of the key enter the command at the bottom to SSH into the VM
 
### 3. Install dependencies

```
# updates ubuntu
sudo apt-get update
sudo apt-get upgrade -y

# nginx install
sudo apt-get install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx

# nodejs install
sudo apt-get purge nodejs npm
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt-get install -y nodejs
sudo apt-get install -y npm

# pm2 install
sudo npm install pm2 -g
```

### 4. Copy files from Local Host to Remote Host

- SCP Command (used to migrate from lh to gh) `scp -i [key_name] -r [file_source] [file_destination] ` 
- The file destination is formatted as @user:PUBLIC_DNS so in this example it uses `ubuntu@ec2-34-240-115-36.eu-west-1.compute.amazonaws.com`

### 5. Reconfigure NGINX to allow reverse proxy

1. `sudo nano /etc/nginx/sites-available/default`
2. Reconfigure the file as such
```
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/html;

        
        index index.html index.htm index.nginx-debian.html;

        server_name _;

        location / {
                proxy_pass http://localhost:3000;
        }

}
```
3. `sudo systemctl restart nginx`

### 6. Run NPM start and the app should be working
- Using the public IP address found,
![](image/public_address.PNG)
- The app should now be working
![](image/appworking.PNG)

## Database Extension

### Set up Database VM

- Same configuration as app vm, except for security protocols which look like 

![](image/security_prot_db.PNG)

### Install dependencies on DB VM.

```
# updates ubuntu
sudo apt-get update
sudo apt-get upgrade -y

# retrieves key from mongodb
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv D68FA50FEA312927
echo "deb https://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list

# updates ubuntu
sudo apt-get update
sudo apt-get upgrade -y

sudo apt-get install -y mongodb-org=3.2.20 mongodb-org-server=3.2.20 mongodb-org-shell=3.2.20 mongodb-org-mongos=3.2.20 mongodb-org-tools=3.2.20

# enables and starts mongodb
sudo systemctl restart mongod
sudo systemctl enable mongod
```

Now configure the mongod.conf file such that the `bind ip` is `0.0.0.0`

Reset mongod so the configuration can be made live `sudo systemctl restart mongod`

### Reconfiguring the App VM

- Set a environment variable called DB_HOST and set equal to `mongodb://IPv4_db_address/27017/posts`
- Run `node seed.js` in seeds directory (inside app directory)
- Run `npm start` to start the app