# On-Screen Display (OSD)

_On-Screen Display (OSD)_ overlays flight information such as battery status, altitude, speed, and warnings directly onto the live video feed from your vehicle’s camera. This allows pilots to monitor critical telemetry data in real time without looking away from the video stream.

There are several types of OSD hardware and systems available. Some OSDs are fully integrated into flight controllers, while others are standalone modules that connect between the camera and video transmitter. PX4 supports a number of popular OSD solutions, enabling you to display telemetry data on your FPV feed using compatible hardware.

The video below shows PX4 holding position using the [Ark Flow](..dronecanark_flow.md) sensor for velocity estimation in [Position Mode](..flight_modes_mcposition.md)

lite-youtube videoid=aPQKgUof3Pc title=ARK Flow with PX4 Optical Flow Position Hold

!-- ARK Flow with PX4 Optical Flow Position Hold 20210605 --

The image below shows an optical flow setup with a separate flow sensor ([PX4Flow](..sensorpx4flow.md)) and distance sensor ([Lidar-Lite](..sensorlidar_lite.md))

![Optical flow lidar attached](....assetshardwaresensorsoptical_flowflow_lidar_attached.jpg)

## Setup

An Optical Flow setup requires a downward facing camera and a downward facing [distance sensor](..sensorrangefinders.md) (preferably a LiDAR).
These can be combined in a single product, such as the [ARK Flow](..dronecanark_flow.md), [ARK Flow MR](..dronecanark_flow_mr.md) and [Holybro H-Flow](httpsholybro.comproductsh-flow), or they may be separate sensors.

The sensor(s) can be connected via MAVLink, I2C or any other bus that supports the peripheral.

 info
If connected to PX4 via MAVLink the Optical Flow camera sensor must publish the [OPTICAL_FLOW_RAD](httpsmavlink.ioenmessagescommon.html#OPTICAL_FLOW_RAD) message, and the distance sensor must publish the [DISTANCE_SENSOR](httpsmavlink.ioenmessagescommon.html#DISTANCE_SENSOR) message.
The information is written to the corresponding uORB topics [DistanceSensor](..msg_docsDistanceSensor.md) and [ObstacleDistance](..msg_docsObstacleDistance.md).


The output of the flow when moving in different directions must be as follows

 Vehicle movement  Integrated flow 
 ----------------  --------------- 
 Forwards          + Y             
 Backwards         - Y             
 Right             - X             
 Left              + X             

Sensor data from the optical flow device is fused with other velocity data sources.
The approach used for fusing sensor data and any offsets from the center of the vehicle must be configured in the [estimator](#estimators).

### Scale Factor

For pure rotations the `OPTICAL_FLOW_RAD.integrated_xgyro` and `OPTICAL_FLOW_RAD.integrated_x` (respectively `integrated_ygyro` and `integrated_y`) have to be the same.
If this is not the case, the optical flow scale factor can be adjusted using [SENS_FLOW_SCALE](..advanced_configparameter_reference.md#SENS_FLOW_SCALE).

tip
The low resolution of common optical flow sensors can cause slow oscillations when hovering at a high altitude above ground ( 20m).
Reducing the optical flow scale factor can improve the situation.


## Flow SensorsCameras

### ARK Flow & ARK Flow MR

[ARK Flow](..dronecanark_flow.md) is a [DroneCAN](..dronecanindex.md) optical flow sensor, [distance sensor](..sensorrangefinders.md), and IMU.
It has a PAW3902 optical flow sensor, Broadcom AFBR-S50LV85D 30 meter distance sensor, and Invensense ICM-42688-P 6-Axis IMU.

[ARK Flow MR](..dronecanark_flow_mr.md) is a [DroneCAN](..dronecanindex.md) optical flow sensor, [distance sensor](..sensorrangefinders.md), and IMU, for mid-range applications.
It has a PixArt PAA3905 optical flow sensor, Broadcom AFBR-S50LX85D  50 meter distance sensor, and Invensense IIM-42653 6-Axis IMU.

### Holybro H-Flow

The [Holybro H-Flow](httpsholybro.comproductsh-flow) is a compact [DroneCAN](..dronecanindex.md) optical flow and [distance sensor](..sensorrangefinders.md) module.
It combines a PixArt PAA3905 optical flow sensor, a Broadcom AFBR-S50LV85D distance sensor, and an InvenSense ICM-42688-P 6-axis IMU.
An all-in-one design that simplifies installation, with an onboard infrared LED enhances visibility in low-light conditions.

### PMW3901-Based Sensors

[PMW3901](..sensorpmw3901.md) is an optical flow tracking sensor similar to what you would find in a computer mouse, but adapted to work between 80 mm and infinity.
It is used in a number of products, including some from Bitcraze, Tindie, Hex, Thone and Alientek.

### Other CamerasSensors

It is also possible to use a boardquad that has an integrated camera.
For this the [Optical Flow repo](httpsgithub.comPX4OpticalFlow) can be used (see also [snap_cam](httpsgithub.comPX4snap_cam)).

## Range Finders

You can use any supported [distance sensor](..sensorrangefinders.md).
However we recommend using LIDAR rather than sonar sensors, because of their robustness and accuracy.

## Estimators

Estimators fuse data from the optical flow sensor and other sources.
The settings for how fusing is done, and relative offsets to vehicle center must be specified for the estimator used.

The offsets are calculated relative to the vehicle orientation and center as shown below

![Optical Flow offsets](....assetshardwaresensorsoptical_flowpx4flow_offset.png)

Optical Flow based navigation is enabled by both [EKF2](#ekf2) and LPE (deprecated).

### Extended Kalman Filter (EKF2) {#ekf2}

For optical flow fusion using EKF2, set [EKF2_OF_CTRL](..advanced_configparameter_reference.md#EKF2_OF_CTRL).

If your optical flow sensor is offset from the vehicle centre, you can set this using the following parameters.

 Parameter                                                                                           Description                                                             
 --------------------------------------------------------------------------------------------------  ----------------------------------------------------------------------- 
 a id=EKF2_OF_POS_Xa[EKF2_OF_POS_X](..advanced_configparameter_reference.md#EKF2_OF_POS_X)  X position of optical flow focal point in body frame (default is 0.0m). 
 a id=EKF2_OF_POS_Ya[EKF2_OF_POS_Y](..advanced_configparameter_reference.md#EKF2_OF_POS_Y)  Y position of optical flow focal point in body frame (default is 0.0m). 
 a id=EKF2_OF_POS_Za[EKF2_OF_POS_Z](..advanced_configparameter_reference.md#EKF2_OF_POS_Z)  Z position of optical flow focal point in body frame (default is 0.0m). 

See [Using PX4's Navigation Filter (EKF2)  Optical flow](..advanced_configtuning_the_ecl_ekf.md#optical-flow) for more information.
