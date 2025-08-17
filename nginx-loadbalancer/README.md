
# 📦 Project 03: NGINX as a Load Balancer

## ✅ Objective
To deploy a load-balanced architecture using NGINX that distributes traffic across multiple Node.js backend servers.

---

![Project Output](/images/project-03-nginx-load-balancer.png)

## ✅ Step-by-Step Implementation

### 1. 🟢 Launch and Connect to EC2 Instance
- Launch an Ubuntu EC2 instance from AWS Management Console.
- SSH into the instance:
```bash
ssh -i <your-key.pem> ubuntu@<public-ip-address>
```

### 2. 🔄 Update System Packages
```bash
sudo apt update
```

### 3. 🌐 Install NGINX
```bash
sudo apt install nginx -y
```

### 4. 🔍 Check NGINX Status
```bash
sudo systemctl status nginx
```
If not running:
```bash
sudo systemctl enable nginx
sudo systemctl start nginx
```
Reload NGINX:
```bash
sudo systemctl reload nginx
```
Visit in browser:
```http
http://<public-ip>
```

### 5. 🔽 Clone the Static Website Project
```bash
git clone https://github.com/jaiswaladi246/nginx-loadbalancer.git
```

### 6. 📁 Copy Frontend to Web Root
```bash
cd /var/www/
sudo cp -r /home/ubuntu/nginx-loadbalancer/frontend/ .
sudo chown -R www-data:www-data frontend/
```

### 7. ⚙️ Create NGINX Load Balancer Config
```bash
cd /etc/nginx/sites-available/
sudo vi nginx-loadbalancer
```
Paste the following config:
```nginx
upstream backend_apis {
    least_conn;
    server 34.229.121.248:3001;
    server 34.229.121.248:3002;
}

server {
    listen 80;
    server_name 34.229.121.248;

    root /var/www/frontend;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location /api/ {
        proxy_pass http://backend_apis;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### 8. 🔁 Link Config to Sites Enabled
```bash
sudo ln -s /etc/nginx/sites-available/nginx-loadbalancer /etc/nginx/sites-enabled/
```

### 9. ✅ Test and Reload NGINX
```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

## ⚙️ Setup Node.js Backend Instances

### Navigate to Backend Folders
```bash
cd /home/ubuntu/nginx-loadbalancer/
```

### For Backend 1:
```bash
cd backend1
npm init -y
npm install express
node index.js
```

### For Backend 2:
```bash
cd backend2
npm init -y
npm install express
node index.js
```

Both backends should now be listening on:
- `3001` for backend1
- `3002` for backend2

### Sample Express Code (`index.js`):
```js
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3001;

app.get('/api', (req, res) => {
  res.send(`Response from backend on port ${PORT}`);
});

app.listen(PORT, () => {
  console.log(`Backend running on port ${PORT}`);
});
```

Set `PORT=3001` or `PORT=3002` in each backend respectively.

---

## 🧪 Simulate Load Traffic
Install Apache Benchmark tool:
```bash
sudo apt install apache2-utils -y
```
Run the load test:
```bash
ab -n 100 -c 10 http://34.229.121.248/api/
```

### 🔍 Why Apache Benchmark (`ab`)?
Apache Bench is used to:
- Simulate concurrent users hitting the `/api` endpoint
- Verify if NGINX load balancing works (round-robin/least connections)
- Measure performance under load

Expected: Alternate responses from port 3001 and 3002 indicating successful load distribution.

---


## 📌 Useful Commands Cheat Sheet
```bash
# Start & Enable NGINX
sudo systemctl start nginx
sudo systemctl enable nginx

# Reload NGINX after config changes
sudo nginx -t
sudo systemctl reload nginx

# Clone repo
git clone https://github.com/jaiswaladi246/nginx-loadbalancer.git

# Run backend services
node backend1/index.js
node backend2/index.js

# Run Apache Benchmark
ab -n 100 -c 10 http://<your-ip>/api/
```

---

## 🎯 Output
You should see different responses from backend 1 and backend 2 when hitting:
```bash
http://<your-ec2-ip>/api/
