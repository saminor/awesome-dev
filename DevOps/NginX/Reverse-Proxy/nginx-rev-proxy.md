Here’s the same content translated into English and aligned to the left:

---

### **Introduction**
To set up a web server on Ubuntu where the frontend service (Next.js) runs on port 3000 and the backend service (Django) runs on port 8000, and to ensure these services run stably and always in the background, follow the step-by-step guide below. We will use tools like `pm2` for managing Node.js (Next.js) processes and `gunicorn` along with `systemd` for Django. Additionally, we’ll configure `nginx` as a reverse proxy to handle requests and route traffic to the appropriate ports.

Based on your previous mention of working on a project named RoyaClinic and encountering issues with accessing port 8000 from the internet, this guide is tailored to address those challenges.

---

### **Prerequisites**
1. Ubuntu server (assuming version 20.04 or 22.04).
2. Root access or a user with sudo privileges.
3. Node.js and npm installed for Next.js.
4. Python and Django installed for the backend.
5. Both Next.js and Django projects deployed on the server.

---

### **Step 1: Setting up the Frontend (Next.js) on Port 3000**
To ensure the Next.js process runs stably, we’ll use `pm2`, a process manager for Node.js applications.

#### 1.1. Install pm2
Install `pm2` globally:
```bash
sudo npm install -g pm2
```

#### 1.2. Run Next.js Project with pm2
Navigate to the Next.js project directory and ensure the project is built (if in production mode):
```bash
cd /path/to/your/nextjs-project
npm run build
```

Start the Next.js project using pm2:
```bash
pm2 start npm --name "nextjs-app" -- start
```

- `nextjs-app` is the name you assign to this process; feel free to change it.
- If the default port is not 3000, you can specify it with `-- --port=3000`.

#### 1.3. Ensure pm2 Restarts Automatically After Server Reboot
To enable pm2 to automatically restart processes after a server reboot:
```bash
pm2 startup
pm2 save
```

You can check the process status:
```bash
pm2 status
```

---

### **Step 2: Setting up the Backend (Django) on Port 8000**
For Django, we’ll use `gunicorn` as the WSGI server and manage it with `systemd` to ensure it runs stably in the background.

#### 2.1. Install gunicorn
Install `gunicorn`:
```bash
pip install gunicorn
```

#### 2.2. Test Django with gunicorn
Navigate to the Django project directory and test running it with gunicorn:
```bash
cd /path/to/your/django-project
gunicorn --bind 0.0.0.0:8000 your_project_name.wsgi:application
```

- Replace `your_project_name` with the name of your Django project (the directory containing the `wsgi.py` file).
- If everything works, this command should start the server on port 8000. Press Ctrl+C to stop it.

#### 2.3. Create a systemd Service for gunicorn
To keep gunicorn running stably, create a new `systemd` service file. Open a new file:
```bash
sudo nano /etc/systemd/system/gunicorn.service
```

Add the following content (adjust paths and user settings to match your environment):
```ini
[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=your_username
Group=www-data
WorkingDirectory=/path/to/your/django-project
Environment="PATH=/path/to/your/venv/bin"
ExecStart=/path/to/your/venv/bin/gunicorn --workers 3 --bind 0.0.0.0:8000 your_project_name.wsgi:application

[Install]
WantedBy=multi-user.target
```

- Replace `your_username` with the username running the project.
- Adjust `/path/to/your/venv/bin` to the path of your virtual environment.
- Replace `your_project_name` with your Django project name.

Save and close the file (Ctrl+O, Enter, Ctrl+X in nano).

#### 2.4. Enable and Start the gunicorn Service
Enable and start the service:
```bash
sudo systemctl start gunicorn
sudo systemctl enable gunicorn
```

Check the service status:
```bash
sudo systemctl status gunicorn
```

If everything is set up correctly, gunicorn will run on port 8000 in the background.

---

### **Step 3: Install and Configure Nginx as a Reverse Proxy**
To manage requests and route traffic to ports 3000 (Next.js) and 8000 (Django), use Nginx as a reverse proxy.

#### 3.1. Install Nginx
Install Nginx:
```bash
sudo apt update
sudo apt install nginx
```

#### 3.2. Configure Nginx
Create a new configuration file for your site:
```bash
sudo nano /etc/nginx/sites-available/your-site
```

Add the following content (adjust for your domain or server IP):
```nginx
server {
    listen 80;
    server_name your-domain.com;  # Replace with your domain or server IP

    # Forward requests to the frontend (Next.js)
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    # Forward requests to the backend API (Django)
    location /api/ {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Forward admin requests to Django
    location /admin/ {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

- Replace `your-domain.com` with your server’s domain or IP.
- Adjust the `/api/` path if your API is accessible via a different prefix.

#### 3.3. Enable the Site and Restart Nginx
Create a symbolic link and restart Nginx:
```bash
sudo ln -s /etc/nginx/sites-available/your-site /etc/nginx/sites-enabled/
sudo nginx -t  # Test configuration
sudo systemctl restart nginx
```

---

### **Step 4: Open Firewall Ports (Optional)**
If your server uses a firewall (e.g., ufw), open port 80 (for Nginx):
```bash
sudo ufw allow 80/tcp
```

To directly access ports 3000 and 8000 (not recommended since Nginx handles this), open them as well:
```bash
sudo ufw allow 3000/tcp
sudo ufw allow 8000/tcp
```

---

### **Step 5: Check Service Stability**
- For Next.js (pm2):
  ```bash
  pm2 status
  ```
- For Django (gunicorn):
  ```bash
  sudo systemctl status gunicorn
  ```
- For Nginx:
  ```bash
  sudo systemctl status nginx
  ```

---

### **Additional Notes**
1. **Accessing Port 8000 from the Internet:** If you still face issues, ensure port 8000 is open in your server’s firewall and network settings (e.g., AWS Security Group). Using Nginx to route traffic from port 80 to 8000 eliminates the need to expose port 8000 directly.
2. **HTTPS:** For security, enable SSL/TLS using Let’s Encrypt:
   ```bash
   sudo apt install certbot python3-certbot-nginx
   sudo certbot --nginx
   ```
3. **Logs:** Check logs for debugging:
   - pm2: `pm2 logs nextjs-app`
   - gunicorn: `sudo journalctl -u gunicorn`
   - Nginx: `sudo tail -f /var/log/nginx/error.log`

---

If you encounter any issues or need further clarification, feel free to ask!