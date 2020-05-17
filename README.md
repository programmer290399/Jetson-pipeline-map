# Jetson-Pipeline-Map
<hr></hr>

![img](./data/Jetson_video_pipeline.png)
<hr></hr>
<p align="center">Complete pipeline Architecture</p>


## Introduction:

* The above map may look a little intimidating but please bear with me, this doc is a comprehensive guide which will make things crystal clear.

* This pipeline aims to accomplish most if not all objectives of the codenscious.ai's smart DVR project, later named Iceberg.

* This project aims to accomplish the following objectives:
    1. Automate Surveillance of multiple CCTV cameras.
    2. Provide on demand remote video stream to user.
    3. Automatically generate alerts based on user defined rules.
    4. Be a plug and play replacement of a common DVR.
    5. Be close to if not less than in cost of a common DVR.
    6. Be a standalone device.
    7. The software that ships with this project is a SaaS app which runs on a subscription based model.

## Overview:

* Here I will give a high level overview of how this whole pipeline works and later in this doc I will go over each part/module and cover three major things about each part viz.
    1. **Status:** Current status of the Module.
    2. **Setup Procedure:** Steps to get the Module up and running.
    3. **Scope:** The improvements that should/could be made to the Module.

* This pipeline starts with a **Analogue Video Source** (eg. CCTV camera), 
the video stream from the analogue source is then converted to a RTSP stream by a **Analogue to RTSP converter**.

* This RTSP video stream is then fed to the **DeepStream App** running on the Jetson nano, the **DeepStream App** runs inference on the video stream and creates detection metadata and a annotated local RTSP video stream on the Jetson Nano.

* Another application named **Jetson Kinship** also runs on Jetson Nano which can stream the local RTSP output of the **DeepStream App** over the internet to the **RTMP Re-Streaming Server**, the streaming can be switched on or off remotely via the **Server Messenger** app running on the **Analytics Server**.

* The **Jetson Kinship** module uses a kafka pipeline to communicate with the **Analytics Server**. The **DeepStream App** also sends the detection metadata to the **Analytics Server** using a kafka pipeline.

* The **Analytics Server** also runs a kafka broker instance which is used to establish communication channels in between various parts of the pipeline.

* Now, when the user requests the video feed from the User Interface, the **Backend Logic** communicates with **Server Messenger** which sends a prompt to **Jetson Kinship** module to start the streaming. When the user closes the video stream or is inactive then we send the prompt to stop video streaming.

* Also the user can set various rules which are checked by the **Analytics Server**'s **Backend Logic** and then it creates alerts and takes various other actions accordingly.

* The **RTMP Re-Streaming Server** simply runs a Nginx based re-streaming service which takes in a RTMP video stream and provides a HLS video stream which can be played in the web browser directly by any number of users (theoretically).

* This sums up the high level overview of this pipeline, in subsequent sections we'll go through in depth discussion of :
    1. **Analogue to RTSP converter**
    2.  **DeepStream App**
    3.  **Jetson Kinship**
    4. **Server Messenger**
    5. **RTMP Re-Streaming Server**
    6. **Analytics Server**

* The **Backend Logic** and user interface were not designed or developed by me, for any queries regarding any of them, please contact Prashant Sir.

## Modules :

### Analogue to RTSP Converter:
<hr></hr>

#### Status:
* Currently we're using a common DVR to convert the Analogue video stream to a RTSP video stream.

#### Setup Procedure:

- **STEP 1:** Plug in the analogue CCTV camera into the DVR, make sure the DVR is powered on.
- **STEP 2:** Connect the DVR to the network using a ethernet cable, this network should be same as the one to which Jetson Nano would be connected.
- **STEP 3:** Enable video streaming over RTSP in the DVR settings and note the IP assigned to the DVR.
- **STEP 4:** The general structure of the live RTSP link provided by the Hikvision DVR is (This MAY change slightly with the DVR):
    ```
    rtsp://<username>:<password>@<address>:<port>/Streaming/Channels/<id>/
    ```
    For ex :
    ```
    rtsp://admin:admin123@192.168.29.41:554/Streaming/Channels/101
    ```
    Use this URL and plunge in your credentials(as set in STEP 3) to get the RTSP live video stream up and running.

    >**NOTE:** Make sure that ports required for RTSP streaming are not blocked over the network, Please contact Sachin sir for detailed information on network related issues.

#### Scope:

* There is a great scope of improving rather redesigning this module, currently we're using an external DVR which defeats the goal (iv) discussed in Introduction.

* We need to develop a custom solution which can be directly integrated into the device we ship.

* As I was not experienced in embedded systems domain I was not able to develop a working embedded DVR replacement prototype.

* However I tried reverse engineering the DVR and found that the Analogue signal was converted to a digital video stream using a chip named `TP-2328`.

* The above mentioned chip had no data sheets available also sourcing it was next to impossible. Some google-fu revealed that there was another chip which accomplished similar goals named [`ADV7403`](https://www.analog.com/en/products/adv7403.html).

* However the process of designing a carrier board for [`ADV7403`](https://www.analog.com/en/products/adv7403.html) is still pending. However, I am unsure that this a good solution. I also ordered the chip and it can be obtained from codenscious.ai's inventory.

* There were other prebuilt solutions too but they were too costly, you can find them [here](http://www.sensoray.com/products/4023.htm).

* There were some old school basic guides I found, which may be helpful for someone having domain knowledge related to this, you can find them [here](http://www.techmind.org/vd/mk1/vdescrpt.html), [here](http://www.techmind.org/vd/vidmk2.html) and [here](https://www.ti.com/solution/stb-dvr?variantid=34486&subsystemid=21024).

* However, there can be other simpler solutions to this conversion problem like directly reading analogue video signal on the GPIO pins of Jetson Nano or putting the analogue video signal over ethernet cable, etc.


### DeepStream App:
<hr></hr>
