## Install moodle on Raspberry Pi 5
    sudo apt-get update && sudo apt-get upgrade
    sudo apt-get install nginx mariadb-server git certbot python3-certbot-nginx
    sudo apt-get install php php-fpm php-mysql php-xml php-gd php-curl php-zip php-ldap php-intl php-soap php-mbstring php-xmlrpc php-bcmath php-cli php-zip php-json php-readline unzip
    sudo systemctl enable --now nginx mariadb php8.4-fpm

### Creade moodle database and user
Login with <code>sudo mariadb -u root -p</code> and then

Remove anonymous users

    DELETE FROM mysql.user WHERE User='';
    
Disallow remote root login

    DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1', '::1');
    
Remove test database

    DROP DATABASE IF EXISTS test;
    DELETE FROM mysql.db WHERE Db='test' OR Db='test\\_%';
    
Reload privilege tables

    FLUSH PRIVILEGES;
    
Create Database and user

    CREATE DATABASE moodle DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
    CREATE USER 'moodleuser'@'localhost' IDENTIFIED BY 'your_secure_password';
    GRANT ALL PRIVILEGES ON moodle.* TO 'moodleuser'@'localhost';
    FLUSH PRIVILEGES;
    EXIT;

### Get moodle
    cd /var/www
    sudo git clone -b MOODLE_501_STABLE git://git.moodle.org/moodle.git
    sudo mkdir /var/www/moodledata
    sudo chown -R www-data:www-data /var/www/moodle
    sudo chown -R www-data:www-data /var/www/moodledata
    sudo chmod 0777 /var/www/moodle
    sudo chmod 0777 /var/www/moodledata

### Configure nginx
    sudo nano /etc/nginx/sites-available/moodle

Configuration file (Important: root-directory as of Moodle 5.1 is /.../moodle/public)
<code>
server {
    listen 80;
    server_name moodle.lädi.org;
  
    root /var/www/moodle/public;
    index index.php index.html index.htm

    client_max_body_size 100M;
 
    location / { 
        try_files $uri $uri/ =404; 
    }
    location ~ [^/]\.php(/|$) {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php/php8.4-fpm.sock;  # Adjust if different PHP version
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
    }

    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        try_files $uri =404;
        expires max;
        log_not_found off;
    }
}
</code>

    sudo ln -s /etc/nginx/sites-available/moodle /etc/nginx/sites-enabled/
    sudo nginx -t
    sudo /etc/init.d/apache2 stop
    sudo rm /etc/nginx/sites-enabled/default
    sudo nano /etc/php/8.4/fpm/php.ini #Change line with max_input_vars = 5000    
    sudo chown -R root /var/www/moodle
    sudo chmod -R 0755 /var/www/moodle
    sudo systemctl restart nginx
