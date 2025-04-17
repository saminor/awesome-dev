# How to Config Postgers On Django

a detailed step-by-step guide for configuring PostgreSQL with Django on Linux.
This guide includes the commands to install PostgreSQL, create a database, set up a user with full access, and configure Django to connect to PostgreSQL.

---

### **1. Install PostgreSQL**
First, ensure PostgreSQL is installed on your Linux system. Use the following commands to install it:

```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
```

After installation, the PostgreSQL service should start automatically. You can check its status with:

```bash
sudo systemctl status postgresql
```

If it’s not running, start it using:

```bash
sudo systemctl start postgresql
```

---

### **2. Log in to the PostgreSQL Shell**
Switch to the `postgres` user to access the PostgreSQL shell:

```bash
sudo -i -u postgres
```

Then, open the PostgreSQL interactive terminal:

```bash
psql
```

---

### **3. Create a Database**
Inside the PostgreSQL shell, create a new database for your Django project. Replace `<your_database_name>` with your desired database name:

```sql
CREATE DATABASE <your_database_name>;
```

Example:

```sql
CREATE DATABASE myprojectdb;
```

---

### **4. Create a PostgreSQL User**
Create a new PostgreSQL user. Replace `<your_username>` and `<your_password>` with your desired username and password:

```sql
CREATE USER <your_username> WITH PASSWORD '<your_password>';
```

Example:

```sql
CREATE USER myprojectuser WITH PASSWORD 'mypassword';
```

---

### **5. Grant Access to the User**
Give the user full access to the newly created database:

```sql
GRANT ALL PRIVILEGES ON DATABASE <your_database_name> TO <your_username>;
```

Example:

```sql
GRANT ALL PRIVILEGES ON DATABASE myprojectdb TO myprojectuser;
```

---

### **6. Exit the PostgreSQL Shell**
Exit the PostgreSQL shell by typing:

```sql
\q
```

And then return to your normal user shell:

```bash
exit
```

---

### **7. Install the `psycopg2` Library**
Django requires the `psycopg2` package to connect to PostgreSQL. Install it using pip:

```bash
pip install psycopg2-binary
```

If you're using a virtual environment, make sure it’s activated before running this command.

---

### **8. Configure Django Settings**
Update your Django project’s `settings.py` file to use PostgreSQL as the database backend. Replace the placeholders with your actual database credentials:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': '<your_database_name>',
        'USER': '<your_username>',
        'PASSWORD': '<your_password>',
        'HOST': 'localhost',
        'PORT': '5432',  # Default PostgreSQL port
    }
}
```

Example:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'NAME': 'myprojectdb',
        'USER': 'myprojectuser',
        'PASSWORD': 'mypassword',
        'HOST': 'localhost',
        'PORT': '5432',
    }
}
```

---

### **9. Apply Migrations**
Run Django migrations to set up the database schema:

```bash
python manage.py makemigrations
python manage.py migrate
```

---

### **10. Test the Connection**
You can test the connection by running the Django development server:

```bash
python manage.py runserver
```

If everything is configured correctly, your Django app should now be connected to PostgreSQL.

---

### **Optional: Enable Remote Access (if needed)**
By default, PostgreSQL only allows local connections. If you need to connect to the database from a remote server, follow these steps:

#### a) Edit the PostgreSQL Configuration File:
Open the `postgresql.conf` file:

```bash
sudo nano /etc/postgresql/<version>/main/postgresql.conf
```

Replace `<version>` with your PostgreSQL version (e.g., `14`).

Find the line starting with `#listen_addresses` and update it to:

```conf
listen_addresses = '*'
```

#### b) Edit the `pg_hba.conf` File:
Open the `pg_hba.conf` file:

```bash
sudo nano /etc/postgresql/<version>/main/pg_hba.conf
```

Add the following line to allow connections from any IP address (replace `<your_ip>` with a specific IP if needed):

```conf
host    all             all             0.0.0.0/0               md5
```

#### c) Restart PostgreSQL:
Restart the PostgreSQL service to apply the changes:

```bash
sudo systemctl restart postgresql
```

You should now be able to connect to the database remotely.

---

### **Summary of Commands**
Here’s a quick summary of the key commands used in this guide:

1. Install PostgreSQL:
```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
```

2. Switch to the PostgreSQL shell:
```bash
sudo -i -u postgres
psql
```

3. Create a database and user:
```sql
CREATE DATABASE <your_database_name>;
CREATE USER <your_username> WITH PASSWORD '<your_password>';
GRANT ALL PRIVILEGES ON DATABASE <your_database_name> TO <your_username>;
```

4. Exit the shell:
```sql
\q
exit
```

5. Install `psycopg2`:
```bash
pip install psycopg2-binary
```

6. Apply migrations:
```bash
python manage.py makemigrations
python manage.py migrate
```

