# Project 02: NGINX as a Reverse Proxy with Node.js Backend

## ✅ Objective
Host a static frontend using NGINX and reverse proxy all API requests to a Node.js backend running on the same EC2 instance.

---

![Project Output](/images/project-02-nginx-reverse-proxy.png)

## ✅ Step-by-Step Implementation

### 1. 🟢 Launch and Connect to EC2 Instance
* Launch an **Ubuntu EC2 instance** from the AWS Management Console.
* SSH into your instance:

```bash
ssh -i <your-key.pem> ubuntu@<public-ip-address>
````

---

### 2. 🔄 Update System Packages

```bash
sudo apt update
```

---

### 3. 🌐 Install NGINX

```bash
sudo apt install nginx -y
```

---

### 4. 🔍 Check NGINX Status

```bash
sudo systemctl status nginx
```

If not running:

```bash
sudo systemctl enable nginx
sudo systemctl start nginx
sudo systemctl reload nginx
```

Open your browser:

```
http://<public-ip>
```

You should see the default NGINX welcome page.

---

### 5. 🔽 Clone Static Website Project

```bash
git clone https://github.com/jaiswaladi246/nginx-node-proxy.git
```

---

### 6. 📂 Copy Frontend to Web Root

```bash
cd /var/www/
sudo cp -r /home/ubuntu/nginx-node-proxy/frontend/ .
sudo chown -R www-data:www-data frontend/
```

Check structure:

```bash
ls -l /var/www/
```

Expected output:

```
frontend/  html/
```

---

### 7. ⚙️ Create NGINX Site Configuration

```bash
cd /etc/nginx/sites-available/
sudo vi nginx-node-proxy
```

Paste the following configuration:

```nginx
server {
    listen 80;
    server_name _;  # Use your domain or "_" as a catch-all

    root /var/www/frontend;
    index index.html;

    # Serve static files
    location / {
        try_files $uri $uri/ =404;
    }

    # Reverse proxy to Node.js API
    location /api/ {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    # WebSocket support
    location /socket.io/ {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
    }
}
```

---

### 8. 🔗 Enable Site & Reload NGINX

```bash
sudo ln -s /etc/nginx/sites-available/nginx-node-proxy /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

### 9. ⚙️ Set Up Node.js Backend

```bash
cd ~/nginx-node-proxy/backend
```

Install Node.js using NVM:

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
. "$HOME/.nvm/nvm.sh"
nvm install 22
```

Verify:

```bash
node -v
npm -v
nvm current
```

---

### 10. 🚀 Initialize & Install Backend Dependencies

```bash
npm init -y   # Initializes package.json with defaults
npm install express cors socket.io
```

Start the backend:

```bash
node index.js
```

Expected output:

```
Backend listening at http://0.0.0.0:3000
```

---

### ✅ Common Errors & Fixes

| Issue                                                     | Resolution                                                                               |
| --------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| `nginx: [emerg] unexpected end of file`                   | Ensure all `{}` in nginx conf are closed.                                                |
| `nginx: open() "/etc/nginx/sites-enabled/default" failed` | Remove broken symlink: `sudo rm /etc/nginx/sites-enabled/default`                        |
| Frontend not loading                                      | Ensure `index.html` is in `/var/www/frontend/` not in a nested directory.                |
| `ERR_CONNECTION_REFUSED` on port 3000                     | Ensure backend is running: `sudo lsof -i :3000` and EC2 Security Group allows port 3000. |
| `curl http://localhost` returns 404                       | Double-check the `root` path in NGINX config and static files exist.                     |

---

### 🔍 Test URLs

* Frontend: [http://<your-ec2-ip>](http://<your-ec2-ip>)
* API Test: [http://<your-ec2-ip>/api/hello](http://<your-ec2-ip>/api/hello)

---

## ✅ Summary

* ✅ NGINX configured as static server + reverse proxy
* ✅ Node.js backend responds to `/api/` routes
* ✅ WebSocket support added via `/socket.io/`
* ✅ Permissions and ownership configured
* ✅ NGINX errors resolved and backend is reachable

---

## 📁 Project Folder Structure

```
/var/www/frontend/
├── index.html
├── css/
├── js/
└── images/

~/nginx-node-proxy/backend/
├── index.js
├── package.json
└── node_modules/
```

---

## ✅ Useful Commands

```bash
# Check NGINX status
sudo systemctl status nginx

# Reload after config changes
sudo systemctl reload nginx

# View logs
sudo tail -f /var/log/nginx/error.log

# Check if port 3000 is in use
sudo lsof -i :3000
```

