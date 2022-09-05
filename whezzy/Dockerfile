FROM debian:wheezy

# Install packages
RUN apt-get update && apt-get install -y debconf wget lsb-release
RUN debconf-set-selections << 'mysql-apt-config mysql-apt-config/select-preview select '
RUN debconf-set-selections << 'mysql-apt-config mysql-apt-config/select-product select Ok'
RUN debconf-set-selections << 'mysql-apt-config mysql-apt-config/select-server select mysql-5.6'
RUN debconf-set-selections << 'mysql-apt-config mysql-apt-config/select-tools select '
RUN debconf-set-selections << 'mysql-apt-config mysql-apt-config/unsupported-platform select abort'
ENV DEBIAN_FRONTEND noninteractive
RUN wget http://dev.mysql.com/get/mysql-apt-config_0.2.1-1debian7_all.deb --no-check-certificate
RUN DEBIAN_FRONTEND=noninteractive dpkg -i mysql-apt-config_0.2.1-1debian7_all.deb
RUN apt-get update
RUN DEBIAN_FRONTEND=noninteractive apt-get --force-yes -y install mysql-community-server
RUN apt-get install -y --force-yes supervisor git apache2 libapache2-mod-php5 php5-mysql pwgen php-apc php5-mcrypt 
RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf
  
# Add image configuration and scripts
ADD start-apache2.sh /start-apache2.sh
ADD start-mysqld.sh /start-mysqld.sh
ADD run.sh /run.sh
RUN chmod 755 /*.sh
ADD my.cnf /etc/mysql/conf.d/my.cnf
ADD supervisord-apache2.conf /etc/supervisor/conf.d/supervisord-apache2.conf
ADD supervisord-mysqld.conf /etc/supervisor/conf.d/supervisord-mysqld.conf
RUN chmod -R 755 /etc/supervisor/conf.d/* /etc/mysql/conf.d/my.cnf
RUN rm -rf /var/lib/mysql/*

# Add MySQL utils
ADD create_mysql_admin_user.sh /create_mysql_admin_user.sh
RUN chmod 755 /*.sh

# config to enable .htaccess
ADD apache_default /etc/apache2/sites-enabled/000-default.conf
RUN chmod -R 755 /etc/apache2/sites-enabled/*
RUN a2enmod rewrite

# Configure /app folder with sample app
RUN mkdir -p /app && rm -fr /var/www/html && ln -s /app /var/www/html
RUN echo "Hello! I'am a base image!" >> /app/index.html
RUN chmod -R 755 /app
RUN rm -f /var/www/index.htm* /etc/apache2/sites-enabled/000-default

#Environment variables to configure php
ENV PHP_UPLOAD_MAX_FILESIZE 10M
ENV PHP_POST_MAX_SIZE 10M

RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
RUN chmod +x /*.sh
EXPOSE 80 3306
CMD ["/run.sh"]