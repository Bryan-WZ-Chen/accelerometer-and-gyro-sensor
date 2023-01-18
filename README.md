# accelerometer-and-gyro-sensor
An overview of how to bring up sensor using Qualcomm Sensor Hub

## Information
* SOC platform: Qualcomm QCS6490
* Platform info: IDP
* Communication bus: I2C

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
How does the diagram map to the actual Android BSP(board support package) and nonhlos?


Just as a remark if you don't know what nonhlos is:</br>
<span style="color:red">nonHLOS can be considered the core BSP and other supported software package, which performs the bare minimum functionalities at system boot-up</span> including loading the High Level OS(Linux/ Android/ Windows). Non HLOS is typically provided by the vendors owning the HW platform.



