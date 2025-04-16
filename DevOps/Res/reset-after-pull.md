### General Process
1. **On your local system:** Write the new code, commit it, and push it to GitHub. Do this as usual (e.g., with `git add .`, `git commit -m "message"`, and `git push origin main`).
2. **On the web server:** SSH into the server, pull the changes from GitHub, and then update the services if necessary.

Now, regarding your main question: **Yes, after pulling changes, you might need to restart the services for the changes to take effect.** This depends on the type of changes, but let's go into the details.

### Do You Need to Restart Anything?
- **Yes, probably, but not always.** 
  - If the changes include new code or modifications to core project files (like Django or Next.js code), the services need to be restarted so the new code loads.
  - If the changes are only to static files (like CSS or images), you might not need a restart, as Nginx or the services might serve them automatically.

#### 1. For the Backend (Django with Gunicorn):
- **Why might you need to restart?** Gunicorn keeps Django code in memory, so if you pulled new code (like changes to models, views, or settings), you need to restart Gunicorn to apply the changes.
- **Recommended commands:**
```bash
cd /path/to/your/django/project  # Navigate to the Django project directory (e.g., RoyaClinic)
git pull origin main  # Assuming the main branch; this pulls the changes
```
Then, restart Gunicorn:
```bash
sudo systemctl restart gunicorn
```
- If the changes include database migrations (like adding a new field), definitely run these commands:
```bash
python manage.py makemigrations
python manage.py migrate
```
- After restarting, check the status:
```bash
sudo systemctl status gunicorn
```

#### 2. For the Frontend (Next.js with PM2):
    - **Why might you need to restart?** PM2 manages the Next.js process, but if you have new code (like changes to components or pages), you need to rebuild the project and restart PM2 to load the changes.
    - **Recommended commands:**
```bash
cd /path/to/your/nextjs/project  # Navigate to the Next.js project directory
git pull origin main  # Pull the changes
npm run build  # Rebuild the project (this step is important for production)
pm2 restart nextjs-app  # Replace nextjs-app with the name you previously set
```
- After restarting, check the status:
```bash
pm2 status
```

### Important Tips for Stability
- **Minimizing Downtime:** Restart commands are usually quick and bring services back up without long interruptions. If you want to be more professional, you can use `pm2 reload` for Next.js to restart the process without downtime.
- **Test Before Applying:** Always test changes on your local system first to ensure no issues arise. After pulling on the server, verify that everything works.
- **Monitoring After Applying:** After restarting, check the services to ensure everything is stable:
  - Gunicorn: `sudo systemctl status gunicorn`
  - PM2: `pm2 status`
- **Git Workflow:** Make sure to manage branches properly (e.g., using `main` or `develop`) to avoid conflicts.

