# h7 My Own Mini Salt Project
This is a repository for Configuration Management Systems - Palvelinten Hallinta ICT4TN022-3018 - Autumn 2022 course's final module.

# Introduction

This Salt project's purpose is to demonstrate some of the things we were taught during this eight week course. I will try to make a salt state that downloads and installs apache2 and also sets up the default web page.

This project will also show you how you can make two new virtual machines on VirtualBox communicate with each other with SaltStack as one of the new machines being a Salt-master computer and the other being a Salt-minion computer.

Most of the course the Salt exercises were done locally with one computer so I decided that as a part of this project I want to set up two completely new computers, one being the master computer and the other being the minion computer.

I created both computers as virtual machines in VirtualBox version 6.1. </br>
For both of my computers I used the same operating system which was Linux Ubuntu 22.04.1 LTS with the desktop image. </br>
The link for downloading said OS: https://releases.ubuntu.com/22.04/ 

I named the computers as the following screen captures show:

__The Salt-master computer:__ </br>
![Screenshot 2022-12-07 175657](https://user-images.githubusercontent.com/116954333/206472816-d818a864-b447-4a3e-991e-b5ba9a0af243.png)

__The Salt-minion computer:__ </br>
![Screenshot 2022-12-07 183029](https://user-images.githubusercontent.com/116954333/206472854-0e088c31-2aa0-4d76-870c-71357ad60f5d.png)

I also want to point out that everything I do in this project can also be done with Vagrant virtual machines and I would personally even recommend it. Just for the sake of this project, I wanted to create everything with using just VirtualBox as most users are more familiar with it rahter than Vagrant. That being said if you got curious about Vagrant I will link below some good articles, where you can find more information about it. </br>

Vagrant's own website: https://www.vagrantup.com/

Karvinen 2017: https://terokarvinen.com/2017/04/11/vagrant-revisited-install-boot-new-virtual-machine-in-31-seconds/ </br>
Nowadays Tero Karvinen recommends using `vagrant init debian/bullseye64` instead of `vagrant init bento/ubuntu-16.04`.
               
Also my own exercises done based on Tero Karvinen's article and teaching (in Finnish): https://github.com/JRissanen/h6-Kulkurin-projekti               


# Part 1: The Beginning Of The Project

The first thing after both computers had installed succesfully was to change the network for both computers from NAT to Briged adapter as to get both computers a different ip-address so that they can communicate with each other.

I did it by opening Oracle VM VirtualBox Manager and right clicking the computer which settings I wished to change and clicking the "Settings..." tab at the very top. After that I went to "Network" tab and then changed the "Adapter 1" "Attached to:" from the default "NAT" to "Bridged Adapter".

![Screenshot 2022-12-08 171523](https://user-images.githubusercontent.com/116954333/206483533-5e5e22ec-1cc6-4044-9b41-fad70f194a18.png)

This was a necessary step because if you have it on "NAT" the ip-address for both of the new virtual machines will be the VirtualBox default "10.0.2.15" and then creating the master-minion architechture with SaltStack wouldn't work.

Now that the intial settings have been defined properly I was able to start the actual project.

I started with installing Salt-master on the Master-PC and the Salt-minion on the Minion-PC. </br>
I also run the command: `sudo apt-get update` as to ensure that all the packeges of all the repositories are up to date as to avoid accidentally installing older not up to date packeges.

__Commands were:__

__Master-PC:__
```
 juliusmaster@Master-PC:~$ sudo apt-get update
 juliusmaster@Master-PC:~$ sudo apt-get -y install salt-master
```

__Minion-PC:__
```
 juliusminion@Minion-PC:~$ sudo apt-get update
 juliusminion@Minion-PC:~$ sudo apt-get -y install salt-minion
```

Next I went on to the Minion-PC to change the settings of the salt-minion configuration file to set the Master PC's ip-address at the top pf the configuration file so that the Minion-PC would know who the Master-PC is. I used micro text editor to do this. </br>
After this I restarted the minion-service and checked its status with commands: </br>
`sudo systemctl restart salt-minion.service` and `sudo systemctl status salt-minion.service`.

Then I went on to the Master-PC and checked all the minion keys with command: `sudo salt-key`. </br>
After that I accepted the unaccpeted key from Minion-PC with the command: `sudo salt-key -A`.

Lastly it was time to check if I had set the Salt master-minion architecture properly. </br>
I ran the commands: `sudo salt '*' cmd.run whoami` and `sudo salt '*' cmd.run whoami` as to check that the Minion-PC answered correctly when I called all minions with the Master-PC.

![Screenshot 2022-12-08 180642](https://user-images.githubusercontent.com/116954333/206498522-b502dd7f-4fcd-4277-a809-0c14f598150e.png)

I was glad to see that everything worked properly. The connection was created succesfully and I was now ready to move on to making salt states on the Master-PC.

__After Thoughts And Information:__

When you set the master ip-address on the minion pc's minion configuration file, you can also set a unique ID applying the same logic. Just make sure you use the same amount of spaces between the option and the value in both parts, for example:
```
 master: <ip-address>
 id:  <unique ID>
```
I used two spaces after the "master:" part.

When I used the command: `cat` to show the changes I made in the minion configuration file, I also gave it some parameters to remove all the comments from the output to make it easier to read. Credit for this neat little trick belongs to Kevin van Zonneveld and here is a link to the site where I found the command: https://kvz.io/cat-a-file-without-the-comments.html

# Part 2: Creating The Salt State And an Index.html-file

To begin this part, I needed to create the required "/srv/salt" directory on the Master-PC, where all the salt states are saved for the Minion-PC (or PCs depending how many you have). Also in that directory a sub directory as to keep the salt states organized. The sub directory's name will also be the name of the salt state you will use to apply the state to your minion(s) later on, so keep that in mind.

All the commands are done on the __Master-PC__:

`juliusmaster@Master-PC:~$ sudo mkdir /srv/salt/MyOwnMiniProject`. </br>
And I moved into the directory: </br>
`juliusmaster@Master-PC:~$ cd /srv/salt/MyOwnMiniProject`.

Then I created an index.html-file which the apache web server is going to use when the salt state is called: </br>
`juliusmaster@Master-PC:/srv/salt/MyOwnMiniProject$ sudoedit ReadyToUse_index.html`. </br>
The index.html-file is just semi empty html-file:
```
<!doctype html>
<html>
  <head>
    <title>ReadyToUse</title>
  </head>
  <body>
    Feel free to cutomise your new ready to use web page!
  </body>
</html>
```

After this I created the init.sls-file which every salt state needs to operate: </br>
`juliusmaster@Master-PC:/srv/salt/MyOwnMiniProject$ sudoedit init.sls`. </br>
The init.sls-file:
```
apache2:
  pkg.installed
/var/www/html/index.html:
  file.managed:
    - source: salt://MyOwnMiniProject/ReadyToUse_index.html
a2enmod userdir:
  cmd.run:
   - creates: /etc/apache2/mods-enabled/userdir.conf
apache2service:
  service.running:
    - name: apache2
    - watch:
      - cmd: 'a2enmod userdir'
```
First row of the "init.sls" file defines the ID of the data I want the salt state to operate, in this case apache2. </br>
Second row tells the function I want, in this case I want apache2 to be installed. </br>

Third row defines the location of the data I want to operate. Since all html-files in Linux are stored in the "/var/www/html/index.html" directory, that is also what I want to use. </br>
Fourth row gives the function to operate a file in that directory. </br>
Fifth row gives the source of the file I want to operate, which in this case is the file I created earlier.

Sixth row targets the apache2 command which allows users to enable user directories. </br>
Seventh row gives the function to run the above command. </br>
Eighth row is a parameter that tells the state to only run the command if a file doesn't exist already. In this case the userdir configuration file. </br>

Ninth row I want the apache2 web server to keep running after the initial installation, however I can't use the basic "apache2" ID because I already defined it in the first row to be installed. Luckily there is an easy trick to get around it. Every service application has two names we can use: the default one and an another one that ends with .service. In this case I already used the default "apache2" so now I used the other one which is apache2.service.  </br>
Tenth row defines that the service keeps running. </br>
Eleventh row defines the name of the service again. </br>
Twelfth row is the salt watcher function which takes action incase there are modifications in whatever it watches. </br>
Thirteenth row defines the action of the watcher function in this case it is the command that enables the user directories in apache.

Screen capture of my Master-PC's terminal:

![Screenshot 2022-12-14 154252](https://user-images.githubusercontent.com/116954333/207610902-c1dcd3d7-03df-447c-9312-dc0986510bfe.png)

For this part I used Tero Karvinen's apache2 with salt example: </br>
https://terokarvinen.com//2018/apache-user-homepages-automatically-salt-package-file-service-example/

And for the meanings and explanations of each row in the init.sls-file I used SaltStack's own article:
https://docs.saltproject.io/en/latest/topics/tutorials/starting_states.html


__After Thoughts And Information:__

The command `sudoedit <file name>` creats a copy of the file you wish to edit and opens it with your preferred texteditor and allows you to make changes to it as the the current user you are logged in with. After exiting and saving Nano, it compares the copy of the original file and the original file and then overwrites the original file. I could have also used the command `sudo nano <file name>` to edit the files but that would require password and would open the files as root. Nothing wrong with that method either I'm just more used to the `sudoedit` command. </br>
Source for this information: https://superuser.com/questions/785187/sudoedit-why-use-it-over-sudo-vi

Every indentation in the init.sls files needs to be exactly __two__ spaces for the state to work properly.

# Part 3: Running The Salt State Locally

Now everything was ready for testing. </br>

All the commands are done on the __Master-PC__:

To run the salt state locally, to see if it works first on the Master-PC before trying to implement it on the Minion-PC, I used the following command: </br>
`juliusmaster@Master-PC:~$ sudo salt-call --local state.apply MyOwnMiniProject`. 

The print it gave was quite long, but in the end the results were successful:

![Screenshot 2022-12-13 175146](https://user-images.githubusercontent.com/116954333/207619125-908187f5-8a04-4045-834f-4102353c8257.png)

The stdout print told me that I needed to restart apache2 to activate the new configuration so I did: </br>
`juliusmaster@Master-PC:~$ sudo systemctl restart apache2`.

Next from the GUI I checked if everything indeed worked as planned. </br>
I opened Mozilla FireFox web browser and typed "localhost" into the search bar, and everything looked the way it was supposed to:

![Screenshot 2022-12-13 180047](https://user-images.githubusercontent.com/116954333/207620857-0c5d5027-ae77-438d-ad90-b4c4b577e019.png)

# Part 4: Applying The Salt State To The Minion-PC

Since the local test was a success it was now time for the final task to test if the salt state could be applied over the Network to a Minion-PC.

First on the __Minion-PC__ I opened Mozilla FireFox web browser and typed "localhost" into the search bar to see that nothing was yet made and I would get "Unable to connect" error, which I did get:

![Screenshot 2022-12-14 162857](https://user-images.githubusercontent.com/116954333/207623282-3410abea-84d0-4547-81ed-f60b545c8047.png)

Now on the __Master-PC__ I ran the following commands: </br>
To test if the Minion-PC was active and would answer: </br>
`juliusmaster@Master-PC:~$ sudo salt '*' cmd.run hostname`. </br>
And to apply the previously created salt state to all the minions (in this case I only had the one: Minion-PC):
`juliusmaster@Master-PC:~$ sudo salt '*' state.apply MyOwnMiniProject`.

The end results were succesful! </br>
The Salt State was applied correctly without any problems. </br>
On the __Minion-PC__ I just had to do the same steps as on the __Master-PC__:
I had to restart apache2 for the configurations to be activated: </br>
`juliusminion@Minion-PC:~$ sudo systemctl restart apache2`.

Then I just had to write the "localhost" into the search bar again and it worked!

![Screenshot 2022-12-14 164515](https://user-images.githubusercontent.com/116954333/207628190-e71d021c-af21-4744-add4-988582136efa.png)

# Conclusion





















__Sources:__

https://docs.saltproject.io/en/latest/topics/tutorials/starting_states.html

https://terokarvinen.com/2022/palvelinten-hallinta-2022p2/?fromSearch=Configuration%20Management%20Systems

https://terokarvinen.com//2018/apache-user-homepages-automatically-salt-package-file-service-example/

https://kvz.io/cat-a-file-without-the-comments.html

https://superuser.com/questions/785187/sudoedit-why-use-it-over-sudo-vi

https://www.vagrantup.com/

