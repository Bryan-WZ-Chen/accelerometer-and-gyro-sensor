# accelerometer-and-gyro-sensor
An overview of how to bring up sensor using Qualcomm Sensor Hub

## Information
* SOC platform: Qualcomm QCS6490
* Platform info: IDP
* Communication bus: I2C

## Outline
* Introduction
* Architecture of sensor
* Implementation
* Result


## Introduction
Due to confidential policy of Qualcomm, this tutorial won't involve in too many details. Instead, we just demonstrate how to bring up accelerometer-and-gyro-sensor(A+G) sensor on Qualcomm's platform.

## Architecture of sensor
Following picture is the structure of Qualcomm's sensor hub.

![structure](./architecture.jpg)

As you can see in the figure, there are mainly three components:
* app
* sensor execution environment(SEE) including Snapdragon Sensor Core(SCC)
* physical sensor

This diagram drastically simplies the actual structure but the idea is SEE framework is responsible for provides services to sensors, manages registry, handles client interface etc.

As a result, we'll mainly work on SEE.</br>
Then here comes a question:</br>
How does the diagram map to Android OS architecture?


Before that we need to talk about what non-hlos is:</br>
>non-hlos can be considered the core BSP and other supported software package, which performs the bare minimum functionalities at system boot-up including loading the High Level OS(Linux/ Android/ Windows). Non-hlos is typically provided by the vendors owning the hardware(HW) platform.

>So it will be something like the following diagram I draw:
 Non-hlos includes firmware and device drivers thus it is located between the Hardware Abstraction Layer (HAL) and the Linux kernel. Sometimes you see people say non-HlOS software is located below the Linux kernel layer and above the hardware and sometimes you see statement like non-HLOS software communicates with the Linux kernel through the Hardware Abstraction Layer (HAL). They both are correct because "non-hlOS includes firmware and device drivers". It's easily confusing since the Android OS architecture diagrams on the Internet don't include non-hlos and you can barely find any useful explanation about non-hlos. Possibly it's because non-hlos is typically provided by the vendors owning the HW platform which is proprietary to them. 

![structure2](./android-stack_2x.png)

As stated above, the interconnections of non-hlos and others are extremely complicated. So back to the problem, the actual mapping between sensor execution environment(SEE) and Android OS architecture is basically on nonhlos layer. It makes sense since we have physical sensors and we want it to be integrated into Android OS thus non-hlos takes the role.

Thanks to the fact hardware(HW) platform vendors seem nonhlos as their proprietaries, nonhlos often contains well-structured model according to the one they propose (in the case is SEE).

So the actual file you need to change varies according to the chipset and platform you use and I will use Qualcomm QCS6490 as exmaple.
It is very likely that you can find files I change at your platform with (since the file name may not change):
* Linux find command
* Linux grep command

## Implementation
<b>To bring up sensors, we need to add device driver and register that sensor on the system so that upper layer application can get sensor data or information. </b> </br>

We split it into two parts: device(sensor) driver and registry. Let's look at device driver first.</br>

### Device(sensor) driver
First ask for the device(sensor) driver from the sensor vendor's FAE engineer. Copy the entire driver folder to path:
```
nonhlos/adsp_proc/ssc_drivers/${your_device_driver}
```
> As a side note, Qualcomm has the so called "preferred vendor list" which is some sensors that can use directly without the need of bring up. If you don't have the same path as mine above, you can find where Qualcomm puts other preferred vendor list sensor's driver. After finding the location, paste your device driver there.

Then we need to add the newly added device(sensor) driver to the config file so that it will get compiled. To do so, add driver to path
```
adsp_proc\ssc\chipset\[platform]\por.py
```
And do the following changes:
```
#POR sensors list
   include_sensor_vendor_libs.extend(['lsm6dst',
                                      'sns_ak0991x',
                                      'sns_tmd2725',
                                      'sns_shtw2',
                                      'sns_bu52053nvx'])
   include_sensor_vendor_libs.extend(['YourDriver'])  //<<<Add custom driver here>>>

   # 1. Island drivers
   env.AddUsesFlags(['SNS_ISLAND_INCLUDE_LSM6DST'])
   env.AddUsesFlags(['SNS_ISLAND_INCLUDE_AK0991X'])
   env.AddUsesFlags(['SNS_ISLAND_INCLUDE_LPS22HX'])
   env.AddUsesFlags(['SNS_ISLAND_INCLUDE_DPS368'])
   env.AddUsesFlags(['SNS_ISLAND_INCLUDE_TMD2725'])
   <<<Add custom driver build flag here to include in island if required>>>
```
### Registry
Then we need to registry our sensor, sometimes you may see people say "push registry files", it's basically the same thing.</br>
Push registry to path: </br>
```
/vendor/etc/sensors/config/
```

## Result
