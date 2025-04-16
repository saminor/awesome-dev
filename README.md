# awesome-dev
awesome points and technical things for developing and devops 


برای اینکه سرویس‌های فرانت‌اند (Next.js) و بک‌اند (Django) روی سرور اوبونتو پایدار باشند و همیشه در بک‌گراند اجرا شوند، می‌توانید از ابزارهای زیر استفاده کنید:

---

### 1. **استفاده از Systemd برای مدیریت سرویس‌ها**
Systemd یک ابزار مدیریت سرویس در لینوکس است که امکان ایجاد سرویس‌های پایدار و خودکار را فراهم می‌کند. شما می‌توانید فایل‌های سرویس برای هر دو برنامه ایجاد کنید.

#### مراحل:
##### ایجاد فایل سرویس برای Next.js:
1. فایل جدیدی به نام `nextjs.service` ایجاد کنید:
   ```bash
   sudo nano /etc/systemd/system/nextjs.service
   ```

2. محتوای زیر را در فایل قرار دهید:
   ```ini
   [Unit]
   Description=Next.js Service
   After=network.target

   [Service]
   ExecStart=/usr/bin/node /path/to/your/nextjs/app/.next/server.js
   WorkingDirectory=/path/to/your/nextjs/app
   Restart=always
   User=your-username
   Environment=PATH=/usr/bin:/usr/local/bin
   Environment=NODE_ENV=production

   [Install]
   WantedBy=multi-user.target
   ```

3. سرویس را فعال کنید:
   ```bash
   sudo systemctl enable nextjs.service
   sudo systemctl start nextjs.service
   ```

##### ایجاد فایل سرویس برای Django:
1. فایل جدیدی به نام `django.service` ایجاد کنید:
   ```bash
   sudo nano /etc/systemd/system/django.service
   ```

2. محتوای زیر را در فایل قرار دهید:
   ```ini
   [Unit]
   Description=Django Service
   After=network.target

   [Service]
   ExecStart=/path/to/your/virtualenv/bin/python /path/to/your/django/manage.py runserver 0.0.0.0:8000
   WorkingDirectory=/path/to/your/django/project
   Restart=always
   User=your-username
   Environment=PATH=/path/to/your/virtualenv/bin:/usr/bin:/usr/local/bin
   Environment=DJANGO_ENV=production

   [Install]
   WantedBy=multi-user.target
   ```

3. سرویس را فعال کنید:
   ```bash
   sudo systemctl enable django.service
   sudo systemctl start django.service
   ```

---

### 2. **استفاده از PM2 برای مدیریت Node.js و Django**
PM2 یک ابزار مدیریت پروسه است که معمولاً برای اپلیکیشن‌های Node.js استفاده می‌شود، اما می‌توان از آن برای اپلیکیشن‌های دیگر نیز استفاده کرد.

#### مراحل:
##### نصب PM2:
1. ابتدا PM2 را نصب کنید:
   ```bash
   npm install -g pm2
   ```

##### اضافه کردن Next.js:
1. اپلیکیشن Next.js را با PM2 اجرا کنید:
   ```bash
   pm2 start /path/to/your/nextjs/app/.next/server.js --name "nextjs"
   ```

##### اضافه کردن Django:
1. اپلیکیشن Django را با PM2 اجرا کنید:
   ```bash
   pm2 start /path/to/your/virtualenv/bin/python --name "django" -- /path/to/your/django/manage.py runserver 0.0.0.0:8000
   ```

##### ذخیره تنظیمات:
1. تنظیمات PM2 را ذخیره کنید تا پس از ری‌استارت سرور به صورت خودکار اجرا شوند:
   ```bash
   pm2 save
   pm2 startup
   ```

---

### 3. **تنظیم Reverse Proxy با Nginx**
برای دسترسی به سرویس‌ها از طریق اینترنت، بهتر است از Nginx به عنوان Reverse Proxy استفاده کنید.

#### مراحل:
##### نصب Nginx:
1. نصب Nginx:
   ```bash
   sudo apt update
   sudo apt install nginx
   ```

##### تنظیمات Nginx:
1. فایل تنظیمات جدیدی برای سرویس‌های خود ایجاد کنید:
   ```bash
   sudo nano /etc/nginx/sites-available/yourproject
   ```

2. محتوای زیر را در فایل قرار دهید:
   ```nginx
   server {
       listen 80;

       server_name yourdomain.com;

       location / {
           proxy_pass http://127.0.0.1:3000;
           proxy_http_version 1.1;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection 'upgrade';
           proxy_set_header Host $host;
           proxy_cache_bypass $http_upgrade;
       }

       location /api/ {
           proxy_pass http://127.0.0.1:8000;
           proxy_set_header Host $host;
           proxy_set_header X-Real-IP $remote_addr;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header X-Forwarded-Proto $scheme;
       }
   }
   ```

3. فایل را فعال کنید:
   ```bash
   sudo ln -s /etc/nginx/sites-available/yourproject /etc/nginx/sites-enabled/
   ```

4. تنظیمات Nginx را بررسی کنید:
   ```bash
   sudo nginx -t
   ```

5. Nginx را ری‌استارت کنید:
   ```bash
   sudo systemctl restart nginx
   ```

---

### نتیجه:
- با استفاده از Systemd یا PM2، سرویس‌های شما همیشه اجرا خواهند شد و در صورت قطع شدن خودکار ری‌استارت می‌شوند.
- با تنظیم Nginx، می‌توانید دسترسی به سرویس‌ها را از طریق دامنه یا IP عمومی خود مدیریت کنید.

اگر سوالی داشتی یا به کمک بیشتری نیاز داشتی، بگو تا راهنمایی کنم!
