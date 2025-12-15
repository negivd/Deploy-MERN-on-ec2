# 🚀 Deploy MERN Application on AWS EC2 with Load Balancer, NGINX, SSL (Certbot) & Cloudflare

Repository:
https://github.com/negivd/Deploy-MERN-on-ec2.git

Domain:
vishwadeep.online (Purchased from Namecheap)

## 📁 Project Structure
```text
Deploy-MERN-on-ec2/
├── backend/
│   ├── index.js
│   ├── conn.js
│   ├── controllers/
│   ├── routes/
│   ├── .env
│   └── package.json
│
└── frontend/
    ├── src/
    │   ├── components/
    │   └── url.js
    ├── build/
    └── package.json
```

## 🧱 Backend EC2 Configuration (BE1 + BE2)

Backend EC2 Instances:

| Instance | Private IP | Name Tag |
|--------|------------|----------|
| BE1 | 10.0.10.162 | vishi-BE1 |
| BE2 | 10.0.11.64 | vishi-BE2 |

---

## 🛠 Backend Setup (Run on BOTH BE1 & BE2)

Install required packages:

sudo apt update -y  
sudo apt install -y nodejs npm git  

Clone repository:

git clone https://github.com/negivd/Deploy-MERN-on-ec2.git  
cd Deploy-MERN-on-ec2/backend  

Create .env file:

nano .env  

PORT=3001  
MONGO_URI=mongodb+srv://negivd_db_user:vishi1976@vishi-mern.6a8pb1d.mongodb.net/vishi-travelmemory  

Database: vishi-travelmemory  
Collection: trips  

Install dependencies:

npm install  

Start backend with PM2:

sudo npm install -g pm2  
pm2 start index.js --name backend  
pm2 startup  
pm2 save  

Verify backend:

pm2 logs backend  
sudo lsof -i -P -n | grep LISTEN  
curl http://localhost:3001/hello  

Expected Output:
Hello World!

---

## 🔁 Internal Backend Load Balancer

Internal Load Balancer DNS:
internal-vishi-backend-lb-533076128.us-west-1.elb.amazonaws.com

Target Group: backend-tg  
Port: 3001  
Protocol: HTTP  

Health Check:
Path: /hello  
Port: 3001  

Test from Frontend EC2:

curl http://internal-vishi-backend-lb-533076128.us-west-1.elb.amazonaws.com/hello  

Expected Output:
Hello World!

---

## 🌐 Frontend EC2 Configuration (FE1 + FE2)

| Instance | Public IP |
|-------|-----------|
| FE1 | 13.56.248.29 |
| FE2 | 54.151.39.113 |

---

## 🛠 Frontend Setup (Run on BOTH FE1 & FE2)

Install packages:

sudo apt update -y  
sudo apt install -y nodejs npm git nginx  

Clone repository:

git clone https://github.com/negivd/Deploy-MERN-on-ec2.git  
cd Deploy-MERN-on-ec2/frontend  

Update API base URL:

nano src/url.js  

export const baseUrl = "/api";

Install dependencies:

rm -rf node_modules package-lock.json  
npm install  

Build React app:

npm run build  

Fix permissions:

sudo chown -R www-data:www-data build  
sudo chmod -R 755 build  

---

## 🔀 NGINX Reverse Proxy Configuration

Create config file:

sudo nano /etc/nginx/sites-available/frontend  

NGINX configuration:

server {
    listen 80;

    root /home/ubuntu/Deploy-MERN-on-ec2/frontend/build;
    index index.html index.htm;

    location /api/ {
        proxy_pass http://internal-vishi-backend-lb-533076128.us-west-1.elb.amazonaws.com;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location / {
        try_files $uri /index.html;
    }
}

Enable configuration:

sudo rm /etc/nginx/sites-enabled/default  
sudo ln -s /etc/nginx/sites-available/frontend /etc/nginx/sites-enabled/frontend  
sudo nginx -t  
sudo systemctl restart nginx  

Test API:

curl http://localhost/api/trip  

Expected Output:
[]

---

## 🌐 Frontend Public Load Balancer

Public LB DNS:
vishi-frontend-lb-1476995213.us-west-1.elb.amazonaws.com

Target Group: frontend-tg  
Port: 80  
Protocol: HTTP  

Test:

curl http://vishi-frontend-lb-1476995213.us-west-1.elb.amazonaws.com  

---

## 🍃 MongoDB Atlas Configuration

Cluster:
vishi-mern.6a8pb1d.mongodb.net

Database:
vishi-travelmemory

Collection:
trips

Connection String:

mongodb+srv://negivd_db_user:vishi1976@vishi-mern.6a8pb1d.mongodb.net/vishi-travelmemory

IP Whitelist:

0.0.0.0/0

---

## 📌 Backend API Endpoints

GET /trip  
GET /trip/:id  
POST /trip  
GET /hello  

Proxied via frontend:

/api/trip  
/api/trip/:id  
/api/hello  

---

## 🧪 End-to-End Testing

From Frontend EC2:

curl http://localhost  
curl http://localhost/api/trip  

Browser URLs:

http://13.56.248.29  
http://54.151.39.113  
http://vishi-frontend-lb-1476995213.us-west-1.elb.amazonaws.com  

Backend via Internal LB:

http://internal-vishi-backend-lb-533076128.us-west-1.elb.amazonaws.com/hello  

---

## 🔐 SSL Setup Using Certbot

Install Certbot:

sudo apt install -y certbot python3-certbot-nginx  

Generate SSL Certificate:

sudo certbot --nginx -d vishwadeep.online -d www.vishwadeep.online  

Test Auto Renewal:

sudo certbot renew --dry-run  

---

## ☁️ Cloudflare Configuration (Domain: vishwadeep.online)

Add domain to Cloudflare and select Free plan

Update Namecheap nameservers with Cloudflare NS records

DNS Records:

| Type | Name | Value | Proxy |
|----|----|-----|------|
| A | vishwadeep.online | vishi-frontend-lb-1476995213.us-west-1.elb.amazonaws.com | Proxied |
| CNAME | www | vishwadeep.online | Proxied |

Cloudflare SSL/TLS:

Mode: Full (Strict)  
Enable: Always HTTPS  

Enable:
WAF  
Auto Minify  
Brotli Compression  

---

## ✅ Final Architecture Summary

High Availability MERN Application  
Private Backend Load Balancer  
NGINX Reverse Proxy  
SSL with Certbot  
Cloudflare CDN + WAF  
Production Ready AWS Setup  

---

## 👨‍💻 Author

Vishwadeep Negi  
AWS | DevOps | MERN Deployment
