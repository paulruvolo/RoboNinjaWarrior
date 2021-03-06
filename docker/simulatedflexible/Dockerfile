# This Docker file is used to encapsulate the Neato setup used in various
# Olin College courses
FROM osrf/ros:melodic-desktop-full-bionic
MAINTAINER Paul Ruvolo Paul.Ruvolo@olin.edu

ENV NEATO_WORLD=empty_no_spawn
ENV DEBIAN_FRONTEND=noninteractive

# Install Node.js 10.x repository
RUN curl -sL https://deb.nodesource.com/setup_10.x | bash -

# install ROS and other packages
RUN apt-get update && apt-get install -y \
    software-properties-common \
    ros-melodic-pointcloud-to-laserscan \
    nodejs \
    libjansson-dev \
    imagemagick \
    wget \
    unzip \ 
    x11vnc \
    xvfb \
    fvwm \
    tcpdump \
    net-tools \
    iputils-ping \
    python-pip \
    libxtst-dev \
    libssl-dev \
    libjpeg-dev \
    vim && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# Setup catkin workspace and ROS environment.
RUN /bin/bash -c "source /opt/ros/melodic/setup.bash && \
                  mkdir -p ~/catkin_ws/src && \
                  cd ~/catkin_ws/src && \
		  git clone -b qea https://github.com/comprobo18/comprobo18.git && \
                  catkin_init_workspace"

RUN /bin/bash -c "source /opt/ros/melodic/setup.bash && \
                  cd ~/catkin_ws/ && \
		  catkin_make && \
                  echo 'source ~/catkin_ws/devel/setup.bash' >> ~/.bashrc"

RUN /bin/bash -c "mkdir -p ~/.gazebo/models; cp -r ~/catkin_ws/src/comprobo18/neato_gazebo/model/* ~/.gazebo/models"

# get the rest of the Gazebo models

COPY dl_models.sh /tmp
#RUN /bin/bash -c "/tmp/dl_models.sh"

ENV GAZEBO_MODEL_PATH=/root/.gazebo/models

RUN /bin/bash -c "mkdir ~/.gazebo/models/mobile_base && \
		  source ~/catkin_ws/devel/setup.bash && \
		  cp ~/catkin_ws/src/comprobo18/neato_robot/neato_description/sdf/neato.sdf ~/.gazebo/models/mobile_base/model.sdf && \
		  cp ~/catkin_ws/src/comprobo18/neato_robot/neato_description/model.config ~/.gazebo/models/mobile_base && \
		  cp -r ~/catkin_ws/src/comprobo18/neato_robot/neato_description/meshes ~/.gazebo/models/mobile_base"

RUN /bin/bash -c "cd ~ && \
		  hg clone https://bitbucket.org/osrf/gzweb && \
		  cd gzweb && \
		  hg up gzweb_1.4.0 && \
		  source /usr/share/gazebo/setup.sh && \
		  xvfb-run -s \"-screen 0 1280x1024x24\" npm run deploy --- -m local -t"

ENV GAZEBO_PORT=11345
ENV GZWEB_PORT=8080
ENV ROS_HOSTNAME=deepthought.olin.edu
ENV ROS_PORT=11311

CMD /bin/bash -c "export ROS_MASTER_URI=http://$ROS_HOSTNAME:$ROS_PORT && export GAZEBO_MASTER_URI=http://localhost:$GAZEBO_PORT && ((sleep 10; cd ~/gzweb && npm start -p $GZWEB_PORT) & (source ~/catkin_ws/devel/setup.bash && roslaunch neato_gazebo neato_no_spawn.launch gui:=false load_camera:=false neato_world:=$NEATO_WORLD))"
