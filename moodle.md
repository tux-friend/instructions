## Install moodle on Raspberry Pi 5
    sudo apt-get update && sudo apt-get upgrade
    sudo apt-get install nginx mariadb-server git certbot python3-certbot-nginx
    sudo apt-get install php php-fpm php-mysql php-xml php-gd php-curl php-zip php-ldap php-intl php-soap php-mbstring php-xmlrpc php-bcmath php-cli php-zip php-json php-readline unzip
    sudo systemctl enable --now nginx mariadb php8.4-fpm

### Creade moodle database and user
Login with <code>sudo mariadb -u root -p</code> and then

    -- Remove anonymous users
    DELETE FROM mysql.user WHERE User='';
    -- Disallow remote root login
    DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1', '::1');
    -- Remove test database
    DROP DATABASE IF EXISTS test;
    DELETE FROM mysql.db WHERE Db='test' OR Db='test\\_%';
    -- Reload privilege tables
    FLUSH PRIVILEGES;
    -- Create Database and user
    CREATE DATABASE moodle DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
    CREATE USER 'moodleuser'@'localhost' IDENTIFIED BY 'your_secure_password';
    GRANT ALL PRIVILEGES ON moodle.* TO 'moodleuser'@'localhost';
    FLUSH PRIVILEGES;
    EXIT;


