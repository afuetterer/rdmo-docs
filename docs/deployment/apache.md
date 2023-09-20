# Apache and mod_wsgi

In production, you should create a dedicated user for RDMO. All steps for the installation, which do not need root access, should be done using this user. As before, we assume this user is called `rdmo` and it's home is `/srv/rdmo` and therefore your `rdmo-app` is located in `/srv/rdmo/rdmo-app`.

Install the Apache server and `mod_wsgi` on Debian or Ubuntu using:

```bash
sudo apt install apache2 libapache2-mod-wsgi-py3
```

Next, create a virtual host configuration. Unfortunately, the different distributions use different versions of Apache and mod_wsgi and therefore require a slightly different setup:

For Debian/Ubuntu in `/etc/apache2/sites-available/000-default.conf` use (other distributions might use a slightly different setup):

```
<VirtualHost *:80>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    Alias /static /srv/rdmo/rdmo-app/static_root/
    <Directory /srv/rdmo/rdmo-app/static_root/>
        Require all granted
    </Directory>

    WSGIDaemonProcess rdmo user=rdmo group=rdmo \
        home=/srv/rdmo/rdmo-app python-home=/srv/rdmo/rdmo-app/env
    WSGIProcessGroup rdmo
    WSGIScriptAlias / /srv/rdmo/rdmo-app/config/wsgi.py process-group=rdmo
    WSGIPassAuthorization On

    <Directory /srv/rdmo/rdmo-app/config/>
        <Files wsgi.py>
            Require all granted
        </Files>
    </Directory>
</VirtualHost>
```

Restart the Apache server: `sudo service apache2 restart`. RDMO should now be available on `YOURDOMAIN`. Note that the Apache user needs to have access to `/srv/rdmo/rdmo-app/static_root/`.

As you can see from the virtual host configurations, the static assets such as CSS and JavaScript files are served independently from the WSGI-python script. In order to do so, they need to be gathered in the `static_root` directory. This can be achieved by running:

```bash
python manage.py collectstatic --clear
```

in your virtual environment  (`--clean` removes existing files before collecting).

In order to apply changes to the RDMO code (e.g. after an [upgrade](../upgrade/index.html)), the webserver needs to be reloaded or the `config/wsgi.py` file needs to appear modified. This can be done using the `touch` command:

```bash
touch config/wsgi.py
```

Also, the `collectstatic` command has to be executed again. Both can be achieved by using

```bash
python manage.py deploy
```

in your virtual environment.
