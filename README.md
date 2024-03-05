# serverless_laravel_application
Deploying a Serverless Laravel Application in AWS using Bref

Serverless deployment is a new service that AWS is offering to help face the challenge of scaling applications, expenses etc. Developers who want to decrease their go-to-market time and build lightweight, flexible applications that can be expanded or updated quickly, benefit greatly from serverless computing. It offers greater scalability, more flexibility, and quicker time to release, all at a reduced cost.

Advantages:

· No server management is necessary
· Only charged for the server space they use
· Scalable
· Quick deployments and updates are possible
. Low latency

Disadvantages:

· Testing are debugging become difficult
· Security concerns
· Serverless architectures are not built for long running processes
· Performance may be affected

Environments used:

· Linux machine
· AWS IAM user
· PHP
· Composer

You can place the directory containing the application file in whichever location.

1. Create an AWS account. Then, create an IAM user with programmatic access and get the access key ID and secret access key.

2. Ensure that the php extensions curl and xml are enabled. You can check using the following command.

   php -m | grep xml
   php -m | grep curl

   If not enables, install using the command,

   sudo apt-get install php-xml
   sudo apt-get install php-curl

Enable them in the file ‘/etc/php/8.1/cli/php.ini’ and restart apache using ‘service apache2 restart’

3. Now, we have to install and configure the serverless framework as a global dependency.

   npm install -g serverless

Now, we have to add the AWS IAM user credentials using the following commad.

   serverless config credentials --provider aws –key <key>  --secret <secret>

4. Install the latest version of Laravel and also create new project in it.

   composer global require laravel/installer
   Get the path and add it to the PATH variable.

   composer global config bin-dir –absolute

   /root/.config/composer/vendor/bin (This is the path I obtained). Add it using the below command


  echo 'export PATH="$PATH:/root/.config/composer/vendor/bin"' >> ~/.bashrc
Now, check the laravel version using,

  laravel --version
  Laravel Installer 4.5.0

5. To create new app,

  laravel new serverless-app
To have Bref configured with Laravel we also need to install the laravel-bridge component.

  composer require bref/bref bref/laravel-bridge --with-all-dependencies

6. Now, we have to create a serverless.yaml file.

  php artisan vendor:publish --tag=serverless-config

Following is its contents.

service: laravel                                                                                                                                                                                           
 
provider:
    name: aws 
    # The AWS region in which to deploy (us-east-1 is the default)
    region: us-east-1
    # The stage of the application, e.g. dev, production, staging… ('dev' is the default)
    stage: dev 
    runtime: provided.al2
 
package:
    # Directories to exclude from deployment
    exclude:
        - node_modules/**
        - public/storage
        - resources/assets/**
        - storage/**
        - tests/**
 
functions:
     This function runs the Laravel website/API
    web:
        handler: public/index.php
        timeout: 28  in seconds (API Gateway has a timeout of 29 seconds)
        layers:
            - ${bref:layer.php-81}
        events:
            -   httpApi: '*' 
     This function lets us run artisan commands in Lambda
    artisan:
        handler: artisan
        timeout: 120  in seconds
        layers:
            - ${bref:layer.php-81}  PHP
            - ${bref:layer.console}  The "console" layer
 
plugins:
    # We need to include the Bref plugin
    - ./vendor/bref/bref

7. Before deploying, we need to clear any configuration change generated on our machine.

   php artisan config:cache
   php artisan config:clear

8. 1. In case if your application requires database, you can deploy it in a server or in AWS RDS.

9. 1. Next we need to change the location of compiled view and a few other things like changing session driver to cookie and log channel to standard error in environment (.env) file.

Open .env file and add following:

CACHE_DRIVER =array
VIEW_COMPILED_PATH =/tmp/storage/framework/views 
SESSION_DRIVER =array 
LOG_CHANNEL =stderr
10. Next we need to make changes in app service provider so that if compiled view directory is not present then it should recreate it automatically.

So copy and paste below section in boot method of app/Providers/AppServiceProvider.php

  public function boot()
  {
  // Make sure the directory for compiled views exist
  if (! is_dir(config('view.compiled'))) {
  mkdir(config('view.compiled'), 0755, true);
  }
  }


11. After that, we’re ready to deploy.

serverless deploy
