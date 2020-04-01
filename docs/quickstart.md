## Installation Script
!!! tip
    The usage of this installation script is highly recommended for fresh installs. If already have
    dependencies installed, is indicated to update then manually before running the script.

We provide an automated shell script that first install all dependencies and then configure Hyperion.
All you need to do is to run it:
````
./install_infra.sh
````

The script will ask you for some information:
````
Enter rabbitmq user [hyperion]:
````

Enter the desired rabbitmq user and hit enter. If you leave it in blank, the default user
`hyperion` will be set.

Then, it will ask your for the rabbitmq password:
```
Enter rabbitmq password [123456]:
```
Again, input the password and hit enter.

Now, the script will do the work, this can take a while. Get a cup of coffee and relax. =)

!!! info
    The install script may ask you for the admin password. This is needed to install the dependencies, please, provide it.
   
 