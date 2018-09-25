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

가제보 상에서 만든 로봇 모델을 통해서 여러 가지 시뮬레이션 동작을 살펴보기 위해 훨씬 접근하기 쉬운 플러그 인을 생산합니다.

제일 먼저 빈 공간의 가제보를 실행시키는 것이 아니라 가제보가 실행될 때 로봇 모델도 같이 실행될 수 있게 환경을 구성하기 위하여

'base.world' 파일을 생성합니다.

### 1) base.world

``` xml
<model name="my_basemodel">
  <include>
    <uri>model://korus_base</uri>
  </include>
```
가제보 모델 디렉토리에 존재하는 korus_base 모델을 가져와서 base.world 실행파일에서는 이 모델을 "my_basemodel"이라고 정의합니다.
모델을 포함할 때는 'include'를 사용합니다. 코드를 확인하면 기본 설정으로 되어 있는 Sun, ground_plane도 같이 있는 것을 확인할 수 있습니다.

### 2) control_plugin.cc

가제보 상에서 로봇 모델을 제어할 수 있는 플러그 인 클래스를 생성합니다.

Load 함수는 플러그 인이 가제보 시뮬레이션에서 동작할 때 가제보에서 호출되는 함수입니다.

이 함수 안에서 가제보 상에서 원하는 다양한 동작을 설정합니다. 함수에 필요한 인자는 시뮬레이션 안에서 불러올 모델과 sdf 요소들입니다.

``` cpp
this->model = _model;

this->joint0 = _model->GetJoints()[0];
this->joint1 = _model->GetJoints()[1];
```
모델에 대한 조인트 정보들을 가져옵니다. 모델은 포인터로 저장합니다. 

base model은 조인트가 left/right wheel 두 개이기 때문에 두 조인트 값만 가져옵니다.

``` cpp
this->pid = common::PID(0.1, 0, 0);

this->model->GetJointController()->SetVelocityPID(
  this->joint0->GetScopedName(), this->pid);
this->model->GetJointController()->SetVelocityPID(
  this->joint1->GetScopedName(), this->pid);
```
로봇 모델에 대해 PID 제어도 가능합니다. 코드에서는 P 제어만 사용하였습니다.

``` cpp
this->rosNode.reset(new ros::NodeHandle("gazebo_client"));

ros::SubscribeOptions so =
  ros::SubscribeOptions::create<std_msgs::Float32>(
    "/" + this->model->GetName() + "/vel_cmd",
    1,
    boost::bind(&ControlPlugin::OnRosMsg, this, _1),
    ros::VoidPtr(), &this->rosQueue);
this->rosSub = this->rosNode->subscribe(so);

this->rosQueueThread =
  std::thread(std::bind(&ControlPlugin::QueueThread, this));
```
ROS와 연동시키기 위해 ROS와 관련된 node를 생성합니다. 코드에서는 구독자 노드를 생성합니다.

topic은 커멘드창에서 실행시킬 'cmd_vel'과 관련된 topic을 구독하고 topic 안에 있는 메시지를 처리하는 동작을 추가합니다.

``` cpp
public: void OnRosMsg(const std_msgs::Float32ConstPtr &_msg)
  {
    this->SetVelocity(_msg->data);
  }
```

자료형이 float32인 메시지를 처리하는 함수입니다. 구독한 topic에서 발생하는 메시지를 받을 때 메시지를 'SetVelocity'함수에 데이터를 보냅니다.

``` cpp
public: void SetVelocity(const double &_vel)
  {
    this->model->GetJointController()->SetVelocityTarget(
      this->joint0->GetScopedName(), _vel);
    this->model->GetJointController()->SetVelocityTarget(
      this->joint1->GetScopedName(), _vel);
  }
```

'SetVelocity'함수에서는 각 조인트에 인자로 받은 데이터의 값을 목표 값으로 설정하여 적용합니다.


이 함수를 통하여 ROS에서 속도 값을 지정하면 지정된 속도 값을 목표로 조인트가 회전할 것입니다.

``` cpp
private: void QueueThread()
{
  static const double timeout = 0.01;
  while (this->rosNode->ok())
  {
    this->rosQueue.callAvailable(ros::WallDuration(timeout));
  }
}
```

QueueThread 함수는 메시지를 처리하기 위한 ROS helper 함수입니다.

``` cpp
private: physics::ModelPtr model;

private: physics::JointPtr joint0, joint1;

private: common::PID pid;

private: transport::NodePtr node;

private: transport::SubscriberPtr sub;

private: std::unique_ptr<ros::NodeHandle> rosNode;

private: ros::Subscriber rosSub;

private: ros::CallbackQueue rosQueue;

private: std::thread rosQueueThread;
```
위 변수들은 control_plugin.cpp 에서 사용되는 Load 함수에서 사용되는 멤버 변수를 initialize한 부분입니다.
