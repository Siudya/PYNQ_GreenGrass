# AWS IOT LAB
## About this Lab
In this lab, you will go through the process of  how to deploy AWS IoT Greengrass to PYNQ-Z2.
## What is AWS Greengrass
AWS IoT Greengrass is software that extends cloud capabilities to local devices. This enables devices to collect and analyze data closer to the source of information, react autonomously to local events, and communicate securely with each other on local networks. AWS IoT Greengrass developers can use AWS Lambda functions and prebuilt connectors to create serverless applications that are deployed to devices for local execution.
The following diagram shows the basic architecture of AWS IoT Greengrass.
![](image/greengrass.png)
Please refer to [What Is AWS IoT Greengrass?](https://docs.aws.amazon.com/greengrass/latest/developerguide/what-is-gg.html) for more information about AWS IoT Greengrass.

## Preparation
- 3 PYNQ-Z2 boards ( PYNQ v2.4)
- AWS account
- 1 router

Conncet the 3 PYNQ-Z2 boards and host PC to LAN ports of router. Conncet WAN port of router to the Internet.
![](https://github.com/xupsh/PYNQ_GreenGrass/blob/master/image/first.jpg)

## Step by Step
- ### Connect & Preparation
  1. Run the following commands on all boards.
```shell
sudo adduser --system ggc_user
sudo addgroup --system ggc_group
```
  2. Conncet your boards to your pc through usb using serial link. Run the following code in the terminals opened for boards and find your boards' ips.
  ```shell
  hostname -I
  ```  
  3. Close the serial link windows and using ssh to connect to your boards.
- ### Create Group
  ***Tip: In our lab, we will have one board used as core, two boards as device (one board used as publisher and the other used as subscriber).***  
  ***Tip: This part you can refer to [this](https://docs.aws.amazon.com/zh_cn/greengrass/latest/developerguide/gg-config.html)***
  1. Sign in to the AWS Management Console on your computer and open the AWS IoT Core console. If this is your first time opening this console, choose Get started.
  2. Choose Greengrass.  
  ![choose greengrass](https://docs.aws.amazon.com/greengrass/latest/developerguide/images/console-greengrass.png)  
  ***Tip: If you don't see the Greengrass node in the navigation pane, change to an AWS Region that supports AWS IoT Greengrass. For the list of supported regions, see AWS Regions and Endpoints in the AWS General Reference.***
  3. On the Welcome to AWS IoT Greengrass page, choose Create a Group.  
  An AWS IoT Greengrass group contains settings and other information about its components, such as devices, Lambda functions, and connectors. A group defines how its components can interact with each other.
  4. On the Set up your Greengrass group page, choose Use easy creation to create a group and an AWS IoT Greengrass core.  
  Each group requires a core, which is a device that manages local IoT processes. A core needs a certificate and keys that allow it to access AWS IoT and an AWS IoT policy that allows it to perform AWS IoT and AWS IoT Greengrass actions. When you choose the Use easy creation option, these security resources are created for you and the core is provisioned in the AWS IoT registry.  
  ![set up](https://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-005.png)
  5. Enter a name for your group (for example, MyFirstGroup), and then choose Next.  
  ![name](https://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-006.png)
  6. Use the default name for the AWS IoT Greengrass core, and then choose Next.
  ![default name](https://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-007.png)
  7. On the Run a scripted easy Group creation page, choose Create Group and Core.  
  ![create group and core](https://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-008.png)
  8. Download your core's security resources and configuration file. On the confirmation page, under Download and store your Core's security resources, choose Download these resources as a tar.gz.
  ![download](https://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-009.png)
  9. After you download the security resources, choose Finish.  
  The group configuration page is displayed in the console:  
  ![finish](https://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-009.2.png)
- ### Run the Core
  ***Tip: This part you can refer to [this](https://docs.aws.amazon.com/zh_cn/greengrass/latest/developerguide/gg-device-start.html)***
  1. Download the AWS IoT Greengrass Core software installation package. Choose the CPU architecture and distribution (and operating system, if necessary) that best describe your core device. This time we choose ARMv7l for Raspbian package. The download link is [here](https://d1onfpft10uf5o.cloudfront.net/greengrass-core/downloads/1.10.1/greengrass-linux-armv7l-1.10.1.tar.gz)
  2. In the previous step, you downloaded two files to your computer:  
    - *greengrass-OS-architecture-1.9.2.tar.gz* This compressed file contains the AWS IoT Greengrass Core software that runs on the core device.  
    - *hash-setup.tar.gz* This compressed file contains security certificates that enable secure communications between AWS IoT and the config.json file that contains configuration information specific to your AWS IoT Greengrass core and the AWS IoT endpoint.
    Transfer the two compressed files from your computer to the AWS IoT Greengrass core device.  
    The commands for example:  
    ```shell
    cd path-to-downloaded-files
    scp greengrass-linux-armv7l-1.10.1.tar.gz xilinx@IP-address:/home/xilinx
    scp hash-setup.tar.gz xilinx@IP-address:/home/xilinx
    ```
  3. Open a terminal on the AWS IoT Greengrass core device and navigate to the folder that contains the compressed files.  
  Decompress the AWS IoT Greengrass Core software and the security resources.  
    - The first command creates the /greengrass directory in the root folder of the AWS IoT Greengrass core device (through the -C / argument).  
    - The second command copies the certificates into the /greengrass/certs folder and the config.json file into the /greengrass/config folder (through the -C /greengrass argument).
  ```shell
  cd path-to-compressed-files
  sudo tar -xzvf greengrass-linux-armv7l-1.10.1.tar.gz -C /
  sudo tar -xzvf hash-setup.tar.gz -C /greengrass
  ```
  4. Download the appropriate ATS root CA certificate. The following example downloads AmazonRootCA1.pem. The wget -O parameter is the capital letter O.  
  ```shell
  cd /greengrass/certs/
  sudo wget -O root.ca.pem https://www.amazontrust.com/repository/AmazonRootCA1.pem
  ```
  You can run the following command to confirm that the root.ca.pem file is not empty:  
  ```shell
  cat root.ca.pem
  ```  
  5. GreenGrass needs JRE.  
  ```
  sudo apt-get install openjdk-8-jre
  ```  
  6. Start AWS IoT Greengrass on your core device.  
  ```shell
  cd /greengrass/ggc/core/
  sudo ./greengrassd start
  ```
  You should see a Greengrass successfully started message.  
- ### Configure two devices in AWS web.
  ***Tip: This part you can refer to [this](https://docs.aws.amazon.com/greengrass/latest/developerguide/module4.html)***  
  1. Create AWS IoT Devices in an AWS IoT Greengrass Group. In the AWS IoT Core console, choose Greengrass, choose Groups, and then choose your group.  
  2. On the group configuration page, choose Devices, and then choose Add your first Device.  
  ![CD](https://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-066.png)  
  3. Choose Create New Device.  
  ![ND](https://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-067.png)  
  4. Register this device as HelloWorld_Publisher, and then choose Next.  
  ![RD](https://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-068.png)  
  5. For 1-Click, choose Use Defaults. This option generates a device certificate with attached AWS IoT policy and public and private key.  
  ![1C](https://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-069.png)  
  6. Create a folder on your publisher board. Download the certificate and keys for your device into the folder.  
  ![CF](https://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-070.png)  
  Make a note of the common hash component in the file names for the HelloWorld_Publisher device certificate and keys (in this example, bcc5afd26d). You need it later. Choose Finish.  
  7. Decompress the hash-setup.tar.gz file. For example, run the following command:  
  ```shell
  tar -xzf hash-setup.tar.gz
  ```
  8. Choose Add Device and repeat steps to add a new device to the group.  
  Name this device HelloWorld_Subscriber. Download the certificates and keys for the device to your subscriber board. Save and decompress them in the same folder that you created for HelloWorld_Publisher.  
  Again, make a note of the common hash component in the file names for the HelloWorld_Subscriber device.  
  9. You should now have two devices in your AWS IoT Greengrass group:  
  ![tg](https://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-071.png)  
  10. Save the root CA certificate as root-ca-cert.pem in the same folder as the certificates and keys for both devices.  
  ```shell
  cd path-to-folder-containing-device-certificates
  curl -o ./root-ca-cert.pem https://www.amazontrust.com/repository/AmazonRootCA1.pem
  ```
  ***Tip: If you meet some net issues here, don't be afraid, you can use the root-ca-cert.pem you downloaded before or just ask someone to give you a copy of it.***
- ### Configure Subscriptions  
  *In this step, you enable the HelloWorld_Publisher device to send MQTT messages to the HelloWorld_Subscriber device.*
  1. On the group configuration page, choose Subscriptions, and then choose Add Subscription.
  2. Configure the subscription.
    - Under Select a source, choose Devices, and then choose HelloWorld_Publisher.
    - Under Select a target, choose Devices, and then choose HelloWorld_Subscriber.
    - Choose Next.  
    ![sb](https://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-072.png)  
  3. For Topic filter, enter hello/world/pubsub, choose Next, and then choose Finish.  
  ***Tip: Make sure that the AWS IoT Greengrass daemon is running, as described in Deploy Cloud Configurations to a Core Device.***  
- ### Deploy & Run
  1. On the group configuration page, from Actions, choose Deploy.  
  ![dp](https://docs.aws.amazon.com/greengrass/latest/developerguide/images/gg-get-started-040.png)  
  2. Then you have to Install the AWS IoT Device SDK for Python.
  Run the following commands to install the AWS IoT Device SDK for Python:  
  ```shell
  cd ~
  git clone https://github.com/aws/aws-iot-device-sdk-python.git
  cd aws-iot-device-sdk-python
  sudo python setup.py install
  ```
  3. We will use button.py and sensor.py in this lab, you can find them in this respository. Copy the button.py to your publisher board and sensor.py to your subscriber board. Remember to copy the .py file to the folder that contains the HelloWorld_Publisher and HelloWorld_Subscriber device certificates files.  
  4. Edit the button.py&sensor.py, find places "xxxx" and replace them with your own settings.  
  ![example](https://github.com/xupsh/PYNQ_GreenGrass/blob/master/image/Capture.PNG)  
  ***Tip: The first 'xxxx' place is where you put your endpoint adress. The other two places are where you put your own device certificates files.  You can refer to [this](https://docs.aws.amazon.com/greengrass/latest/developerguide/test-comms.html) to find what 'xxxx' should be.***
  5. In publisher board, run:
  ```shell
  sudo python3 button.py
  ```
  6. And in subscriber board, run:
  ```shell
  sudo python3 sensor.py
  ```
  Wait a minute until the two processes is up.  
  7. Now you can see the led will change with the temperature.  
  ![tm](https://github.com/xupsh/PYNQ_GreenGrass/blob/master/image/IMG_20190809_164954.jpg)
- ### Also you can build a subscriber in AWS cloud.  
  1. Search AWS IoT in *Services*. Choose *Test* and then *Subscribe to a topic*, fill the topic with yours(here hello/world/pubsub). 
  2. In *MQTT payload display*, we choose the second *Display payloads as strings*. 
  3. Subscribe to the topic and you will find a new tip on the left, enter it and you can watch the information of the button actions.  
  ![aws](https://github.com/xupsh/PYNQ_GreenGrass/blob/master/image/Capture.PNG)
## Further
Now you have learned how to use aws greengrass, you can replace the "hello world" function with something cool. Maybe you can refer to [this](https://github.com/wutianze/PYNQ_GreenGrass/blob/master/demo-learning.md).
## Reference
[AWS Tutorial](https://docs.aws.amazon.com/zh_cn/greengrass/latest/developerguide/gg-gs.html)
