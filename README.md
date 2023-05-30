# rails_nginx_phusion_passenger
## Install ruby runtime using Rbenv
```sh
sudo apt update
sudo apt install git curl libssl-dev libreadline-dev zlib1g-dev autoconf bison build-essential libyaml-dev libreadline-dev libncurses5-dev libffi-dev libgdbm-dev
curl -fsSL https://github.com/rbenv/rbenv-installer/raw/HEAD/bin/rbenv-installer | bash
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
source ~/.bashrc
type rbenv
rbenv install 3.2.1
rbenv global 3.2.1
ruby -v
```

## Install nginx
```sh
sudo apt-get install nginx -y
```

## Install Passenger
```sh
sudo apt-get install -y dirmngr gnupg
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 561F9B9CAC40B2F7
sudo apt-get install -y apt-transport-https ca-certificates
sudo sh -c 'echo deb https://oss-binaries.phusionpassenger.com/apt/passenger bionic main > /etc/apt/sources.list.d/passenger.list'
sudo apt-get update
```

```sh
sudo apt-get install -y libnginx-mod-http-passenger
```
If error, try this
```sh
echo "deb https://oss-binaries.phusionpassenger.com/apt/passenger focal main" | sudo tee /etc/apt/sources.list.d/passenger.list
sudo apt update
```

then
```sh
if [ ! -f /etc/nginx/modules-enabled/50-mod-http-passenger.conf ]; then sudo ln -s /usr/share/nginx/modules-available/mod-http-passenger.load /etc/nginx/modules-enabled/50-mod-http-passenger.conf ; fi
sudo ls /etc/nginx/conf.d/mod-http-passenger.conf
which ruby
sudo vim /etc/nginx/conf.d/mod-http-passenger.conf
```

copy path of ruby which return by command
```sh
which ruby
```

Replace path of ruby from above to "passenger_ruby"
```sh
sudo vim /etc/nginx/conf.d/mod-http-passenger.conf
```

```sh
sudo service nginx restart
```

Check installation
```sh
sudo /usr/bin/passenger-config validate-install
sudo /usr/sbin/passenger-memory-stats
```

Next, create directory wanna deploy code to
```sh
sudo mkdir -p /var/www/my_project
```

Config nginx
```sh
sudo vim /etc/nginx/sites-enabled/my_project
```
Paste content to that file
```
server {
  listen 80;
  server_name 18.197.76.76;
  root /var/www/my_project/public;
  rails_env staging;
  passenger_enabled on;
  passenger_ruby /home/ubuntu/.rbenv/shims/ruby;
}
```
Here, we can change
- passenger_ruby is path to ruby by command which ruby
- server_name is domain OR public URL of EC2
- rails_env replace with rails environment want run

Delete file config default of nginx then restart nginx
```sh
sudo rm /etc/nginx/sites-enabled/default
sudo service nginx restart
```

## Install database
```sh
sudo apt update
sudo apt install mysql-server -y
sudo systemctl start mysql.service
sudo mysql
```

Run command inside mysql console
```sh
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'replacepasswordhere';
exit
```

For sure, we can check login mysql with password has been set, in this case, password is 'replacepasswordhere'
```sh
mysql -u root -p
```

Install package for gem mysql
```sh
sudo apt-get install libmysqlclient-dev -y
```

## Deploy
At this step, we can deploy code by CD, capistrano, git clone,...to directory /var/www/my_project
For sure config of rails like: database.yml,...etc

## Restart app Passenger
```sh
passenger-config restart-app /var/www/my_project
```

=> ALL STEP DONE, open browser and check
