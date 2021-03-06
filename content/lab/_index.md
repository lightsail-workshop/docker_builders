---
title: "Lab Guide"
weight: 60
---

### Prerequisites
* You will need to have an AWS account with sufficient priviliges to create resources in Amazon Lightsail to complete this lab.

* You should be familiar with using a command line.

* The ability to edit files with a Linux text editor is helpful, but not mandatory

### Deploy your infrastructure
{{% notice tip %}}
Please create all your resources in the *Oregon / us-west-2* region
{{% /notice %}}

#### Deploy your Lightsail instance
In this step you'll deploy a Lightsail instance running Ubuntu 18.04

* Log into your AWS account

* Navigate to the <a href="https://lightsail.aws.amazon.com/ls/webapp/home" target="_blank"> Lightsail home page</a>
* Close any notifcations that pop up or slide out
* Click *Create instance* 
* Scroll down and under *Select a blueprint* click *OS only* and select *Ubuntu 18.04*

    ![](./images/ubuntu.png?classes=border)

{{% notice tip %}}
You can accept the default value for the instance size
{{% /notice %}}

* Scroll down and under *Identify your instance* enter *Docker* as the name

    ![](./images/instance_name.png?classes=border)

* Click *Create instance* 

#### Deploy the Lightsail database
In this section we'll deploy a Lightsail database. Lightsail databases are a managed database service that allow you to get away from the complexity of deploying and managing database software. Lightsail manages the underlying infrastructure and database engine, you only need to worry about creating and deploying the actual databases and tables running inside the service. 

* From the <a href="https://lightsail.aws.amazon.com/ls/webapp/home/" target="_blank">Lightsail console home page</a> select *Databases* from the horizontal menu

    ![](../../images/databases-menu.jpg?classes=border)

* Click *Create database*

    ![](../../images/create-database.jpg?classes=border)

{{% notice tip %}}
Make sure you're deploying your database into the same region as your other resources.
{{% /notice %}}


* From the *MySQL* dropdown choose version *5.7.26*. 

    ![](../../images/database_version.jpg?classes=border)

* By default Lightsail will create a strong password for you. However this password can contain characters that making copying and pasting difficult, so for this lab we're going to specify a password. To do this first click *Specify login credentials* 

    ![](../../images/specify_credentials.jpg?classes=border)

* Next, uncheck the box next to *Create a strong password for me* and enter *taskstasks* for the password. 

    ![](../../images/password.jpg?classes=border)

* Name your database *todo-db*

    ![](../../images/name-db.jpg?classes=border)

* Click *Create database* 

{{% notice tip %}}
It will take several minutes for the creation process to complete, so feel free to move on to the next step while this happens. You'll come back to the Lightsail database later in this session.
{{% /notice %}}   

### Install Docker
The next step is to install Docker onto your Lightsail instance. You will do this via the Linux command line. To acces the command line, you'll use the built in web-based SSH client. 

* Return to the Lightsail console home page

* Click on the console icon on the card for your instance. This will open up a web-based terminal session for that instance

    ![](../../images/console_icon.png?classes=border)

* The easiest way to install Docker is to simply curl down the installation script from the Docker website. To do that copy and paste the command below into your terminal window

        curl -sSL https://get.docker.com | sh

    You should see a lot of output in the terminal window with the final text being a warning about adding a user to the Docker group granting the ability to run containers that can obtain root priviliges. Since this is just a lab environment, it's safe to ignore this warning. 

* Issue the following command to add the *ubuntu* user to the *docker* group

        sudo usermod -aG docker ubuntu

* You need to close down the terminal window and log back in for this last step to take effect. Enter *exit* into the terminal then click the *Reconnect* button

* To ensure Docker is properly installed issue the following command. This command will pull down a container image from Docker Hub and run a "hello world" program.

        docker run hello-world

    The message should indicate that everything is working properly

        Hello from Docker!
        This message shows that your installation appears to be working correctly.

### Start the web container
In the last step we ran a simple Docker container based on a prepackaged <a href="https://docs.docker.com/engine/reference/commandline/images/" target="_blank">Docker image</a>. Docker images are compiled files that can be instantiated into running programs. There are thousands of prepackaged images on <a href="https://hub.docker.com/search?q=&type=image" target="_blank">Docker hub</a> or you can build your own from scratch. 

