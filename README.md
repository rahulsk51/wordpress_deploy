# wordpress_deploy

Technical Assignment for WordPress site.

Prerequisites for this assignment:
1.	Cloud Server: AWS EC2
2.	GitHub Account:
3.	Site Domain name: 

Server Provisioning:
Step 1:  Install LEMP server (Nginx, MySQL, and PHP).
# sudo apt install nginx mysql-server php-fpm php-mysql
 

Step 2: Configure MySQL for WordPress.
login to mysql DB
#sudo mysql -u root -p
 

After that create a new SQL Database and user for WordPress.
===
CREATE DATABASE wordpress;
CREATE USER 'wordpressuser'@'localhost' IDENTIFIED BY 'your_password';
GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpressuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
 
====

Step 3: Download and Configure WordPress
Navigate to the Nginx web root directory:
#cd /var/www/html

Download the latest version of WordPress:
#sudo wget https://wordpress.org/latest.tar.gz

Extract the downloaded archive
#sudo tar -zxvf latest.tar.gz

Move the WordPress files to the web root:
#sudo mv wordpress/* .
#sudo rm -r wordpress

Set the correct permissions:
#sudo chown -R www-data:www-data /var/www/html/
#sudo chmod -R 755 /var/www/html/

Step 4:Configure Nginx
Create a new server block configuration file:
#sudo vi /etc/nginx/sites-available/wordpress
 

Add the following configuration:
==
server {
    listen 80;
    server_name webforall.myvnc.com
    root /var/www/html/wordpress;
    index index.php index.html index.htm;
    location / {
        try_files $uri $uri/ /index.php?$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
    location ~ /\.ht {
        deny all;
    }
    error_log /var/log/nginx/wordpress_error.log;
    access_log /var/log/nginx/wordpress_access.log;
}
==
create a symbolic link to enable the configuration:
#sudo ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled/

Test the Nginx configuration:
#sudo nginx -t

Website Configuration:
Step 1: Install WordPress
Step 2: Secure WordPress
Step 3: Implement SSL/TLS with Let's Encrypt
install Certbot, Obtain SSL/TLS Certificate and Configure Nginx for SSL.
#sudo apt install certbot python3-certbot-nginx
#sudo certbot --nginx -d webforall.myvnc.com
#sudo certbot certificates

Step 4: Optimize Nginx Configuration (Add the below mentioned configuration in nginx configuration)
Enable Gzip Compression:
==
gzip on;
gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
==

GitHub Repository Setup:
Step 1: Create a GitHub Repository
Go to GitHub (https://github.com/) and log in to your account.

Step 2: Configure GitHub Actions Workflow
1. Inside your repository, click on the "Actions" tab.
2. Select "Set up a workflow yourself."
3. In the editor, define the workflow file (.github/workflows/main.yml):
====
name: Wordpress Deployment

# Trigger deployment only on push to main branch
on:
  push:
    branches:
      - main

jobs:
  deploy:
    name: Deploy to EC2 on main branch push
    runs-on: ubuntu-latest

    steps:
      - name: Checkout the files
        uses: actions/checkout@v2

      - name: Deploy to Server 1
        uses: easingthemes/ssh-deploy@main
        env:
          SSH_PRIVATE_KEY: ${{ secrets.EC2_SSH_KEY }}
          REMOTE_HOST: ${{ secrets.HOST_DNS }}
          REMOTE_USER: ${{ secrets.USERNAME }}
          TARGET: ${{ secrets.TARGET_DIR }}

      - name: Executing remote ssh commands using ssh key
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.HOST_DNS }}
          username: ${{ secrets.USERNAME }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            sudo apt-get -y update
            cd home
            sudo mv * /var/www/html
            chown -R www-data:www-data /var/www/html/*			

=====

Step 3: Set Up Secrets
1. Go to the main page of your GitHub repository.
2. Click on "Settings" and then on "Secrets."
3. Add the following secrets:
          HOST: The IP address or domain of your VPS.
          USERNAME: The username for SSH access to your VPS.
          SSH_PRIVATE_KEY: The private key used for SSH authentication

Step 4: Commit and Push
Commit the changes to your repository and push them to GitHub.
===
#git add .
#git commit -m "Initial setup: GitHub Actions workflow and README"
#git push origin main
===
