# What is this repository for?
Train DRL agents on ROS compatible simulations for autonomous navigation in highly dynamic environments. Flatland-DRL integration is based on Ronja Gueldenring's repo: drl_local_planner_ros_stable_baselines. Following features are included:
* Ros-melodic compatible version of drl_local_planner_ros stable_baselines (see [Franklins notes](#franklins-notes-for-old-repository) at the end for specific modification details)
* Setup to train a local planner with reinforcement learning approaches from [stable baselines](https://github.com/hill-a/stable-baselines) integrated ROS
* Training in a simulator fusion of [Flatland](https://github.com/avidbots/flatland) and [pedsim_ros](https://github.com/srl-freiburg/pedsim_ros)
* Local planner has been trained on static and dynamic obstacles: [video](https://www.youtube.com/watch?v=nHvpO0hVnAg)
* Combination with arena2d levels for highly randomized training and better generalization

# Documentation
Overall workflow of arena-flatland. As a 2d simulator, flatland is utilized. For detailed documentation see: . The agent files including network designs and DRL algorithms are reralized in python.

<p align="center">
  <img src='img/arena-flatland-wf.jpg' alt="teaser results" width="70%"/>
  <p align="center"><i>Architecture of arena-flatland</i></p>
</p>


## Installation
0. Standard ROS setup (Code has been tested with ROS-melodic on Ubuntu 18.04) with catkin_ws
```
sudo apt-get update && sudo apt-get install -y \
libqt4-dev \
libopencv-dev \
liblua5.2-dev \
screen \
ros-melodic-tf2-geometry-msgs \
ros-melodic-navigation \
ros-melodic-rviz 
```

1. Clone this repo into your catkin_ws 
````
mkdir -p catkin_ws/src
cd catkin_ws && catkin_make
cd src
git clone https://github.com/ignc-research/arena-flatland
cd arena-flatland
````

2. Copy Install file and install forks
````
mv .rosinstall ../ 
cd ..
rosws update
```` 

3. Install virtual environment and wrapper (as root or admin!) on your local pc (without conda activated) to be able to use python3 with ros
```
sudo pip3 install --upgrade pip
sudo pip3 install virtualenv
sudo pip3 install virtualenvwrapper
which virtualenv   # should output /usr/local/bin/virtualenv  
```

      
4. Create venv folder inside hom directory
```
cd $HOME
mkdir python_env   # create a venv folder in your home directory 
```

Add this into your .bashrc/.zshrc :
```
echo "export WORKON_HOME=/home/linh/python_env   #path to your venv folder
export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3   #path to your python3 
export VIRTUALENVWRAPPER_VIRTUALENV=/usr/local/bin/virtualenv
source /usr/local/bin/virtualenvwrapper.sh
source ~/.zsh" >> ~/.zshrc
```
Create a new venv
```
mkvirtualenv --python=python3.6 arena-flatland-py3
workon arena-flatland-py3
```

Install packages inside your venv:
```
pip3 install pyyaml rospkg catkin_pkg exception numpy=="1.18.5" tensorflow=="1.5.0" gym pyquaternion mpi4py matplotlib netifaces scikit-build
```     
   
5. Install and build additional packages from drl_forks
```
cd drl_local_planner_forks/stable_baselines/ 
pip3 install -e .
cd $HOME/catkin_ws
catkin_make
```

6. Create folders for training results and eval data (you can follow the same structure as below)
```
cd $HOME/catkin_ws
mkdir -p data/{evaluation_data,tensorboard_log_ppo_10}/{train,test,evaluation_sets}    

```
7. Set system-relevant variables 
* Modify all relevant paths in rl_bringup/config/path_config.ini
```
echo "path_to_venv=/home/user/python_env/arena-flatland-py3
path_to_train_data=/home/user/code/catkin_ws/data
path_to_eval_data_train=/home/user/code/catkin_ws/data/evaluation_data/train
path_to_eval_data_test=/home/user/code/catkin_ws/data/evaluation_data/test
path_to_eval_sets=/home/user/code/catkin_ws/data/evaluation_data/evaluation_sets
path_to_catkin_ws=/home/user/code/catkin_ws/
path_to_tensorboard_log=/home/user/code/catkin_ws/data/tensorboard_log_ppo_10
path_to_models=/home/user/code/catkin_ws/data/agents
ros_version=melodic" >> ~/catkin_ws/src/arena-flatland/rl_bringup/config/path_config.ini
```
    
8. Include source to your setup.zsh/bash for ros packages
```
echo "source ~/catkin_ws/devel/setup.zsh" >> ~/catkin_ws/setup.zsh
```

9. Now you are ready to use arena-flatland. Remeber to Activate your venv to run the training.

## Note: if you are using ROS Kinetic 
1. Follow the same steps but install the ros packages for kinteic (instead of ros-melodic-rviz, install ros-kinetc-rviz, etc.)
2. Export Pythonpath to point to your venv because kinetic will look at its internal site packages for the opencv lib.
```
export PYTHONPATH="/home/linh/python_env/arena-flatland-gpu/lib/python3.6/site-packages:$PYTHONPATH"
```



# Example Usage

1. Train agent
    * Open first terminal (roscore): 
    ```
    roscore
    ```
    * Open second terminal (simulationI:
    ```
    roslaunch rl_bringup setup.launch ns:="sim1" rl_params:="rl_params_scan"
    ```
    * Open third terminal (DRL-agent):
     ```
    source <path_to_venv>/bin/activate 
    python $HOME/catkin_ws/src/arena-flatland/rl_agent/scripts/train_scripts/train_ppo.py
    ```
    * Open fourth terminal (Visualization):
     ```
    roslaunch rl_bringup rviz.launch ns:="sim1"
    ```

2. Execute self-trained ppo-agent
    * Copy your trained agent in your "path_to_models"
    * Open first terminal: 
    ```
    roscore
    ```
    * Open second terminal: 
    ```
    roslaunch rl_bringup setup.launch ns:="sim1" rl_params:="rl_params_scan"
    ```
    * Open third terminal:
    ```
    source <path_to_venv>/venv_p3/bin/activate 
    roslaunch rl_agent run_ppo_agent.launch mode:="train"
    ```
    * Open fourth terminal: 
    ```
    roslaunch rl_bringup rviz.launch ns:="sim1"
    ```
    * Set 2D Navigation Goal in rviz

# Examples: Run pretrained Agents
Note: To be able to load the pretrained agents, you need to install numpy version 1.17.0.
```
<path_to_venv>/venv_p3/bin/pip install numpy==1.17
```

### Run agent trained on raw data, discrete action space, stack size 1
1. Copy the example_agents in your "path_to_models"
2. Open first terminal: 
    ```
    roscore
    ```
3. Open second terminal for visualization: 
    ```
    roslaunch rl_bringup rviz.launch ns:="sim1"
    ```
4. Open third terminal: 
    ```
    roslaunch rl_bringup setup.launch ns:="sim1" rl_params:="rl_params_scan"
    ```
5. Open fourth terminal:
    ```
    source <path_to_venv>/venv_p3/bin/activate 
    roslaunch rl_agent run_1_raw_disc.launch mode:="train"
    ```
### Run agent trained on raw data, discrete action space, stack size 3
1. Step 1 - 4 are the same like in the first example
2. Open fourth terminal:
    ```
    source <path_to_venv>/venv_p3/bin/activate 
    roslaunch rl_agent run_3_raw_disc.launch mode:="train"
    ```

### Run agent trained on raw data, continuous action space, stack size 1
1. Step 1 - 4 are the same like in the first example
2. Open fourth terminal:
    ```
    source <path_to_venv>/venv_p3/bin/activate 
    roslaunch rl_agent run_1_raw_cont.launch mode:="train"
    ```

### Run agent trained on image data, discrete action space, stack size 1
1. Step 1 - 3 are the same like in the first example
4. Open third terminal: 
    ```
    roslaunch rl_bringup setup.launch ns:="sim1" rl_params:="rl_params_img"
    ```
5. Open fourth terminal:
    ```
    source <path_to_venv>/venv_p3/bin/activate 
    roslaunch rl_agent run_1_img_disc.launch mode:="train"
    ```


    
# Franklin's notes for old repository:

### in folder: flatland
#### replace
CV_LOAD_IMAGE_GRAYSCALE
#### to
cv::IMREAD_GRAYSCALE



### in folder: pedsim
#### replace
shared_ptr
#### to
boost:: shared_ptr

#### replace
const function<void (boost:: shared_ptr
#### to
const boost:: function<void (boost:: shared_ptr

### install baseline
cd <path_to_catkin_ws>/src/drl_local_planner_forks/stable_baselines/
workon arenapy3
pip install -e .

# in folder: ws_drl_my/src/drl_local_planner_forks/flatland/flatland_plugins/include/flatland_plugins

"-std=c++11" is not enough , add #include<bits/stdc++.h>

    
    
