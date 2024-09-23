# Personal WSL2 Ubuntu Setup
This is my setup for local Ubuntu 22.04 using WSL2 where I store most of my repositories under `/projects/`. This will be updated time to time.

## Necessary Tools
### NVM
Taken from [here](https://github.com/nvm-sh/nvm).
`curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash`

### ZSH
1. `sudo apt install zsh`
2. `sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"`
3. `git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k`
4. `nano ~/.zshrc`
5. Replace everything with this [here](https://github.com/azri-cs/wsl2-setup/blob/main/.zshrc). The content might be different after running powerlevel10k configuration wizard.
6. Restart terminal.

Personal p10k config: `yyyy3122212223221y1`

### PPA from Ondřej Surý for handling multiple PHP versions
1. `sudo apt install software-properties-common && sudo add-apt-repository ppa:ondrej/php && sudo apt update`
2. `sudo apt-get install php8.3 php8.3-fpm && sudo apt-get install php8.3-mysql php8.3-mbstring php8.3-xml php8.3-gd php8.3-curl php8.3-zip php8.3-intl`

Switching PHP version: `sudo update-alternatives --config php`

### MySQL/PostgreSQL
Taken from [here](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-22-04).
1. `sudo apt install mysql-server`
2. `sudo systemctl start mysql.service`
3. `sudo mysql_secure_installation`

MySQL No Root Password error?
1. `sudo mysql`
2. Inside MySQL, replace `password` with something else: `ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';`

### Python
Ubuntu 22.04 ship with Python 3 pre-installed, so we only need to configure further. Further instructions [here](https://www.digitalocean.com/community/tutorials/how-to-install-python-3-and-set-up-a-programming-environment-on-ubuntu-22-04).
1. `sudo apt install -y python3-pip`
2. `sudo apt install -y build-essential libssl-dev libffi-dev python3-dev`
3. (optional) install python packages by replacing `package_name`: `pip3 install package_name`

### Ruby
`sudo apt install ruby-full`

### Make
`sudo apt install make`

### Composer
Replace the installer checksum (SHA-384) from [here](https://composer.github.io/pubkeys.html).
1. `php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"`
2. `php -r "if (hash_file('sha384', 'composer-setup.php') === 'dac665fdc30fdd8ec78b38b9800061b4150413ff2e3b6f88543c636f7cd84f6db9189d43a81e5503cda447da73c7e5b6') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"`
3. `php composer-setup.php`
4. `php -r "unlink('composer-setup.php');"`

### Global Laravel Create
1. `composer global require laravel/installer`
2. Now you can create new project using `laravel new project-name`.

### Redis Server
1. `sudo apt-get install lsb-release curl gpg`
2. `curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg`
3. `sudo chmod 644 /usr/share/keyrings/redis-archive-keyring.gpg`
4. `echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list`
5. `sudo apt-get update && sudo apt-get install redis`
6. `systemctl status redis.service` to check its status, it should be in green "active (running)".
7. After installing supervisor, create supervisord config file for redis-server at `/etc/supervisor/conf.d`. Refer `redis-supervisor.conf`.

### Supervisor
1. `sudo apt update && sudo apt install supervisor`
2. `sudo systemctl status supervisor` to check its status, it should be in green "active (running)".
3. After creating new config file for a program, run these `sudo supervisorctl reread && sudo supervisorctl update`  to retrieve latest config files & insert them to process group.
4. `sudo supervisorctl start "redis:*"`
5. Have `redis: ERROR (spawn error)` error? Ensure ownership of `/var/log/redis/redis.log` is correct: `sudo ls -l /var/log/redis/redis.log`. If not 'redis' user, run this `sudo chown redis:redis /var/log/redis/redis.log`.
