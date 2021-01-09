apt install software-properties-common
add-apt-repository universe
add-apt-repository universe
apt install curl apache2 composer fping git graphviz imagemagick libapache2-mod-php7.3 mariadb-client mariadb-server mtr-tiny nmap php7.3-cli php7.3-curl php7.3-gd php7.3-json php7.3-mbstring php7.3-mysql php7.3-snmp php7.3-xml php7.3-zip python-memcache python-mysqldb rrdtool snmp snmpd whois python3-pip acl

useradd librenms -d /opt/librenms -M -r
usermod -a -G librenms www-data

cd /opt
git clone https://github.com/librenms/librenms.git

chown -R librenms:librenms /opt/librenms
chmod 770 /opt/librenms
setfacl -d -m g::rwx /opt/librenms/rrd /opt/librenms/logs /opt/librenms/bootstrap/cache/ /opt/librenms/storage/
setfacl -R -m g::rwx /opt/librenms/rrd /opt/librenms/logs /opt/librenms/bootstrap/cache/ /opt/librenms/storage/

su - librenms
./scripts/composer_wrapper.php install --no-dev
exit

systemctl restart mysql
mysql -uroot -p

CREATE DATABASE librenms CHARACTER SET utf8 COLLATE utf8_unicode_ci;
CREATE USER 'librenms'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON librenms.* TO 'librenms'@'localhost';
FLUSH PRIVILEGES;
exit

nano /etc/mysql/mariadb.conf.d/50-server.cnf

innodb_file_per_table=1
lower_case_table_names=0

systemctl restart mysql

nano /etc/php/7.3/apache2/php.ini
	[Date]
	date.timezone = Europe/Oslo 


    a2enmod php7.3
	a2dismod mpm_event
	a2enmod mpm_prefork

nano /etc/php/7.3/cli/php.ini
	[Date]
	date.timezone = Europe/Oslo

    a2enmod php7.3
	a2dismod mpm_event
	a2enmod mpm_prefork
	


nano /etc/apache2/sites-available/librenms.conf

<VirtualHost *:80>
  DocumentRoot /opt/librenms/html/
  ServerName  librenms.kvalheim.net

  AllowEncodedSlashes NoDecode
  <Directory "/opt/librenms/html/">
    Require all granted
    AllowOverride All
    Options FollowSymLinks MultiViews
  </Directory>
</VirtualHost>

a2ensite librenms.conf
a2enmod rewrite

#chown librenms:librenms /opt/librenms/config.php

systemctl restart apache2

cp /opt/librenms/snmpd.conf.example /etc/snmp/snmpd.conf
nano /etc/snmp/snmpd.conf
   public

curl -o /usr/bin/distro https://raw.githubusercontent.com/librenms/librenms-agent/master/snmp/distro
chmod +x /usr/bin/distro
systemctl restart snmpd


cp /opt/librenms/librenms.nonroot.cron /etc/cron.d/librenms
cp /opt/librenms/misc/librenms.logrotate /etc/logrotate.d/librenms



cd /opt/librenms
./validate.php






