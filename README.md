   # 🚀 2ndWeeksofCloudOps - 3 tier Application

✨This repository is created to learn and deploy  3-tier application on aws cloud. this project contain three layer Presentation, Application and database

## 🏠 Architecture
![Architecture of the application](architecture.gif)

## Tech stack

- React 
- Nodejs
- MySQL


**Note**: You should have nodejs installed on your system. [Node.js](https://nodejs.org/)

👉 let install dependency to run react application


```
1.Create rds database into private subnets
2.Create two private  servers in private subnets one is for frontend and another one is for backend
3.Create two TG and loadbalncers one is for frontend another one is for backend 
4.Create both loadblancers in public subnets only and loadbalancer type s internet facing only becuser internal loadbalncer not working for our project 
```
### ->>connect to backend server--
```
 git clone https://github.com/CloudTechDevOps/2nd10WeeksofCloudOps-main.git
   cd backend
```
 ### edit the .env file in bellow path if u dont have any .env file just create in below path
```
2nd10WeeksofCloudOps-main.git/backend/.env

### add this mater
DB_HOST=book.rds.com	#change rds endpoint
DB_USERNAME=admin	#cahnge to nyour rds user name 
DB_PASSWORD="veera"   # change to your rds password
PORT=3306
```
```
yum install mariadb105-server
```
#### SSH into backend server and then run test.sql script from backend to create tables and records 
```
mysql -h book.rds.com -u admin -p<password> < test.sql
```

### Backend deploy process ###
```
sudo dnf install -y nodejs

cd backend

npm install

npm install dotenv

npm install -g pm2

pm2 start index.js --name node-app

pm2 startup

sudo systemctl enable pm2-root

pm2 save
```
#### after that create backend tg and loadbalncer and check your loadbalncer is giving hello response or not 


# ---------------------------------- FrontEnd---------------------------------------

 ### Frontend deploy process ###
```
git clone https://github.com/CloudTechDevOps/2nd10WeeksofCloudOps-main.git
cd client 
```
##### edit the config.js
```
vi client/src/pages/config.js
  
const API_BASE_URL = "http://api.narni.co.in";
 ````
in above line change to your backend loadbalncer url
const API_BASE_URL = "http://backend-loadbalancer-url";
```
sudo dnf install -y nodejs
sudo yum install nginx
sudo systemctl start nginx
sudo systemctl enable nginx
```
### then go to client directory 
### run below commands

### ****(Use npm run build:
### When preparing the app for deployment (e.g., to a server or hosting service like AWS, Netlify, or Vercel).
### Use npm start:
### During development or to start the app in production (for backend apps).)*****
```
npm install 
npm run build
sudo cp -r build/* /usr/share/nginx/html
```
# your frontend part is completed 

### #### after that create frontend tg and loadbalncer and check your loadbalncer is giving project output along with books 
# add the books 

# revrse proxy

### if you want to use internnal loadbalncer please go through below procress
# Reverse Proxy & Backend Setup (Amazon Linux)

This repository contains an Nginx reverse-proxy configuration that serves a React frontend and proxies API requests to a backend at private IP or loadbalancer.

.

## Files
- proxy.conf — Nginx server block (this file)

## Behavior summary (from `proxy.conf`)
- Listens on port 80.
- Routes requests starting with `/api/` to private ip or loadbalncer`.
  - Example: `GET /api/books` -> proxied to privateip or loadbalncer
- Serves a React single-page app from `/usr/share/nginx/html` and falls back to `index.html` for client routes.


## Install Nginx on Amazon Linux 2
Run the following on the public-facing EC2 (nginx) instance.

### After cloneing your your config.js file url must be /api only
once chek in your config file below one is comented or uncomented if commented please uncoment and build the package
```
const API_BASE_URL = "/api";  // For reverse proxy it is mandatory so dont change

```
```bash
sudo yum update -y
sudo yum install -y nginx
# Enable and start nginx
sudo systemctl enable --now nginx
```
### create a proxy file and paste the file from git and chage the backend private ip if you are using internal loadbalncer change it 
```bash
sudo vi /etc/nginx/conf.d/reverse-proxy.conf
```
### paste below file and change loadbalancer url
```
server {
    listen 80;
    server_name _;

    # 🔥  API reverse proxy (WITH PATH FIX)
    location ^~ /api/ {
        proxy_pass http://backend-loadbalncer-url/;
        proxy_http_version 1.1;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # React build
    root /usr/share/nginx/html;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```
# Verify
2. Test and reload nginx:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

## Deploy React build to Nginx
On the machine where you build the React app (or directly on the nginx server):

```bash
# build (on your dev machine or CI)
npm install
npm run build

# Copy the build files to nginx root on the nginx host
sudo rm -rf /usr/share/nginx/html/*
sudo cp -r build/* /usr/share/nginx/html/

# reload nginx
sudo systemctl reload nginx
```

If your React app expects the app to be served at `/` this config will work as-is because the `location /` block uses `try_files` to return `index.html` for client routes.

## Sample Backend Setup (Amazon Linux) — Node.js / Express
These instructions create a minimal API that listens on port 80 and matches the `proxy_pass` target.




**Thank you so much for reading..😅**
