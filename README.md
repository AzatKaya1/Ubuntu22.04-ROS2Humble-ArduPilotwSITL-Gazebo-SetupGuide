# Ubuntu 22.04-ROS2 Humble-ArduPilot w SITL-Gazebo-Setup Guide
This guide explains the step-by-step process of installing and configuring ROS 2 Humble, ArduPilot, and Gazebo Harmonic on Ubuntu 22.04 to create a drone simulation environment. It is intended for Windows users running Ubuntu 22.04 (WSL).

# 1- Install Ubuntu 22.04

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
# 2- Set Locale
-
Make sure you have a locale that supports UTF-8. If you are in a minimal environment (such as a Docker container), the locale may be something minimal like POSIX. Test with the following settings:

```sh
locale  # Check for UTF-8

sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

locale  # Verify settings
```
# 3- Setup ROS 2 Sources
-
Add the ROS 2 apt repository to your system.

### Enable the Ubuntu Universe Repository

```sh
sudo apt install software-properties-common
sudo add-apt-repository universe
```

### Add the ROS 2 GPG Key

```sh
sudo apt update && sudo apt install curl -y
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
```

### Add the ROS 2 Repository

```sh
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null
```

## 4. Install ROS 2 Packages
-

Update your apt repository caches and install ROS 2 packages.
### Update and Upgrade

```sh
sudo apt update
sudo apt upgrade
```

### Install ROS 2 Desktop (Recommended)

```sh
sudo apt install ros-humble-desktop
```

### Install ROS 2 Base (Bare Bones)

```sh
sudo apt install ros-humble-ros-base
```

### Install Development Tools

```sh
sudo apt install ros-dev-tools
```

# 5- Environment Setup
-
Source the ROS 2 setup script to configure your environment. This ensures that ROS is automatically loaded every time a new terminal is opened.

```sh
# Replace ".bash" with your shell if you're not using bash
# Possible values are: setup.bash, setup.sh, setup.zsh
source /opt/ros/humble/setup.bash

# Add to bash file
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
```

# 6- Test ROS 2 Installation
-
Run the talker-listener example to verify the installation.

### Run C++ Talker

```sh
source /opt/ros/humble/setup.bash
ros2 run demo_nodes_cpp talker
```

### Run Python Listener

```sh
source /opt/ros/humble/setup.bash
ros2 run demo_nodes_py listener
```

You should see the talker publishing messages and the listener receiving them. This verifies both the C++ and Python APIs are working properly.























