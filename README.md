# Ubuntu 22.04-ROS2 Humble-ArduPilot w SITL-Gazebo-Setup Guide
This guide explains the step-by-step process of installing and configuring ROS 2 Humble, ArduPilot, and Gazebo Harmonic on Ubuntu 22.04 to create a drone simulation environment. It is intended for Windows users running Ubuntu 22.04 (WSL).

## 1. Install Ubuntu 22.04

First open the Windows PowerShell and install **Ubuntu 22.04**. If a different version is currently installed, follow the instructions below.
```sh
wsl --install ubuntu-22.04

# Installing, this may take a few minutes...
# Enter new UNIX username:
# New password:
# Retype new password:
.
.
.
# Welcome to Ubuntu 22.04.5 LTS

# If a different version is currently installed
wsl --unregister Ubuntu-xx.xx
wsl --install ubuntu-22.04

```
## 2. Set Locale

Make sure you have a locale that supports UTF-8. If you are in a minimal environment (such as a Docker container), the locale may be something minimal like POSIX. Test with the following settings:

```sh
locale  # Check for UTF-8

sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

locale  # Verify settings
```
## 3. Setup ROS 2 Sources

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

Update your apt repository caches and install ROS 2 packages.
### Update and Upgrade
Updates and upgrades the package lists.
```sh
sudo apt update
sudo apt upgrade
```

### Install ROS 2 Desktop (Recommended)
Installs the ROS 2 Humble desktop version, including:
- RViz (3D visualization tool)
- Gazebo (simulation)
- RQT, robot interfaces, and core ROS tools

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

## 5. Environment Setup

Source the ROS 2 setup script to configure your environment. This ensures that ROS is automatically loaded every time a new terminal is opened.

```sh
# Replace ".bash" with your shell if you're not using bash
# Possible values are: setup.bash, setup.sh, setup.zsh
source /opt/ros/humble/setup.bash

# Add to bash file
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
```

## 6. Test ROS 2 Installation

Run the talker and listener example to verify the installation.

### Run C++ Talker

```sh
source /opt/ros/humble/setup.bash
ros2 run demo_nodes_cpp talker
```
Open a new terminal. 

### Run Python Listener

```sh
source /opt/ros/humble/setup.bash
ros2 run demo_nodes_py listener
```

You should see the talker publishing messages and the listener receiving them. This verifies both the C++ and Python APIs are working properly.

## 7. Set Up ArduPilot with ROS 2
Clone the required repositories and set up the workspace.

### Create Workspace and Clone Repositories

```sh
mkdir -p ~/ardu_ws/src
cd ~/ardu_ws
vcs import --recursive --input https://raw.githubusercontent.com/ArduPilot/ardupilot/master/Tools/ros2/ros2.repos src
```

### Update Dependencies

```sh
cd ~/ardu_ws
sudo apt update
sudo rosdep init
rosdep update
source /opt/ros/humble/setup.bash
rosdep install --from-paths src --ignore-src -r -y
```

### Install MicroXRCEDDSGen Build Dependency

```sh
cd
sudo apt install default-jre
cd ~/ardu_ws
git clone --recurse-submodules https://github.com/ardupilot/Micro-XRCE-DDS-Gen.git
cd Micro-XRCE-DDS-Gen
./gradlew assemble
echo "export PATH=\$PATH:$PWD/scripts" >> ~/.bashrc
```

### Test MicroXRCEDDSGen Installation

```sh
source ~/.bashrc
cd
microxrceddsgen -help
```

## 8. Build the Workspace

Build the ROS 2 workspace.

```sh
cd ~/ardu_ws
colcon build --packages-up-to ardupilot_dds_tests
colcon build --packages-up-to ardupilot_dds_tests --event-handlers=console_cohesion+

```
If you encounter this error: The package name passed to `find_package_handle_standard_args` (tinyxml2)
  does not match the name of the calling package (TinyXML2).  This can lead
  to problems in calling code that expects `find_package` result variables
  (e.g., `_FOUND`) to follow a certain pattern...

Follow the instructions below and try the above code again.

```sh
sudo apt install python3-pip
python3 -m pip install --user pexpect
python3 -m pip install --user future
colcon build --packages-up-to ardupilot_dds_tests  # Repeat until no failures
source ./install/setup.bash
cd
```
## 9. Launch ArduPilot SITL

Launch the ArduPilot SITL simulation.

```sh
source /opt/ros/humble/setup.bash # You don't have to source everytime if you have added to bash file 
cd ~/ardu_ws
colcon build --packages-up-to ardupilot_sitl
source install/setup.bash
ros2 launch ardupilot_sitl sitl_dds_udp.launch.py transport:=udp4
```
If you encounter these errors: 
- [ERROR] [micro_ros_agent-1]:...
- [ERROR] [copter.parm -2]:...

  Run ```sudo pip3 install MAVProxy``` and try the previous code again.
  
## 10. Interact with ArduPilot via ROS 2 CLI

Use the ROS 2 CLI to interact with ArduPilot. Open a new terminal and test the instructions below.

```sh
source ~/ardu_ws/install/setup.bash
ros2 node list
ros2 node info /ap
ros2 topic echo /ap/geopose/filtered
```

If ROS 2 topics aren’t being published, ensure the ArduPilot parameter `DDS_ENABLE` is set to 1 and reboot the launch.

```sh
export PATH=$PATH:~/ardu_ws/src/ardupilot/Tools/autotest
sim_vehicle.py -w -v ArduPlane --console -DG --enable-dds
param set DDS_ENABLE 1
```

Ensure the ArduPilot parameter `DDS_DOMAIN_ID` matches your environment variable `ROS_DOMAIN_ID`. The default is 0 for ArduPilot.

## 11. Install Gazebo

Install Gazebo Harmonic (recommended) or Gazebo Garden. We will install Harmonic.

### Clone Required Repositories

```sh
cd ~/ardu_ws
vcs import --input https://raw.githubusercontent.com/ArduPilot/ardupilot_gz/main/ros2_gz.repos --recursive src
```

### Set Gazebo Version

Set the Gazebo version in your `~/.bashrc` file.

```sh
export GZ_VERSION=harmonic
```

### Update ROS Dependencies

```sh
cd ~/ardu_ws
source /opt/ros/humble/setup.bash
sudo apt update
rosdep update
rosdep install --from-paths src --ignore-src -r
```

## 12. Build and Run Tests

Build the workspace and run tests.

```sh
cd ~/ardu_ws
colcon build --packages-up-to ardupilot_gz_bringup
```

If the build fails, install missing dependencies:

```sh

```




















