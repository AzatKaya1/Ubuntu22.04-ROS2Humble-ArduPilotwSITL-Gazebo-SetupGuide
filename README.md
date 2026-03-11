# Ubuntu 22.04-ROS2 Humble-ArduPilot w SITL-Gazebo-Setup Guide
This guide explains the step-by-step process of installing and configuring ROS 2 Humble, ArduPilot, and Gazebo Harmonic on Ubuntu 22.04 to create a drone simulation environment. It is intended for Windows users running Ubuntu 22.04 (WSL).

1- Install Ubuntu 22.04
------------------------------------------------------------------------------------------------------
First open the Windows PowerShell and install **Ubuntu 22.04**. If a different version is currently installed, follow the instructions below.
```sh
wsl --install ubuntu-22.04

#Installing, this may take a few minutes...
#Enter new UNIX username:
#New password:
#Retype new password:

#Welcome to Ubuntu 22.04.5 LTS

#If a different version is currently installed
wsl --unregister Ubuntu-xx.xx
wsl --install ubuntu-22.04

```
2- Set Locale
------------------------------------------------------------------------------------------------------
```sh
locale  # Check for UTF-8

sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

locale  # Verify settings
```
3- Setup ROS 2 Sources
------------------------------------------------------------------------------------------------------
Add the ROS 2 apt repository to your system.

### Enable the Ubuntu Universe Repository

```sh
sudo apt install software-properties-common
sudo add-apt-repository universe
```

























