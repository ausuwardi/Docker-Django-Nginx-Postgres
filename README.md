# Docker-Django-Nginx-Postgres

Dockerfile for Django with uWSGI, Nginx, Postgres, Python 3.

## About this Image/Dockerfile

This **Image/Dockerfile** aims to create a container for **Django** with **PostgreSQL**, **Python 3** and using **uWSGI**, **Nginx** to hosting.

## How to use?

You can build this **Dockerfile** yourself:

```
sudo docker build -t "chenjr0719/django-nginx-postgres" .
```

Or, just pull my **image**:

```
sudo docker pull chenjr0719/django-nginx-postgres
```

Then, run this image:

```
sudo docker run -itd -p 80:80 chenjr0719/django-nginx-postgres
```

Wait a minute, you can see the initial project of **Django** at http://127.0.0.1

### Check it work properly

You can check is **Django** work properly with **PostgreSQL Server** by:

1. First, query your **Django Admin Password**:

   ```
   sudo docker exec -it $CONTAINER_ID cat /home/django/password.txt
   ```

2. Access http://127.0.0.1/admin and log in as Username: admin.

3. Choose **Model_Example** to test **CRUD** function.

## Use your Django project?

If you want to use your **Django** project which you already developed, use following command:

```
sudo docker run -itd -p 80:80 -v $YOUR_PROJECET_DIR:/home/django/website chenjr0719/django-nginx-postgres
```

### Modify Django setting with PostgreSQL

You have to make sure that your **settings.py** is already set all setting with **PostgreSQL**.

1. Enter to your container:

  ```
  sudo docker exec -it $CONTAINER_ID bash
  ```

2. Query password of **PostgreSQL**:

  ```
  cat /home/django/password.txt
  ```

3. Modify **settings.py**:

  ```
  SETTING_PATH=`find /home/django/website -name settings.py`
  sed -i "s|'django.contrib.staticfiles'|'django.contrib.staticfiles',\n '$YOUR_APP_NAME'|g" $SETTING_PATH
  sed -i "s|django.db.backends.sqlite3|django.db.backends.postgresql_psycopg2|g" $SETTING_PATH
  sed -i "s|os.path.join(BASE_DIR, 'db.sqlite3')|'django',\n        'HOST': '127.0.0.1',\n        'USER': 'django',\n        'PASSWORD': '$POSTGRES_DJANGO_PASSWORD'|g" $SETTING_PATH
  ```

4. Let **django** create tables automatically:

  ```
  python3 /home/django/website/manage.py makemigrations
  python3 /home/django/website/manage.py migrate
  ```

5. Exit your container and restart it:

  ```
  exit
  sudo docker restart $CONTAINER_ID
  ```

### Modify project name

If your project name is not **website**, this image will not work properly.

you need to modify the setting of **uwsgi.ini** in your container:

```
sudo docker exec $CONTAINER_ID sed -i "s|module=website.wsgi:application|module=$PROJECT_NAME.wsgi:application|g" /home/django/uwsgi.ini
sudo docker restart $CONTAINER_ID
```

## About Django static files

If you want to use **Django** static files, you have to:

1. Enter to your container:

  ```
  sudo docker exec -it $CONTAINER_ID bash
  ```

2. Modify the setting of **Django**.

  ```
  SETTING_PATH=`find /home/django/website -name settings.py`
  vim $SETTING_PATH
  ```

  In the **Static files** section, if your static files are in templates/static, add following setting:

  ```
  STATICFILES_DIRS = [
  os.path.join(BASE_DIR, "templates/static"),
  ]

  STATIC_ROOT = os.path.join(BASE_DIR, "static")
  ```

3. Run the following command to collect all static files of your project into a folder.

  In default it will use /static/, you can change it by modifying STATIC_ROOT in **settings.py**

  ```
  echo yes | python3 /home/django/website/manage.py collectstatic
  ```

4. If you want to use different name of static folder, you need to modify the setting of **nginx-site.conf** in your container.

  You can this command:

  ```
  sed -i "s|/home/django/website/static|/home/django/website/$STATIC_FOLDER_NAME|g" /etc/nginx/sites-available/default
  ```

5. Exit your container and restart it:

  ```
  exit
  sudo docker restart $CONTAINER_ID
  ```
