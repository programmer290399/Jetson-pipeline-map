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
    1. Current status of the Module.
    2. Steps to get the Module up and running.
    3. The improvements that should/could be made to the Module.

* This pipeline starts with a `Analogue Video Source` (eg. CCTV camera), 
the video stream from the analogue source is then converted to a RTSP stream by a `Analogue to RTSP converter`.

* This RTSP video stream is then fed to the `DeepStream App` running on the Jetson nano, the `DeepStream App` runs inference on the video stream and creates detection metadata and a annotated local RTSP video stream on the Jetson Nano.

* Another application named `Jetson Kinship` also runs on Jetson Nano which can stream the local RTSP output of the `DeepStream App` over the internet to the `RTMP Re-Streaming Server`, the streaming can be switched on or off remotely via the `Server Messenger` app running on the `Analytics Server`.

* The `Jetson Kinship` module uses a kafka pipeline to communicate with the `Analytics Server`. The `DeepStream App` also sends the detection metadata to the `Analytics Server` using a kafka pipeline.

* The `Analytics Server` also runs a kafka broker instance which is used to establish communication channels in between various parts of the pipeline.

* Now, when the user requests the video feed from the User Interface, the `Backend Logic` communicates with `Server Messenger` which sends a prompt to `Jetson Kinship` module to start the streaming. When the user closes the video stream or is inactive then we send the prompt to stop video streaming.

* Also the user can set various rules which are checked by the `Analytics Server`'s `Backend Logic` and then it creates alerts and takes various other actions accordingly.

* The `RTMP Re-Streaming Server` simply runs a Nginx based re-streaming service which takes in a RTMP video stream and provides a HLS video stream which can be played in the web browser directly by any number of users (theoretically).

* This sums up the high level overview of this pipeline, in subsequent sections we'll go through in dept discussion of :
    1. `Analogue to RTSP converter`
    2.  `DeepStream App`
    3.  `Jetson Kinship`
    4. `Server Messenger`
    5. `RTMP Re-Streaming Server`
    6. `Analytics Server`

* The `Backend Logic` and user interface were not designed or developed by me, for any queries regarding any of them, please contact Prashant Sir.

