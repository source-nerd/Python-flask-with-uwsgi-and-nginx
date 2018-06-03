<p align="left">
  <a href="https://github.com/source-nerd/Python-flask-with-uwsgi-and-nginx"><img src="https://camo.githubusercontent.com/9a140a4c68e7c178bc660bee7675f4f25ff7ade3/68747470733a2f2f696d672e736869656c64732e696f2f6e706d2f6c2f7675652e737667" alt="License"></a>
</p>

# PYTHON FLASK WITH NGINX & UWSGI

## CONTENTS

1. app Folder
2. uwsgi.ini
3. nginx_config
4. production_flask.service

## USAGE

1. app Folder - It contains you complete app and directories. Do initialize it as a virtual environment and install `requirements.txt` included in that folder. (Hold on !!) We will be going through it in some time
2. uwsgi.ini - UWSGI server configuration. Edit/Replace the following places:
    * my_app_folder: Replace `PROJECT_FOLDER_HERE` With your Project app Path
    * my_user: Replace `MY_USERNAME_HERE` With your UserName; You can get your username using command `whoami`
3. nginx_config - Contains the main nginx configuration. Edit/Replace the following to make it work properly:
    * uwsgi_pass: Replace `PROJECT_FOLDER_HERE` With your Project Path
    * [OPTIONAL] listen: Port on which you want nginx to Listen
    * [OPTIONAL] server_name: Currently its `0.0.0.0` . You can change it as per your needs.
4. production_flask.service - A Systemd service file to ensure that the server restarts on failure and can be set to auto-start in case of server restart. Edit/Replace the following terms:
    * Replace `USER_NAME_HERE` with your user-name.
    * Replace `PROJECT_FOLDER_HERE` with your project app directory
    * Replace `uwsgi.ini_PATH_HERE` with `uwsgi.ini` path

## PREREQUISITES

1. Clone this repository.
2. A Stable Debian based Linux O.S, preferrably Ubuntu with `sudo` privileges configured. I have tried it on Ubuntu 18.04 and it worked flawlessly.
3. We will me utilizing Virtual Env Wrapper for complete Flask Project.
4. Before proceeding any further, Install the following packages:

    ``` PYTHON
    sudo apt-get update
    sudo apt-get install python3 python3-pip
    sudo apt-get install systemd nginx
    sudo pip3 install virtualenv
    ```

5. Initializing Project Virtual Environment:

    ```PYTHON
    cd PROJECT/APP_FOLDER
    virtualenv -p python3 venv
    source venv/bin/activate
    pip3 install -r requirements.txt
    ```
6. Now, as this project is configured with a simple Hello World Application inside `app.py`, we will be using it for deployment. You can also have your complete project inside the app folder with an `app.py` file

## INSTALLATION INSTRUCTIONS

Now comes the main dreaded and & feared (kiddin!) Installation part.
Considering you have installed all the above steps successfully, will start with Nginx.

1. You will need to place your `nginx_config` in  `/etc/nginx/sites-available/nginx_config`. Then, to enable this nginx configuration, we will have to link it to the nginx sites-enabled directory using this command:
    ```SHELL
    sudo ln -s /etc/nginx/sites-available/nginx_config /etc/nginx/sites-enabled
    ```
    The above command will create a sym-link for `nginx_config`

2. Restart Nginx:
    ```SHELL
    sudo service nginx restart
    ```
3. Before starting up the main service, let make a folder for logs; the configuration for which is defined in `uwsgi.ini`
    ```SHELL
    sudo mkdir -p /var/log/uwsgi
    sudo chown -R thetechfreak:thetechfreak /var/log/uwsgi
    ```
    Instead of `thetechfreak` use your username.
4. Now, we will have to configure systemd service & for that we place `production_flask.service` in `/etc/systemd/system/production_flask.service` and then restart and enable the service to auto start after reboot.
    ```SHELL
    sudo systemctl start production_flask.service
    sudo systemctl enable production_flask.service
    ```
    At this point, our service should successfully start and incase of any updates you can just restart the service using:
    ```SHELL
    sudo systemctl restart production_flask.service
    ```

> ### After this step, your server should be up and running

## LOGGING & MONITORING

In order to check the logs, you can navigate to the logs directors `/var/log/uwsgi/`

Monitoring the service is also very easy as you justneed to go inside your `PROJECT_DIR` and run the following command:

```PYTHON
uwsgitop stats.production_flask.sock
```

OR

If you want to monitor the logs of the application itself, you can make use of `journalctl`: Note: These logs are same as the one which are stared in `/var/log/uwsgi/`; So, unless you really want to have a real time log on your system, it's not required:

```PYTHON
sudo journalctl -u production_flask.service -f
```

## Serving Static Files Using Nginx

If your application requires static files to be served, you can add the following rule inside `nginx_config` file:

```CONFIG
location /static {
    root APP_DIR;
}
```

Replace `APP_DIR` with your application directory and as a result, all your static files located at `APP_DIR/static` will be served by nginx

## FINAL THOUGHTS

These are my final thoughts and some notes which are worth noting.

1. UWSGI is configured in `lazy-apps` mode which is responsible for loading the application one time per worker. You can read more about it [here](http://uwsgi-docs.readthedocs.io/en/latest/articles/TheArtOfGracefulReloading.html)
2. Only Basic configuration of nginx is used; however it is sufficient for most of the use cases. But still, if you want to tune it further you can read up [here](https://docs.nginx.com/nginx/admin-guide/load-balancer/)
3. If your app requires some `parameters` to be passed, you can make use of [CONFIGPARSERS](https://docs.python.org/3/library/configparser.html) but DO-NOT pass it via command line as uWSGI is the one responsible for invoking the app.
4. In this app, `Socket permission` is given to everyone. You can adjust it as per your needs, but make sure that nginx and uWSGI can still talk to each other
5. If something goes wrong, the first place to check is the `log` files. By default, nginx writes error message to the file `/var/log/nginx/errors.log`

Contributions are very welcome.
Do make PULL requests if you want to change anything, or post an ISSUE if you encounter any bugs.
