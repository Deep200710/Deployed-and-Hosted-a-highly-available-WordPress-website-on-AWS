# Deployed and Hosted a highly available WordPress website on AWS

Hosted a highly available WordPress website on AWS EC2

Connected with RDS database service

Publically accessible over the internet


## Resources

Custom VPC

EC2 Instance

Elastic File System (EFS)

Relational Database Service (RDS) (MariaDB)

Elastic Load Balancer (ELB)

ElastiCache

S3 Bucket

AMI

CloudFront (Content delivery network)





## Architecture Diagram


<img width="781" alt="wordpress aws" src="https://user-images.githubusercontent.com/91826247/210163114-5dd90d62-563d-4713-9b5c-dec9e20753ef.png">


## Create a new VPC

- Open the Amazon VPC console
- On the VPC Dashboard, choose Launch VPC 
- Enter the following information
    
    VPC name (wp-vpc)

    IPv4 CIDR block (10.0.0.0/20)

- Choose Create VPC


![vpc (2)](https://user-images.githubusercontent.com/91826247/214017680-79cdf929-751a-47b3-a0a5-38a7e3d4ea19.png)


## To create a subnet (Public/Private)

- Click Create Subnet

    Select your VPC (wp-vpc)

    Select any Availability Zone (ap-south-1a)

    CIDR Block (10.0.0.0/24)

    Name tag (Public-Subnet-ap-south-1a)
- Click Yes, Create Subnet
- Similarly create a Private Subnet
    
    Select your VPC (wp-vpc)

    Select any Availability Zone (ap-south-1a)

    CIDR Block (10.0.1.0/24)

    Name tag (Private-Subnet-ap-south-1a)
    
    ![Screenshot (91)](https://user-images.githubusercontent.com/91826247/214017927-682eebc1-ee7c-4d7c-8dc1-8e6fc16e19e4.png)

    ![Screenshot (92)](https://user-images.githubusercontent.com/91826247/214017989-848eed4c-b023-414f-ab91-535b95a13df8.png)


##  Create a custom route table for Public/Private Subnet

- Open the Route tables
- Choose Create route table
- For Name, enter a name for your route table (eg. Public-RT/Private-RT)
- Choose your VPC (wp-vpc)
- Choose Create route table
![Screenshot (93)](https://user-images.githubusercontent.com/91826247/214018125-830660c5-e198-46fa-96e3-a3240c0eb896.png)

![Screenshot (94)](https://user-images.githubusercontent.com/91826247/214018134-f1e5d4da-091a-49cb-a6af-e3993e34fb8a.png)
![Screenshot (95)](https://user-images.githubusercontent.com/91826247/214018147-af901d78-5f6c-4de4-aa71-69cffd630716.png)


## To create an internet gateway and attach it to your VPC

- Open  and choose Create internet gateway
- Name your internet gateway (wp-vpc-igw)
- Choose Create internet gateway
- Select the internet gateway that you just created, and then choose Actions, Attach to VPC

![Screenshot (96)](https://user-images.githubusercontent.com/91826247/214018216-9cdf9880-7cf1-4420-9784-8ede3bb13598.png)

![Screenshot (97)](https://user-images.githubusercontent.com/91826247/214018232-1e8c9e92-4fdd-41c7-8aa7-5f0f76d68c4b.png)

![Screenshot (98)](https://user-images.githubusercontent.com/91826247/214018243-e4557532-93ce-450b-bd16-2a99a68ba843.png)



## Create a MariaDB Database

- Choose Create database
- Select your engine (MariaDB) and DB engine version
- Settings:
    
    DB instance identifier : Name for the DB instance

    Master username : Username that you will use to log in to your DB instance

    Master password : Type a password for your master user

- Select DB instance class
- Port : Leave the default value of 3306.
- Click Create database

![Screenshot (109)](https://user-images.githubusercontent.com/91826247/214018447-0cb9fe77-c13f-4e68-a103-dc0dde11ccc6.png)

![Screenshot (110)](https://user-images.githubusercontent.com/91826247/214018463-69d63a1f-b813-439a-9cea-191c65c2467a.png)


## Create Elastic File System 

- Open the EFS and Create file system 
- Choose Customize to create a customized file system

    Enter a Name for the file system

    Choose Standard Storage class to create a file system 

    Choose One Zone to create a file system that uses the EFS One Zone


![Screenshot (114)](https://user-images.githubusercontent.com/91826247/214018642-071fc89d-db99-459f-8265-83653e7e9275.png)

![Screenshot (113)](https://user-images.githubusercontent.com/91826247/214018656-5cc51878-96c2-46b4-ad5e-831eed69b6ea.png)

![Screenshot (112)](https://user-images.githubusercontent.com/91826247/214018714-2d125b67-aef5-415c-b679-07f8b6746de9.png)

- Configure network access

    Choose the VPC (wp-vpc)

- For Mount target 

    Choose Availability Zone

    Subnet ID : Choose from the available subnets in an Availability Zone

- Security groups
- Review and create 

## Create ElastiCache Redis Cluster
- Open the ElastiCache console
- Choose Create cluster and then choose Create Redis cluster 
- Create new subnet group (Redis-Subnet)
- Select vpc (wp-vpc)
- Select all subnet in vpc 
- Review and create Redis cluster



![Screenshot (111)](https://user-images.githubusercontent.com/91826247/214018975-6047d7ec-94d3-4ee9-b4e6-cdd71589f93d.png)


## Create NAT instances

- Open the Amazon EC2 console and Launch Instance 
- Choose an Amazon Machine Image (AMI) 
- Select the VPC (wp-vpc) and select your public subnet (Public-subnet-ap-south-1a)
- Enable auto-assign Public IP
- Review and Launch instance

## Create Master-server instances

- Open the Amazon EC2 console and Launch Instance 
- Choose an Amazon Machine Image (AMI) as Ubuntu
- Select the VPC (wp-vpc) and select your public subnet (Public-subnet-ap-south-1a)
- Enable auto-assign Public IP
- Review and Launch instance

![Screenshot (76)](https://user-images.githubusercontent.com/91826247/214019483-224a0871-3303-49a0-9f22-07c5a66df5d8.png)


## Edit Route tables 
- Select Public-RT
   
    Subnet associations and Edit subnet associations

    Select Public subnet (Public-Subnet-1a)

    Edit routes

    Add route : Destination (0.0.0.0/0) 
    Target (Internet Gateway) (wp-vpc-igw)

- Select Private-RT
 
    Subnet associations and Edit subnet associations

    elect Private subnet (Private-Subnet-1a)

    Edit routes

    Add route : Destination (0.0.0.0/0)
    Target (instance) (NAT instance)

## Connect Master-server EC2 instance using SSH

 - Setup
   
   Create a directory 
```
   mkdir /var/www/html
``` 
   
- Install nfs-common
```
  apt-get install nfs-common -y
  ``` 
        
- Mount NFS client
```
   mount -t nfs4 -o krb5p <DNS_NAME>:/ /var/www/html
``` 


- Add permanent mount for EFS

  `vi /etc/fstab`

  ```
  <mount-target-DNS>:/ /var/www/html nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 0 0
   ```

    Reboot system

## wordpress setup on master server

```
apt-get install apache2 libapache2-mod-php -y
systemctl status apcahe2
systemctl enable apcahe2
 ```

```
vi /etc/apache2/sites-available/wordpress.conf
```
- Apache virtual host setup

```
<VirtualHost *:80>

    ServerName wordpress.cloudip.online

    ServerAdmin webmaster@cloudip.online
    DocumentRoot /var/www/html/wordpress

<Directory /var/www/html/wordpress>
        Options Indexes FollowSymLinks MultiViews
        AllowOverride All
        Order allow,deny
        allow from all
        Require all granted
    </Directory>

</VirtualHost>
```

- Enable site with apache
  
   `a2ensite  wordpress.conf`

  Enable mod rewrite

   `a2enmod rewrite`


  Install PHP and extensions

  ```
   sudo apt-get install php php-mysql libapache2-mod-php php-cli php-cgi php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip php-curl php-imagick
    ```

- Download Wordpress tar file

  `wget https://wordpress.org/latest.tar.gz`

   Extract file

    `tar -xvf latest.tar.gz`

  Paste your public ip and setup Wordpress 

    `apt-get install php-mysql `
 
   ```
  sudo apt-get install php php-mysql libapache2-mod-php php-cli php-cgi php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip php-curl php-imagick -y
    ```

  Set directory permissions

   ```
   chown -R www-data:www-data  /var/www/html/wordpress
   chmod -R 755 /var/www/html/wordpress

    ```
 
  Change directory to wordpress 
  
  `vi wp-config.php`

  ```
   <?php
    /**
    * The base configuration for WordPress
    *
    * The wp-config.php creation script uses this file during the installation.
    * You don't have to use the web site, you can copy this file to "wp-config.php"
    * and fill in the values.
    *
    * This file contains the following configurations:
    *
    * * Database settings
    * * Secret keys
    * * Database table prefix
    * * ABSPATH
    *
    * @link https://wordpress.org/support/article/editing-wp-config-php/
    *
    * @package WordPress
    */

    // ** Database settings - You can get this info from your web host ** //
    /** The name of the database for WordPress */
    define( 'DB_NAME', 'wordpress' );

    /** Database username */
    define( 'DB_USER', 'wordpress' );

    /** Database password */
    define( 'DB_PASSWORD', 'wordpress' );

    /** Database hostname */
    define( 'DB_HOST', 'wp-database-server.ctry4zdczbkq.ap-south-1.rds.amazonaws.com' );

    /** Database charset to use in creating database tables. */
    define( 'DB_CHARSET', 'utf8mb4' );

    /** The database collate type. Don't change this if in doubt. */
    define( 'DB_COLLATE', '' );

    /**#@+
    * Authentication unique keys and salts.
    *
    * Change these to different unique phrases! You can generate these using
    * the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}.
    *
    * You can change these at any point in time to invalidate all existing cookies.
    * This will force all users to have to log in again.
    *
    * @since 2.6.0
    */
    define( 'AUTH_KEY',         'y^iFT=pT?2HiI@MncY}:ZR;i6%K^Xl_YQ~F)Ql&<V}}-zI~$j*:GLuA3x&l(%:ap' );
    define( 'SECURE_AUTH_KEY',  'Y&UE#_:J#3[tRHhZ,;I>o^BL)=rm^_9&.rnmMh=arQ^N0qrQUjst!$iW#I|GOh}p' );
    define( 'LOGGED_IN_KEY',    '^3EjfW.9uuyF/2y(IFY@ZF<<&7#eM3X^Q{2N(WK5p!!haJuWFn)k=7+_MKH`{cTI' );
    define( 'NONCE_KEY',        '&{? m:nBr`^gbe29b6nUl/PP_y0;#K:E{OzMl1}]I& -]vdm1n@5=#4z7z;*T8s}' );
    define( 'AUTH_SALT',        'Uw9O~BWF>q]g}TCF#DM|tIVVq&:7=)`p&zP%-2O^7>L9ZT{/;EYmgEld4#t$5rR=' );
    define( 'SECURE_AUTH_SALT', ',Xr!]?hU|rm70/,~`R,mxNa6Qpp+%EDxU8|mr/vUY>OcV u_P/5|O,.5[`|)k}ji' );
    define( 'LOGGED_IN_SALT',   '3tQNfl;v+C|Ug*c,w4#SvI&ZW?gtU4]`6IX~`VHKe2<v;c qt.^*vsO wa>`%1&c' );
    define( 'NONCE_SALT',       'qJgGTGn!KFL!=#-M]uje76k=^+^_H_ ?u*S!,uI#mkT*<l,9@8w#buYF.ZP5Iw5}' );

    /**#@-*/

    /**
    * WordPress database table prefix.
    *
    * You can have multiple installations in one database if you give each
    * a unique prefix. Only numbers, letters, and underscores please!
    */
    $table_prefix = 'wp_';

    /**
    * For developers: WordPress debugging mode.
    *
    * Change this to true to enable the display of notices during development.
    * It is strongly recommended that plugin and theme developers use WP_DEBUG
    * in their development environments.
    *
    * For information on other constants that can be used for debugging,
    * visit the documentation.
    *
    * @link https://wordpress.org/support/article/debugging-in-wordpress/
    */
    define( 'WP_DEBUG', false );

    /* Add any custom values between this line and the "stop editing" line. */



    /* That's all, stop editing! Happy publishing. */

    /** Absolute path to the WordPress directory. */
    if ( ! defined( 'ABSPATH' ) ) {
        define( 'ABSPATH', __DIR__ . '/' );
    }

    /** Sets up WordPress vars and included files. */
    require_once ABSPATH . 'wp-settings.php'   
     ```

  Cd 
   
   `cd /etc/php/8.1/apache2`

   `vi php.ini`  

   `upload_max_filesize = 500M`

   `max_execution_time = 300`

   `post_max_time = 80M`

   Install Redis Object Cache extension

   Define Redis host in 
   
   `vi wp-config.php`

  ```
   define('WP_REDIS_HOST', 'PUT YOUR REDIS INSTANCE ENDPOINT HERE');
   define('WP_CACHE', true);
  ```
- Make S3 bucket for media access
    
    open the S3 console

    Choose Create bucket

    In Bucket name, enter a unique name for your bucket

    Disable Block Public Access

    Create bucket
  
   Go to Bucket policy ->

  ```
   {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::YOURBUCKETNAME/*"
        }
      ]
    }
   ```
   ![Screenshot (125)](https://user-images.githubusercontent.com/91826247/214020049-564bcba5-6ba8-4961-bd1d-07312942c8bf.png)

   ![Screenshot (126)](https://user-images.githubusercontent.com/91826247/214020056-cb29f960-4b28-438b-943f-bc3ac205a69d.png)

   ![Screenshot (127)](https://user-images.githubusercontent.com/91826247/214020074-7dece6f3-057d-4702-8fba-5176225d3662.png)

- Create cloudfront disribiution

  Choose s3 bucket as a origin and create

![Screenshot (128)](https://user-images.githubusercontent.com/91826247/214020177-63b4bcc6-8a41-4a4e-a156-5e4c14837ce4.png)


![Screenshot (129)](https://user-images.githubusercontent.com/91826247/214020182-895649c2-9dca-47a6-b630-e192f39c74f8.png)
![Screenshot (130)](https://user-images.githubusercontent.com/91826247/214020193-c8435347-2e1f-4f25-92b6-c09cabc5bc9a.png)

  install aws cli tool

  `apt install awscli -y`

  cd `/var/www/html/wordpress/wp-content/uploads/`

  `aws s3 sync --delete /var/www/html/wordpress/wp-content/uploads s3://YOURBUCKETNAME`

  CRON JOB for syncing local images to s3 bucket

  `crontab -e`

  `*/1 * * * * aws s3 sync --delete /var/www/html/wordpress/wp-content/uploads s3://YOURBUCKETNAME`

   `vi .htaccess` for rewriting urls to cloudfront CDN

   
    Options +FollowSymlinks
 
    RewriteEngine on
 
    rewriterule ^wp-content/uploads/(.*)$ https://cloudfront Domain name/$1 [r=301,nc]
    

- Create AMI of master server

    Right-click the instance (master server) you want to use as the your AMI, and choose Create Image

     Type a name and description, and then choose Create Image
     
     
     ![Screenshot (135)](https://user-images.githubusercontent.com/91826247/214022818-79c51449-1ae9-4943-9dc9-c033c4181eb7.png)

     


- Create a  Load Balancer 

    Open EC2 console Choose Create Load Balancer

    Choose application load balancer

    Type a name for your load balancer 

    Select a VPC (wp-vpc)

    Select available public and private subnet

    Assign security groups to your load balancer 

    Create target group

    Choose Create
    
    
    ![Screenshot (136)](https://user-images.githubusercontent.com/91826247/214022888-b11ef485-9614-461d-a3ef-8bb91c30050f.png)

    
    ![Screenshot (137)](https://user-images.githubusercontent.com/91826247/214022910-be3e27a2-a2f7-426f-95f1-abd4002caf88.png)
![Screenshot (138)](https://user-images.githubusercontent.com/91826247/214022955-3157338f-2e1b-405e-afc2-020d0e27269a.png)

![Screenshot (139)](https://user-images.githubusercontent.com/91826247/214022993-970c7bd8-97a8-4e59-a9cc-aae6ff36fe7b.png)

![Screenshot (140)](https://user-images.githubusercontent.com/91826247/214023031-a06ac2ac-a3f6-4960-84a7-6ab9a1d39451.png)



- Create a launch configuration

    Open EC2 dashboard then launch instance

    In AMI select previously created AMI

    Select instance type and security groups

    Select do not assign public ip to instance

    Choose create
    
    ![Screenshot (141)](https://user-images.githubusercontent.com/91826247/214023086-57f6fb08-dcbe-494a-bd15-a2cf9d7b34e4.png)

    ![Screenshot (142)](https://user-images.githubusercontent.com/91826247/214023113-3906063e-ef89-4cbc-86e6-39639891da66.png)
![Screenshot (143)](https://user-images.githubusercontent.com/91826247/214023125-c539132f-81a9-4259-8cfa-c49aaf92f6de.png)



- Create an Auto Scaling group

    Open the EC2 console and choose Auto Scaling Groups

    Choose launch configuration

    Choose VPC and subnets

    Configure group size and scaling policies,  Desired and Maximum capacity

    Choose Create Auto Scaling group
![Screenshot (144)](https://user-images.githubusercontent.com/91826247/214023193-f56cf6c9-bb23-423e-9fe4-fc095c7656c9.png)


![Screenshot (145)](https://user-images.githubusercontent.com/91826247/214023201-7d43ab59-25c0-42e2-a2f0-4c85a170bf8e.png)

![Screenshot (146)](https://user-images.githubusercontent.com/91826247/214023227-1824ec45-4c6f-4cf0-ad08-53d7f6840bb5.png)

  ![Screenshot (147)](https://user-images.githubusercontent.com/91826247/214023237-b4102403-111d-4b25-82d4-d6f7bfc11f5e.png)

![Screenshot (148)](https://user-images.githubusercontent.com/91826247/214023247-92bb63b1-c66e-4db7-a7aa-206e3421e721.png)


    
    To confirm that our ASG is working properly

    Go to ec2 dashboard and terminate any one instance. It will initialize a new instance
    
    
    
    
    
    
- So that's our complete project for deploying wordpress website on AWS
