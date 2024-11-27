Jagger
===

Jagger (http://jagger.heanet.ie) is developed by HEAnet to manage the Edugate multiparty SAML federation. Other organisations use Jagger to manage their federations but it can be used to manage the web-of-trust for a single entity. It can also be used a a GUI for the Shibboleth SAML Identity Provider (www.shibboleth.net)

Features: 

1. Synchronise SAML metadata from another federation.
2. Create and manage a federation
3. Create a single circle of trust containing metadata of all entities that your organisation particpates via multiple federations.
4. GUI to manage to the attribute policy of identity providers based on the Shibboleth SAML implementation.
5. Filter the RequestedAttribute's of a SAML service provider to allow and IdP release attributes to such providers based on a policy set in the Jagger GUI.
6. Create and edit metadata of individual entities.
7. Notification subsystem with subscription options


Installation: check INSTALL.txt


Upgrades: UPGRADE.txt


Documentation (Admin guide) - http://jagger.heanet.ie/jaggerdocadmin/ (not final yet)

----

# HOWTO Install and Configure Jagger Federation Registry on Debian based Linux Distribution

## Requirements

### Hardware

-   CPU: 2 Core (64 bit)
-   RAM: 4 GB
-   HDD: 10 GB
-   OS:
    - Debian
    - Ubuntu

### Software

-   Apache Web Server
-   OpenSSL
-   Shibboleth Service Provider - Optionally
-   PHP

### Others

-   SSL Credentials: HTTPS Certificate & Private Key
-   Logo:
    -   size: 64px by 350px wide and 64px by 146px high
    -   format: PNG
    -   style: with a transparent background

## Notes

This HOWTO uses `example.org` and `jagger.example.org` as example values.

Please remember to **replace all occurencences** of:

-   the `example.org` value with the domain name
-   the `jagger.example.org` value with the Full Qualified Domain Name of the Jagger instance.

## Configure the environment

1.  Become ROOT:

    ``` text
    sudo su -
    ```

2.  Be sure that your firewall **is not blocking** the traffic on port **443** and **80** for the Jagger server.

3.  Set the SP hostname:

    **!!!ATTENTION!!!**: Replace `jagger.example.org` with your SP Full Qualified Domain Name and `<HOSTNAME>` with the Jagger hostname

    -   ``` text
        echo "<YOUR-SERVER-IP-ADDRESS> jagger.example.org <HOSTNAME>" >> /etc/hosts
        ```

    -   ``` text
        hostnamectl set-hostname <HOSTNAME>

## Configure APT Mirror

Debian Mirror List: <https://www.debian.org/mirror/list>

Ubuntu Mirror List: <https://launchpad.net/ubuntu/+archivemirrors>

Example with the Consortium GARR italian mirrors:

1.  Become ROOT:

    ``` text
    sudo su -
    ```

2.  Change the default mirror:

    -   Debian 12 - Deb822 file format:

        ``` text
        bash -c 'cat > /etc/apt/sources.list.d/garr.sources <<EOF
        Types: deb deb-src
        URIs: https://debian.mirror.garr.it/debian/
        Suites: bookworm bookworm-updates bookworm-backports
        Components: main

        Types: deb deb-src
        URIs: https://debian.mirror.garr.it/debian-security/
        Suites: bookworm-security
        Components: main
        EOF'
        ```

    -   Ubuntu:

        ``` text
        bash -c 'cat > /etc/apt/sources.list.d/garr.list <<EOF
        deb https://ubuntu.mirror.garr.it/ubuntu/ jammy main
        deb-src https://ubuntu.mirror.garr.it/ubuntu/ jammy main
        EOF'
        ```

3.  Update packages:

    ``` text
    apt update && apt-get upgrade -y --no-install-recommends
    ```

## Install Dependencies

``` text
apt install fail2ban vim wget ca-certificates openssl ntp git --no-install-recommends
```

## Install Mariadb Server
```text
apt install mariadb-server`
```

## Install MySQL Server

``` text
apt install default-mysql-server --no-install-recommends
```

### Protect MySQL Server

``` text
mysql_secure_installation
```
   
On Ubuntu:
  - Would you like to setup VALIDATE PASSWORD component? **No**
  - Remove anonymous users? **Yes**
  - Disallow root login remotely? **Yes**
  - Remove test database and access to it? **Yes**
  - Reload privilege tables now? **Yes**

On Debian:
  - Root password: **empty or a desired value for the root password of MariaDB**
  - Switch to unix_socket: **Y**
  - Change the root password? **N**
  - Remove anonymous users? **Y**
  - Disallow root login remotely? **Y**
  - Remove test database and access to it? **Y**
  - Reload privilege tables now? **Y**

## Install Apache Web Server

``` text
sudo apt install apache2
```

Enable the required Apache modules:

``` text
sudo a2enmod rewrit
```

``` text
sudo a2enmod unique_id
```

``` text
service apache2 restart
```

## Install Jagger

1. Install packages required:

   - Ubuntu:

     - ```txt
       apt install curl php libapache2-mod-php php-common php8.3-opcache php-gd php-curl php-mysql php-intl php-xml php-mbstring php-xmlrpc php-soap php-bcmath php-cli php-zip php-gearman php-apcu php-memcached php-amqplib python3-pip default-jdk gearman-job-server --no-install-recommends
       ```

   - Debian:

     - ```txt
       apt install curl php libapache2-mod-php php-common php8.3-opcache php-gd php-curl php-mysql php-intl php-xml php-mbstring php-xmlrpc php-soap php-bcmath php-cli php-zip php-gearman php-apcu php-memcached php-amqplib python3-pip default-jdk gearman-job-server --no-install-recommends
       ```

edit php settings as

   * `vim /etc/php/8.3/apache2/php.ini`
```php
date.timezone = Asia/Karachi
memory_limit = 256M
max_execution_time = 60
```
   * `vim /etc/php/8.3/cli/php.ini`

```php
date.timezone = Asia/Karachi
max_execution_time = 60
```

```txt
service apache2 restart
```

2. Install Composer:

   - ```txt
     curl -sS https://getcomposer.org/installer | php
     ```

   - ```txt
     cp composer.phar /usr/local/bin/composer
     ```

3. Install CodeIgniter:
   
   - ```txt
     wget https://github.com/bcit-ci/CodeIgniter/archive/refs/tags/3.1.13.tar.gz -O /opt/codeigniter-3.1.13.tar.gz
     ```
     
   - ```txt
     cd /opt
     ```
     
   - ```txt
     tar zxf /opt/codeigniter-3.1.13.tar.gz
     ```

   - ```txt
     mv /opt/CodeIgniter-3.1.13 /opt/codeigniter
     ```

4. Download Jagger:

   - ```txt
     git clone https://github.com/Edugate/Jagger /opt/rr3
     ```
     
5. Install required third parties libraries:

   - ```txt
     vim /opt/rr3/application/composer.json
     ```

     replace `"mtdowling/cron-expression": "1.1.*",` with `"dragonmantank/cron-expression": "3.*",`
     replace version of  `"laminas/laminas-permissions-acl":` with `"2.*",`
     
     and if there any issue Check the version of the following installed libraries on the server and change the version on this `composer.json` file.
      
   - ```txt
     cd /opt/rr3/application ; sudo composer install
     ```

7. Configure the "index.php" file:

   - ```text
     cp /opt/codeigniter/index.php /opt/rr3/
     vim /opt/rr3/index.php
     ```

     by setting `$system_path = '/opt/codeigniter/system'`.

     You may also want to set production environment. To do it find the line:

     `define('ENVIRONMENT', isset($_SERVER['CI_ENV']) ? $_SERVER['CI_ENV'] : 'development');`

     and change with

     `define('ENVIRONMENT', isset($_SERVER['CI_ENV']) ? $_SERVER['CI_ENV'] : 'production ');`

#### Configure Apache Virtual Host

Disable the default configuration
   * `cd /etc/apache2/sites-available/`
   * `a2dissite 000-default.conf`
   * `systemctl reload apache2`

Create a new conf file for RR3 as `rr3.conf`

Past and Edit `rr3.conf` with following

   * `vim rr3.conf`

```apache
<VirtualHost *:80>
 
        ServerName HOSTNAME.YOUR-DOMAIN.ac.lk 
        ServerAdmin noc@YOUR-DOMAIN.ac.lk 
        DocumentRoot /opt/rr3/
        Alias /rr3 /opt/rr3
<Directory /opt/rr3>

          Require all granted

          RewriteEngine On
          RewriteBase /rr3
          RewriteCond $1 !^(Shibboleth\.sso|index\.php|logos|signedmetadata|flags|images|app|schemas|fonts|styles|images|js|robots\.txt|pub|includes)
          RewriteRule  ^(.*)$ /rr3/index.php?/$1 [L]
  </Directory>
  <Directory /opt/rr3/application>
          Order allow,deny
          Deny from all
  </Directory>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Enable rr3 site by, 

   * `a2ensite rr3` 
   
and restart Apache 

   * `systemctl reload apache2`

## Configure Jagger database

```text
mysql -u root
```

- ```text
  CREATE DATABASE rr3 CHARACTER SET utf8 COLLATE utf8_general_ci;
  ```
- ```text
  CREATE USER 'rr3user'@'localhost' IDENTIFIED BY 'rr3pass';
  ```
- ```text
  GRANT ALL PRIVILEGES ON rr3.* TO rr3user@'localhost';
  ```
- ```text
  FLUSH PRIVILEGES;
  ```

## Configure Jagger

- ```text
  mkdir /var/log/rr3
  ```
  
- ```text
  chown www-data /var/log/rr3
  ```

- ```text
  chown www-data:www-data /opt/rr3/application /opt/rr3/application/models/Proxies
  ```
  
- ```text
  cd /opt/rr3
  ```

- ```text
  ./install.sh
  ```

- ```text
  cd /opt/rr3/application/config
  ```
  
- ```text
  cp config-default.php config.php
  ```

  `vim config.php` base configuration:

  - `$config['base_url'] = 'https://jagger.example.org/rr3';`
  - `$config['index_page'] = '';`
  - `$config['log_threshold'] = 1;`
  - `$config['log_path'] = '/var/log/rr3/';`
  - `$config['encryption_key'] = '<ENCRYPTION-KEY>';`

     `<ENCRYPTION-KEY>` generation:
    
     ```text
     tr -c -d '0123456789abcdefghijklmnopqrstuvwxyz' </dev/urandom | dd bs=32 count=1 2>/dev/null;echo
     ```

- ```text
  cp config_rr-default.php config_rr.php
  ```

  `vim config_rr.php` base configuration:

  - `$config['rr_setup_allowed'] = TRUE`  (HAS TO COME BACK to FALSE after Jagger setup)
  - `$config['site_logo'] = 'logo-default.png';`  (set filename to be used as main logo in top-left corner. File should be stored in `/opt/rr3/images/` folder.)
  - `$config['syncpass'] = <SYNCPASS>`
  - `$config['support_mailto'] = 'support@example.com';`
  
    `<SYNCPASS>` generation:
    
    ```text
    tr -c -d '0123456789abcdefghijklmnopqrstuvwxyz' </dev/urandom | dd bs=32 count=1 2>/dev/null;echo
    ```

  - `$config['gearman'] = TRUE;`
  - `$config['Shib_required'] = array('Shib_mail','Shib_username');`

    nameids - array of allowed NameID in JAGGER remove it from config
    ```php
    /*
    $config['nameids'] = array(
         'urn:mace:shibboleth:1.0:nameIdentifier' => 'urn:mace:shibboleth:1.0:nameIdentifier',
         'urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress' => 'urn:oasis:names:tc:SAML:1.1:nameid-format:emailAddress',
         'urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified'=>'urn:oasis:names:tc:SAML:1.1:nameid-format:unspecified',
         'urn:oasis:names:tc:SAML:2.0:nameid-format:transient' => 'urn:oasis:names:tc:SAML:2.0:nameid-format:transient',
         'urn:oasis:names:tc:SAML:2.0:nameid-format:persistent' => 'urn:oasis:names:tc:SAML:2.0:nameid-format:persistent',
                 );
    */
    ``` 
    > Note: The above config has been commented out!
  
- ```text
  cp database-default.php database.php
  ```

  `vim database.php` base Configuration:
  - `$db['default']['hostname'] = '127.0.0.1';`
  - `$db['default']['username'] = 'rr3user';`
  - `$db['default']['password'] = 'rr3pass';`
  - `$db['default']['database'] = 'rr3';`
  - `$db['default']['dsn']      = 'mysql:host=127.0.0.1;port=3306;dbname=rr3';`
  
- ```text
  cp email-default.php email.php
  ```
  
- ```text
  cp memcached-default.php memcached.php
  ```

## Populate database tables

- ```text
  cd /opt/rr3/application
  ```

- ```text
  ./doctrine
  ```

- ```text
  ./doctrine orm:schema-tool:create
  ```

- ```text
  ./doctrine orm:generate-proxies
  ```

### Install Letsencypt and enable https

```bash
add-apt-repository ppa:certbot/certbot
apt install python3-certbot-apache
certbot --apache -d YOUR-DOMAIN
```
```
Plugins selected: Authenticator apache, Installer apache
Enter email address (used for urgent renewal and security notices) (Enter 'c' to
cancel): YOU@YOUR-DOMAIN

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
agree in order to register with the ACME server at
https://acme-v02.api.letsencrypt.org/directory
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(A)gree/(C)ancel: A

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing to share your email address with the Electronic Frontier
Foundation, a founding partner of the Let's Encrypt project and the non-profit
organization that develops Certbot? We'd like to send you email about our work
encrypting the web, EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: Y

Obtaining a new certificate
Performing the following challenges:
http-01 challenge for YOUR_DOMAIN
Waiting for verification...
Cleaning up challenges
Created an SSL vhost at /etc/apache2/sites-available/rr3-le-ssl.conf
Enabled Apache socache_shmcb module
Enabled Apache ssl module
Deploying Certificate to VirtualHost /etc/apache2/sites-available/rr3-le-ssl.conf
Enabling available site: /etc/apache2/sites-available/rr3-le-ssl.conf

Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 2
Redirecting vhost in /etc/apache2/sites-enabled/rr3.conf to ssl vhost in /etc/apache2/sites-available/rr3-le-ssl.conf

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Congratulations! You have successfully enabled https://YOUR-DOMAIN

```
## By following these steps, you ensure the database is backed up, verified, and restored correctly.

Step 1: Check database name
```text
mysql -u root
```
```text
SHOW DATABASES;
```
Step 2: Create backup
```text
mysqldump -u root -p rr3 > /home/user/rr3_backup.sql
```

Step 3: Restore the database
```text
mysql -u root -p rr3 < /home/user/rr3_backup.sql
```

## Setup Jagger Registry

Go to https://jagger.example.org/rr3/setup and create the Admin user.

After that, set to FALSE the line:

`$config['rr_setup_allowed'] = TRUE`

on `/opt/rr3/application/config/config_rr.php`

## Documentation

https://jagger.heanet.ie/jaggerdocadmin/index.html

Here is a GitHub-compatible README format for the provided instructions, reorganized for clarity and consistency:

---

# PyFF Installation Guide for Ubuntu

This guide provides step-by-step instructions to install and configure PyFF on Ubuntu, including the setup of necessary OS packages, Python virtual environments, and configuration of metadata directories.

## Prerequisites

- Ubuntu
- Root access (`sudo`)
- FTP access to the server (IP: `103.4.92.6`) to fetch certificates and metadata files

## System Setup

### 1. Set Timezone to Asia/Karachi

```bash
sudo timedatectl set-timezone Asia/Karachi
```

### 2. Set Up Directories and Fetch Certificates

#### Create the required directories:

```bash
cd /opt
mkdir xmlsig
cd xmlsig
```

#### Fetch certificates via FTP:

```bash
ftp 103.4.92.6
get metadata-signer2.key
get metadata-signer2.pem
get metadata-signer2-rsa.key
bye
```

Check the files in `/opt/xmlsig`:

```bash
ls
# Expected output: metadata-signer2.key  metadata-signer2.pem  metadata-signer2-rsa.key
```

#### Set Up Signed Metadata Directory

```bash
cd /opt/rr3/
mkdir signedmetadata
cd signedmetadata
```

Fetch the certificate:

```bash
ftp 103.4.92.6
get metadata-signer2.pem
bye
```

## Installing PyFF

### 1. Install Required Packages

Start by installing essential OS packages:

```bash
sudo apt-get install build-essential python3-dev libxml2-dev libxslt1-dev libyaml-dev
sudo apt-get install python3-lxml python3-yaml python3-eventlet python3-setuptools
```

### 2. Install Python Virtual Environment

```bash
sudo apt-get install python3-virtualenv
```

### 3. Set Up PyFF Directory

```bash
cd /opt
mkdir pyff
```

### 4. Install PyFF in Virtual Environment

```bash
virtualenv /opt/pyff
source /opt/pyff/bin/activate
pip install pyFF
deactivate
```

## Configuration and Script Setup

### 1. Create Necessary Directories in `/opt/pyff`

```bash
cd /opt/pyff
mkdir metadata
mkdir scripts
mkdir certs
mkdir output
```

### 2. Fetch Scripts from FTP

```bash
cd scripts/
ftp 103.4.92.6
get export-script.sh
get run-pyff.sh
bye
```

Make the scripts executable:

```bash
chmod 777 export-script.sh
chmod 777 run-pyff.sh
```

### 3. Fetch Additional Certificates

```bash
cd .. # /opt/pyff/certs
ftp 103.4.92.6
get eduGAIN-signer-ca-new.pem
get eduGAIN-signer-ca.pem
bye
```

### 4. Fetch and Configure the `interfederation.fd` File

```bash
cd /opt/pyff
ftp 103.4.92.6
get interfederation.fd
```

Edit the first line of `interfederation.fd`:

```bash
vi interfederation.fd
# Change the first line to:
# /opt/pyff/metadata as edugain-md /opt/pyff/certs/eduGAIN-signer-ca-new.pem
:wq!
```

### 5. Run the Export Script

```bash
cd scripts/
./export-script.sh
```

### 6. Verify Metadata Files

- Check that the metadata file has been created in `/opt/pyff/metadata`.

```bash
cd /opt/pyff/metadata
ls
```

- Check that the signed metadata file has been created in `/opt/rr3/signedmetadata/`.

```bash
cd /opt/rr3/signedmetadata/
ls
```

---

# XMLSECTOOL Installation Guide

This guide explains how to install XMLSECTOOL on Ubuntu, configure the necessary environment (including Java), and create a script for signing metadata using XMLSECTOOL.

## Prerequisites

- Ubuntu (or similar)
- Root privileges (`sudo`)
- FTP access to fetch scripts and certificates
- Java (JRE/JDK)

## System Setup

### 1. Install Java (JRE/JDK)

#### Install OpenJDK (JRE)

Execute the following command to install the Java Runtime Environment (JRE) from OpenJDK:

```bash
sudo apt install default-jre
```

After installation, confirm the Java version:

```bash
java -version
```

#### Install OpenJDK (JDK)

To install the Java Development Kit (JDK) along with the JRE, use the following command:

```bash
sudo apt install default-jdk
```

Confirm the installation by checking the Java version:

```bash
java -version
```

### 2. Check JAVA Path

To verify the path to Java, use the following command:

```bash
update-alternatives --list java
# Expected output: /usr/lib/jvm/java-11-openjdk-amd64/bin/java
```

## Download and Install XMLSECTOOL

### 1. Download XMLSECTOOL

Navigate to the `/opt` directory:

```bash
cd /opt
```

Download the XMLSECTOOL package:

```bash
wget http://shibboleth.net/downloads/tools/xmlsectool/3.0.0/xmlsectool-3.0.0-bin.zip
```

### 2. Extract and Set Up XMLSECTOOL

Extract the downloaded ZIP file:

```bash
unzip xmlsectool-3.0.0-bin.zip
```

Remove the ZIP file after extraction:

```bash
rm xmlsectool-3.0.0-bin.zip
```

Rename the extracted directory to `xmlsectool` for consistency:

```bash
mv xmlsectool-3.0.0 xmlsectool
```

## Create and Configure the Signing Script

### 1. Fetch the Signing Script

Navigate to the `/opt/rr3/` directory:

```bash
cd /opt/rr3/
```

Download the `sign.sh` script via FTP:

```bash
ftp 103.4.92.6
get sign.sh
bye
```

### 2. Edit the `sign.sh` Script

Edit the `sign.sh` file to set the correct Java path and specify the necessary certificates and keys:

```bash
vi sign.sh
```

Add or modify the following lines in the script:

```bash
#!/bin/bash
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64

# Optional arguments
G=$1
H=$2
XMLSECTOOLDIR="/opt/xmlsectool"
SIGNCERT="/opt/xmlsig/metadata-signer2.pem"
SIGNKEY="/opt/xmlsig/metadata-signer2-rsa.key"
SIGNPASS="c@mpb3ll123"
RR3_PATH="/opt/rr3"
RR3_URL="http://rr.pern.edu.pk/rr3/tools/sync_metadata/metadataslist"
Y="/opt/rr3/tempfile"

# Navigate to XMLSECTOOL directory
cd ${XMLSECTOOLDIR}

# Fetch the metadata list based on arguments
if [ $G == "provider" ]; then
  wget --no-check-certificate -O /opt/rr3/tempfile ${RR3_URL}/${H}
else
  wget --no-check-certificate -O /opt/rr3/tempfile ${RR3_URL}
fi

# Process each line in the metadata list
for i in `cat ${Y}`; do
  group=`echo $i|awk -F ";" '{ print $1 }'|tr -d ' '`
  name=`echo $i|awk -F ";" '{ print $2 }'|tr -d ' '`
  srcurl=`echo $i|awk -F ";" '{ print $3 }'|tr -d ' '`

  dstoutput="/opt/rr3/signedmetadata/${group}/${name}"
  if [ ! -d "/opt/rr3" ]; then
    exit 3
  fi
  if [ ! -d "$dstoutput" ]; then
    mkdir -p $dstoutput
  fi

  # Sign the metadata
  ${XMLSECTOOLDIR}/xmlsectool.sh --sign --digest SHA-256 --referenceIdAttributeName ID \
    --certificate ${SIGNCERT} --key ${SIGNKEY} --outFile ${dstoutput}/metadata.xml --inUrl ${srcurl}
done

# Clean up
rm ${Y}
```

### 3. Finalize the Script

After SSL conversion, you might want to ensure that URLs in the script use `https` instead of `http`. Update the script accordingly where the URL is defined:

```bash
RR3_URL="https://rr.pern.edu.pk/rr3/tools/sync_metadata/metadataslist"
```

### 4. Make the Script Executable

Ensure the script is executable:

```bash
chmod +x /opt/rr3/sign.sh
```
