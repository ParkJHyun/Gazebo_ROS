# Gazebo_ROS

korus의 base model만 사용하여 Gazebo 상에서 ROS로 구동하는 방법입니다.

[![Alt text for your video](https://user-images.githubusercontent.com/35755034/45861655-d1fd6480-bda8-11e8-8565-d405201e048f.jpg)](https://www.youtube.com/embed/7F-1URlKaKY)

(위 이미지를 클릭하면 재생 영상을 확인할 수 있습니다.)

1. 터미널 실행

``` c
  $ cd ~/korus_plugin/build
  $ source /opt/ros/kinetic/setup.bash
  $ roscore
```
2. 다른 터미널 실행

``` c
  $ cd ~/korus_plugin/build
  $ export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:~/korus_plugin/build
  $ source /opt/ros/kinetic/setup.bash
  $ gazebo ../base.world
```
3. 다른 터미널 실행

``` c
  $ source /opt/ros/kinetic/setup.bash
  $ rostopic pub /my_basemodel/cmd_vel std_msgs/Float32 1.5
```

URDF 파일은 깃허브 내 Tutorial_ROSpy/urdf_base/korus_base.urdf 파일에서 확인할 수 있습니다.

관련 Tutorial들은 Gazebo 사이트에서 확인할 수 있습니다.

**Gazebo Tutorial**: http://gazebosim.org/tutorials

**※먼저 korus_base 디렉토리는 ~/.gazebo/models 내에, korus_plugin 디렉토리는 작업공간 내에 존재해야합니다.**

## 1. korus_base

## 2. korus_plugin
