
# 📄 Project 01: Static Website Hosting using NGINX on AWS EC2  

---

## 🔧 Objective  
Deploy a static website using **NGINX** on an **Ubuntu EC2 instance** by cloning a GitHub project, configuring file permissions, setting up a virtual host, and troubleshooting NGINX configuration.  

---

## ✅ Step-by-Step Implementation  

### 1. 🟢 Launch & Connect to EC2 Instance  
- Launch an Ubuntu EC2 instance from AWS Management Console.  
- Connect to the instance via SSH:  

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

If it’s not running:

```bash
sudo systemctl enable nginx
sudo systemctl start nginx
```

Reload NGINX:

```bash
sudo systemctl reload nginx
```

Open in browser:

```
http://<public-ip>
```

---

### 5. 🔽 Clone Static Website Project

```bash
git clone https://github.com/jaiswaladi246/static-site.git
```

---

### 6. 📁 Explore Important Folders

| Path          | Description                     |
| ------------- | ------------------------------- |
| `/etc/nginx/` | Main NGINX config folder        |
| `/var/www/`   | Default directory for web files |

---

### 7. 📂 Copy Website Code to Web Root

```bash
sudo cp -r /home/ubuntu/static-site/ /var/www/
ls -ltr /var/www/
```

Expected output:

```
html/        # default NGINX site
static-site/ # your project
```

---

### 8. 👤 Set Correct File Ownership

```bash
cd /var/www/
sudo chown -R www-data:www-data static-site/
```

---

### 9. ⚙️ Create Custom NGINX Site Configuration

```bash
cd /etc/nginx/sites-available/
sudo vi static-site.conf
```

Paste this config:

```nginx
server {
    listen 80 default_server;
    server_name localhost;  # catch-all for all unknown domains

    root /var/www/static-site;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

Save & exit: `:wq!`

---

### 10. 🔗 Enable the Site Configuration

```bash
sudo ln -s /etc/nginx/sites-available/static-site.conf /etc/nginx/sites-enabled/
```

---

### 11. 🧪 Test NGINX Configuration

```bash
sudo nginx -t
```

If you get this error:

```
open() "/etc/nginx/sites-enabled/static-site" failed
```

👉 It means your `nginx.conf` is including the wrong path.

---

### 12. 🛠️ Fix nginx.conf Include Path

```bash
sudo vi /etc/nginx/nginx.conf
```

Replace:

```nginx
include /etc/nginx/sites-enabled/*;
```

With:

```nginx
include /etc/nginx/sites-enabled/static-site.conf;
```

Save & exit.

---

### 13. 🔁 Reload NGINX

```bash
sudo systemctl reload nginx
```

---

### 14. 🌍 Visit Your Hosted Static Website

```
http://<your-ec2-public-ip>
```

---

## 📁 Sample Project Structure

```
/var/www/static-site/
├── index.html
├── about.html
├── css/
├── images/
└── js/
```

---

## 🧠 NGINX Configuration Summary

```nginx
server {
    listen 80 default_server;
    server_name localhost;
    root /var/www/static-site;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

| Directive     | Purpose                              |
| ------------- | ------------------------------------ |
| `listen 80`   | Listen on HTTP port                  |
| `server_name` | Matches localhost or any domain      |
| `root`        | Path to website files                |
| `try_files`   | Serve file, directory, or return 404 |

---


## 📌 Useful Commands Cheat Sheet

```bash
# Start/Enable NGINX
sudo systemctl start nginx
sudo systemctl enable nginx

# Clone project
git clone https://github.com/jaiswaladi246/static-site.git

# Copy files
sudo cp -r static-site/ /var/www/

# Change ownership
sudo chown -R www-data:www-data /var/www/static-site/

# Create and link config
sudo vi /etc/nginx/sites-available/static-site.conf
sudo ln -s /etc/nginx/sites-available/static-site.conf /etc/nginx/sites-enabled/

# Test and reload
sudo nginx -t
sudo systemctl reload nginx
```



