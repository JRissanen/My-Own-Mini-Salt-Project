# h7 My Own Mini Salt Project
This is a repository for Configuration Management Systems - Palvelinten Hallinta ICT4TN022-3018 - Autumn 2022 course's final module.

# Introduction

This Salt project's purpose is to demonstrate some of the things we were taught during this eight week course. I will try to make a salt state that downloads and installs some useful applications for ICT students, such as Zoom video confrencing and Visual Studio Code.

This project will also show you how you can make two new virtual machines on VirtualBox communicate with each other with SaltStack as one of the new machines being a Salt-master computer and the other being a Salt-minion computer.

Most of the course the Salt exercises were done locally with one computer so I decided that as a part of this project I want to set up two completely new computers, one being the master computer and the other being the minion computer.

I created both computers as virtual machines in VirtualBox version 6.1. </br>
For both of my computers I used the same operating system which was Linux Ubuntu 22.04.1 LTS with the desktop image. The link for downloading said OS: https://releases.ubuntu.com/22.04/ 

I named the computers as the following screen captures show:

__The Salt-master computer:__ </br>
![Screenshot 2022-12-07 175657](https://user-images.githubusercontent.com/116954333/206472816-d818a864-b447-4a3e-991e-b5ba9a0af243.png)

__The Salt-minion computer:__ </br>
![Screenshot 2022-12-07 183029](https://user-images.githubusercontent.com/116954333/206472854-0e088c31-2aa0-4d76-870c-71357ad60f5d.png)

# The beginning of the project

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

__After thoughts and information:__

When you set the master ip-address on the minion pc's minion configuration file, you can also set a unique ID applying the same logic. Just make sure you use the same amount of spaces between the option and the value in both parts, for example:
```
 master: <ip-address>
 id:  <unique ID>
```
I used two spaces after the "master:" part.

When I used the command: `cat` to show the changes I made in the minion configuration file, I also gave it some parameters to remove all the comments from the output to make it easier to read. Credit for this neat little trick belongs to Kevin van Zonneveld and here is a link to the site where I found the command. </br>
Link to Kev's site: https://kvz.io/cat-a-file-without-the-comments.html






























































