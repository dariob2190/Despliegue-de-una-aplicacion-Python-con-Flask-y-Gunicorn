Vagrant.configure("2") do |config|
  config.vm.box = "debian/bullseye64"
  config.vm.network "forwarded_port", guest: 80, host: 8080
  config.vm.network "forwarded_port", guest: 5000, host: 5000
  config.vm.network "private_network", ip: "192.168.2.45"
  config.vm.synced_folder ".", "/vagrant"

  config.vm.provision "shell", inline: <<-SHELL
    # Al aprovisionar desde aquí sólo tiene necesita este fichero
    apt-get update
    apt-get install -y python3-pip nginx git
    pip3 install pipenv python-dotenv

    mkdir -p /var/www/app
    chown -R vagrant:www-data /var/www/app
    chmod -R 775 /var/www/app

    cd /var/www/app
    echo "FLASK_APP=wsgi.py" > .env
    echo "FLASK_ENV=production" >> .env

    echo "from flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
    '''Index page route'''
    return '<h1>App desplegada</h1>'" > application.py

    echo "from application import app
if __name__ == '__main__':
   app.run(debug=False)" > wsgi.py

    su - vagrant -c "cd /var/www/app && pipenv install flask gunicorn"

    echo "[Unit]
Description=flask app service
After=network.target

[Service]
User=vagrant
Group=www-data
WorkingDirectory=/var/www/app
Environment='PATH=/usr/local/bin:/usr/bin:/bin'
ExecStart=/usr/local/bin/pipenv run gunicorn --workers 3 --bind unix:/var/www/app/app.sock wsgi:app

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/flask_app.service

    systemctl daemon-reload
    systemctl enable flask_app
    systemctl start flask_app

    echo "server {
    listen 80;
    server_name localhost;

    location / {
        include proxy_params;
        proxy_pass http://unix:/var/www/app/app.sock;
    }
}" > /etc/nginx/sites-available/app.conf

    ln -s /etc/nginx/sites-available/app.conf /etc/nginx/sites-enabled/
    rm /etc/nginx/sites-enabled/default
    systemctl restart nginx

    echo "Aprovisionamiento completado"
  SHELL
end