Set up Magento2 with Docker
===========================

# Clone the repository
In order to clone the magento docker repository, create your project folder, navigate to it and execute the 
following command:

```
git clone https://git.exozet.com/php-tgd/magento2.git .
```

_Don't forget to change git origin to point to your own project!_

# Set up docker
## Edit the local enviroment file
Navigate to the docker folder inside your project and edit the PROJECT_TAG variable in the **.env** file. 
The PROJECT_TAG value will be added to your docker container names.

```
PROJECT_TAG="your_tag_here_in_one_word"
```

## Start the docker container
Run the following code:

```
docker-compose up -d
```

This will start downloading the required images and set up the main PHP image (this might take a few minutes)

Once all images are downloaded, you can start with installing magento in your php container

## Installing magento in you docker PHP container
Locate your php container named **magento2_php_{PROJECT_TAG}** 
(where {PROJECT_TAG} is the environment tag you set up in your docker .env file).

Connect to the terminal of this container (add you container name instead of the "PLACEHOLDER" word)

```
docker exec -it PLACEHOLDER sh
```

### Run the following commands inside your PHP container terminal to install your local version of Magento 2:

1. **Download magento2**

   You will be asked for your magento marketplace public and private keys,
   available at https://marketplace.magento.com/customer/accessKeys/
   (create an account and access key if you don't have one, it's free)

    ```
    composer create-project --repository-url=https://repo.magento.com/ magento/project-community-edition .
    ```

    **Explanation:**
    * **--repository-url=https://repo.magento.com/** is your base repository location
    * **magento/project-community-edition** is the latest magento version
    * **.** tells composer to clone repository in the current folder, 
    which should be /var/www/html/ inside the docker PHP container   


2. **Install magento2**

    ```
    php bin/magento setup:install --base-url=http://magento2.docker:8080/ --db-host=mysql --db-name=magento_db --db-user=root --db-password=magento_pass --admin-firstname=admin --admin-lastname=admin --admin-email=admin@admin.com --admin-user=admin --admin-password=changeme123 --language=en_US --currency=USD --timezone=America/Chicago --use-rewrites=1 --backend-frontname=admin --search-engine=elasticsearch7 --elasticsearch-host=elasticsearch --elasticsearch-port=9200
    ```

    **Explanation:**
    * **--base-url=http://magento2.docker:8080/** your local magento url, you have to set it up in your local /etc/conf 
    configuration OR change the url to one that you already have set up
    * **--db-host=mysql** is your mysql hostname inside the docker-compose.yaml file
    * **--db-name=magento_db** is your mysql database set up inside the docker-compose.yaml file
    * **--db-user=root** is the mysql user that is connecting to your database
    * **--db-password=magento_pass** is the mysql root password that is set up inside the docker-compose.yaml file
    * **--admin-firstname=admin** the first name of the admin user that will be created
    * **--admin-lastname=admin**  the last name of the admin user that will be created
    * **--admin-email=admin@admin.com** the email address of the admin user that will be created
    * **--admin-user=admin** the username of the admin user that will be created
    * **--admin-password=** the password of the admin user that will be created
    * **--language=en_US** the default language of the magento installation
    * **--currency=USD** the default currency of the magento installation
    * **--timezone=America/Chicago** the default timezone of the magento installation
    * **--use-rewrites=1** YES, use rewrites, mainly this is used in the real world
    * **--backend-frontname=admin** the admin folder of the magento installation
    * **--search-engine=elasticsearch7** the version of elasticsearch that is set up inside the docker-compose.yaml file
    * **--elasticsearch-host=elasticsearch** the host of the elasticsearch engine that is set up inside 
    the docker-compose.yaml file
    * **--elasticsearch-port=9200** the port of the elasticsearch engine that is set up inside the docker-compose.yaml file


3. **Post install setup**

    ```
    php bin/magento indexer:reindex
    php bin/magento setup:upgrade
    php bin/magento setup:static-content:deploy -f
    php bin/magento module:disable Magento_AdminAdobeImsTwoFactorAuth Magento_TwoFactorAuth 
    php bin/magento cron:install --force
    php bin/magento setup:di:compile
    php bin/magento cache:flush
    ```
    
    **Explanation**
    * **indexer:reindex** will create the first index in your local magento installation
    * **setup:upgrade** discovers and installs all the plugins that are in you magento codebase
    * **setup:static-content:deploy -f** generates and deploys all the static content that is used by magento
    * **module:disable Magento_AdminAdobeImsTwoFactorAuth Magento_TwoFactorAuth** disables two-factor authentication, 
    as this is not needed in development
    * **cron:install --force** install all base cron jobs of you local magento installation
    * **setup:di:compile** compile all plugins
    * **cache:flush** clear all the cache


4. **Create an admin user**

    ```
    php bin/magento admin:user:create  --admin-user admin --admin-password changeme123 --admin-firstname admin --admin-lastname admin --admin-email admin@admin.com
    ```
    
    **Explanation**
    * **--admin-user admin** set user name to be admin
    * **--admin-password** set the password of the admin user
    * **--admin-firstname admin** set up the first name of the admin user
    * **--admin-lastname admin** set up the last name of the admin user
    * **--admin-email admin@admin.com** set up the email of the admin user


5. **Add sample data to you local magento installation (skip for a clean installation)**

    When running the following command, you might be asked again 
    to enter your magento marketplace public and private keys

    ```
    php bin/magento sampledata:deploy
    ```

    After this, install the downloaded sample data in magento:   

    ```
    php bin/magento setup:upgrade
    php bin/magento setup:static-content:deploy -f
    php bin/magento cache:clean
    ```

### Verify if all went well

At this point you should have a local installation of magento. The magento store code should be available in your 
docker container under the folder /var/www/html which is also mapped to `magento2-src` inside your project folder.

Open your browser and access http://magento2.docker:8080 
(or the url that you set up when installing your version of magento). 
This should open the front page of your magento store.

In order to access your admin panel, go to http://magento2.docker:8080/admin and enter the username and password 
for the admin user you created.

To access your database inside PhpMyAdmin, go to http://magento2.docker:1234 and enter your database user credentials 
from your docker-compose.yaml file.

## Plugin development tips for magento
* plugins inside magento need to be placed inside the folder magento2-src/app/code/{your pugin name} 
in your project folder
* each time you add or modify a PHP file inside a magento plugin, you need to run the following commands:
    ```
    php bin/magento setup:upgrade
    php bin/magento setup:di:compile
    php bin/magento cache:flush
    ```
    **Explanation**
    * **setup:upgrade** will discover all plugins
    * **setup:di:compile** compiles all plugin codes to be used with magento
    * **cache:flush** clears all the cache


## Troubleshooting

> **Important**
> 
> On some systems you might need to run the following command, in order to add the necessary rights to run magento:
> 
> ```
> chown -R www-data:www-data /var/www/html
> chmod -R 777 /var/www/html
> ```
> 