For this session we'll be using a prepackaged Docker image as covering the process of building Docker images is out of scope.

* Issue the `docker run` command to start the web front end

        docker run -d -p 80:80 --name web mikegcoleman/todo-php:reinvent
        
    * *-d*  starts the container in the background (detached mode)
    * *-p* insructs Docker to route any connections coming into the Lightsail instance on port 80 to be routed to the docker container on the same port
    * *--name* provides a friendly name for the container
    
    If the container started successfully you should see the container ID displayed
    
        a1e4d7d02f0f9f9017fa22b007a5deaaf95c71395458c0f9dbd8ee02b309e4c
        
{{% notice tip %}}
Your container ID will be different the one displayed above is for illustration purposes only
{{% /notice %}}

### Access the running website
You should now be able to access your running website. To do so navigate to the IP address of your Lightsail instance. You can find the IP address of your Lightsail instance in the botton right hand corner of your instance card. 

![](../../images/ip_address.png?classes=border)

If the application is running you'll see a website like the one below. 

![](../../images/website.png?classes=border)


{{% notice tip %}}
The application is running, but it's not configured correctly as it cannot find the database. You'll fix this error in the next section
{{% /notice %}}

{{% notice tip %}}
Notice that the Front-end host displayed at the bottom of the screen matches the first part of your container ID.  
{{% /notice %}}

### Attach the Lightsail database
In order to the connect to the Lightsail database you'll need to have database endpoint, user name, and password. 

* Return to the database section of the Lightsail console, and click on the name of your database

* Under *Connection details* click on *Show* to display the password

    ![](../../images/connection_details.png?classes=border)

* Navigate back to the website and click on settings

    ![](../../images/settings.png?classes=border)

* Fill in the details on the setting page with the values found under the *Connections details* section for your database

* Click *Add Task* from the website menu and enter a task

* Click *Add Task* to save the task to the database

The containerized web application is now saving the task data into the Lightsail database. 

{{% notice tip %}}
Notice that the Database host displayed at the bottom of the screen matches the endpoint value for your Lightsail database.   
{{% /notice %}}

### Dealing with data persistence

One of the issues with containers is that they are ephemeral, meaning that any data saved in the container is lost if the container is restarted. In the case of the Todo application it saves the information for the database connection in a file. 

* Stop the running container 

        docker stop web

* Remove the running container

        docker rm web

* Restart the container

        docker run -d -p 80:80 --name web mikegcoleman/todo-php:reinvent
    
* Go back to your web browser, and reload the Todo application. Notice that it once again cannot find the database connection. 

There are a couple of ways of dealing with this issue. You could refactor the application to use environment variables, which can be passed when you start the container. You could also use a secrets manager to securely store the connection information. 

The other way to handle the issue is to instruct Docker to remap the file from the container file system to the host's file system. With this approach, any information written to the file will persist even if the container is removed. 

* Stop the running container 

        docker stop web

* Remove the running container

        docker rm web

* Create a local copy of the configuration file by cloning down the git repo and moving the *connectvalues.php* file into the home directory (issue each command below one by one)

        cd ~

        git clone -b loft https://github.com/mikegcoleman/todo-php

        mv /home/ubuntu/todo-php/connectvalues.php .

* Ensure the permissions on the file are correct (they need to be set to represent the container's user account)

        sudo chown www-data:www-data /home/ubuntu/connectvalues.php 

* Restart the container using the `-v` switch and the instruct Docker to remap the container's configuration file to the one on the local host. 

        docker run -d -p 80:80 --name web -v ~/connectvalues.php:/var/www/html/connectvalues.php mikegcoleman/todo-php:reinvent

* Follow the steps in the  *Attach the Lightsail database* section to specify the database connection information

* Go back to the web browser and reload the site, notice that the data you entered previously is now being retrieved from the database

* Now go ahead and stop the container, remove it, and restart it 

        docker stop web

        docker rm web

        docker run -d -p 80:80 --name web -v ~/connectvalues.php:/var/www/html/connectvalues.php mikegcoleman/todo-php:reinvent

* Reload the Todo website, and notice that this time after deleting and starting a new container the configuration information was persisted. 

* If you look at the contents of the file on your host machine you can see it contains the connection values for your Lightsail database

        cat /home/ubuntu/connectvalues.php














