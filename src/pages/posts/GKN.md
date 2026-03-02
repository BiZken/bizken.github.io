---
layout: ../../layouts/Post.astro
title: "Summer Student Computer Vision @ GKN Aerospace"
description: "Computervison using segmentation to detect and blur parts in production due to export control."
date: "Summer 2025"
type: blog
tags: ["Machine Learning", "YOLO", "computer vision", "python", "RTSP", "Flask"]
backLink: /blog
backLabel: Back to posts
---

during the summer 2025 I worked at GKN Aerospace GTC in Trollhättan Sweden.
After discussions with the people there the plan for my project ended up
being a combination of three other employees ideas.
- Experiment with segmentation in computer vision
- Create an interface for a new Rotoclear camera
- Make it possible to monitor the production process from outside the shop floor

## Problem
A quick Google search will show the many domains GKN is involved in: space, commercial aviation, and defense.
Because of this, they are very strict about what can and cannot be shown. The current issue is that there is no way to view machine operations from outside the shop floor due to IP-protected parts.

However, if there were a way to hide or obscure the actual part being manufactured, it could make it possible to install screens in the offices to monitor production progress.

This is where my project came in: to create a proof-of-concept (PoC) application that takes an RTSP stream and blurs the part in real time.

(_Fun fact: 90% of all the commercial planes in the sky have GKN parts on it_)


### Things Used
- Label Studio
- Yolo
- Flask
- MinIO Bucket

# Steps

### Computer Vision
First of all, just to get the segmentation to work, I needed data to train the computer vision model. So I had to bother some other employees for help because, for some reason, I'm not "qualified" to operate the machine, it is what it is I guess. So I got the video footage of the part moving around in the machine, it's a five-axis machine.

Then Came the thing every computer vision person loves, labeling. I took every frame and labeled it using labelstudio with _polygon segmentation_ and these were later moved to MinIO bucket for storage.

All the images in the MinIO bucket were fetched and used for training the model _yolo11n-seg_. having the model I ran it locally on the file itself with all 60FPS and it worked great however the plan wasn't to have it run locally instead it would come from an RTSP stream from the camera. Too achieve this I spun up a docker container running mediamtx and ffmpeg to stream and get it. Getting the stream was no problem however the time it took for the model to run inference and also apply the blur effect was too slow since it processes each frame as an image. To handle this I skipped each frame the application took in and process I also lowered the size of each frame.

<img src="/GKN/Resultat1.png" alt="Res1" style="max-width: 500px; width: 100%;">
<img src="/GKN/Resultat2.png" alt="Res2" style="max-width: 500px; width: 100%;">
<img src="/GKN/Resultat3.png" alt="Res3" style="max-width: 500px; width: 100%;">


This lowered the FPS however it did run smoothly without any interruptions or glitches.
Functioning video stream and computer vision model check now what? Time for interface.

### Interface Overview

According to the documentation of the machine you can control the cameras operations like _brightness_, _zoom_ _contrast_ etc with simple API calls. For the Interface I setup a few things it needed to have.
 - login for admins and "normal users"
 - if no detection of part, blur everything
 - Clear video stream with option for full screen
 - permissions (only admin can change settings when Blur is active)
 - have sessions so if admin unblurs it will still be blurred for normal users
 - API calls/controls
 - Getting information from the camera itself (rpm, temp, etc)

#### Users
First thing first I created a simple sqldatabase for users, one admin account and one normal user. The reason for the different users are pretty simple:
- having it play somewhere else and not onsite
- when settings change on the camera it changes what the AI model sees meaning it will have a hard time detecting the part, so only admins can change it.

Also a simple login page was created for this:
<img src="/GKN/GKNlogin.png" alt="GKN login" style="max-width: 500px; width: 100%;">


#### Interface
I used Flask to create the the whole thing just because I have worked with it before and its easy to start from absolute scratch. I Started off with just a simple split of the page one half for the stream and one for the controls and info. I picked a dark mode just because I worked also on this late at night and I didn't want to get flashbanged like in counter-strike when I opened it up. All was done in simple html, css and javascript, the javascripts was more for the functionalities and tracking of API info because I had to simulate the communication with the camera since at this time it was segmented from talking to the "office network". 


Backend for the AI model also had to be tweaked, incase the model had a confidence score less than 80% it blurred everything out from the stream to protect unauthorized people from seeing the part being blurred incase the model decides to go haywire. I also future plans for an application like this would be connected to a calender system that automatically enables blurs for IP protected parts. When Blur is active the admin is the only one able to control the settings, he will also be able to turn it on and off. In the backend it checks for status on the part if IP protected or not every _X_ seconds.

<img src="/GKN/calandersystem.png" alt="calandersystem" style="max-width: 500px; width: 100%;">

The Controls that the camera supported:
- Change between cameras
- take screenshots
- toggle light on camera
- zoom in and out
- enable auto exposure
- enable auto white balance
- Increase/decrease brightness
- rotate camera
- auto noise reduction
- increase/decrease sharpness
- increase/decrease Saturation
- increase/decrease contrast


The current settings I had displayed together with a _info_ tab from the camera itself with info like which camera it was, temperature, rpm on the lens. I also added controls for the zoom on the actual video, necessary? nope I just wanted to try it.

An option for fullscreen was also added incase the stream itself would just be on a screen somewhere in the office.

For normal users when it is an IP protected part they will not be able to change settings and the part will be blurred:
***Normal User***
<img src="/GKN/interface.png" alt="interface" style="max-width: 500px; width: 100%;">


The admin user on the other hand will be able change settings as well as disable and enable the blur. The blurring effect is user based so its only the admin who sees the unblurred part, normal users still have it blurred. The settings on the other hand is global because its the camera itself so its what the camera send with RTSP that's being effected, before the backend can do something about it
***ADMIN user***

<img src="/GKN/InterfaceAdmin.png" alt="Admininterface" style="max-width: 500px; width: 100%;">

# Second Part
There is however a problem with this model its only for the part I have trained it on that it can detect, since there is parts that differ in size and shape there is also a need to have a way to train models on those parts. The part I used was able to be placed in the machine because it was already made, however parts that are made for the first time ever you wouldn't have that option. To counter this I decided to create another application on the time i had left that lets you create a data set with your specific part.

<img src="/GKN/parts.png" alt="parts" style="max-width: 500px; width: 100%;">


## Idea
The idea I had was pretty simple, create an inteface where you upload a 3D model and then it creates images where said part is "placed" in the machine from different angles. Also have the option to have it really look like its in the machine so with the part holder as well.

## Creation
Every single frame I got from the video of the machine  determined the mounting location in pixels and mapped all of these with a json file. X and Y coordinates, scale and what rotation the part needed to be had to be taken for each frame.
***Example params for images 1 and 50:***

```JSON
{
  "1": {
    "X": 440,
    "Y": 260,
    "Scale": 1.2,
    "RotateRange": 5.4
  },
 "50": {
   "X": 370,
   "Y": 380,
   "Scale": 1.2,
   "RotateRange": 125
  }
``` 


#### 3D model
3D models often comes in .stl files so I had to create a "stl reader" that took screenshots of the part from different angles and then I had to remove the background before the placement of the part on the base image.

##### problem
As with all things issues decides to come in to the picture one is the metallic texture. Just training on the model it didn't work because it didn't have the metallic texture/finish so what I had to do was to take images of the frames I had in different angles of the part I had access too. these textures had to then be placed randomly on the screenshots of the 3D model as a mask to follow the shape of it.

***Example Textures:***
<div style="display: flex; gap: 10px; align-items: center;">
  <img src="/GKN/T1.png" alt="parts" style="max-width: 70px; width: 20%;">
  <img src="/GKN/T2.png" alt="parts" style="max-width: 70px; width: 20%;">
  <img src="/GKN/T3.png" alt="parts" style="max-width: 70px; width: 20%;">
</div>


***Example Masked Texures on model:***
<div style="display: flex; gap: 10px; align-items: center;">
  <img src="/GKN/M1.png" alt="parts" style="max-width: 100px; width: 30%;">
  <img src="/GKN/M2.png" alt="parts" style="max-width: 100px; width: 30%;">
  <img src="/GKN/M3.png" alt="parts" style="max-width: 100px; width: 30%;">
</div>


#### Holder
Now when I had the models with texture/finish on them I needed to have a way to make it look like the part is actually placed inside the machine before starting to label the images so I had to save the bottom part of the machine only and keep the rest as transparent to be able to place the model ontop of background image, then place the model and then the holder ontop.

***Example Images of holder:***

<img src="/GKN/H1.png" alt="parts" style="max-width: 500px; width: 100%;">
<img src="/GKN/H2.png" alt="parts" style="max-width: 500px; width: 100%;">

#### Final Assembly.
So after every step had been done the model with one texture were placed and images were generated with it, making it look like its actually being held. This generated 350 images in total but with only one texture and I had 10 of them so I had to come up with a way to use all textures but without needing to label all 3500 of them. So I created an _Extender_.

### Extender

When you had all the 350 images labeled you uploaded your zip file you got from label studio with the images and their correspodining labels. Since each texture swap didn't change the shape of the part, the original labels still applied. So the extender took each labeled image, swapped in the remaining 9 textures, and duplicated the label for each — turning 350 labeled images into 3,500 and it did this with all the images and all the labels. So you labeled 350 images and the output is 3500 labeled images. The reason for the different textures is because on the machine there is a window that allows light to reflect on the material. 



***Example Image:***
<img src="/GKN/RH1.png" alt="RH" style="max-width: 500px; width: 100%;">


## Interface
This also needed an interface and I built it as a PoC for future implementation so with more machines, sizes and different alloys.

First I created a SQL database with info about the different settings you would be able to set:
```SQL
CREATE TABLE IF NOT EXISTS parts (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    part_id TEXT NOT NULL,
    size TEXT NOT NULL,
    add_hold bool NOT NULL
)
```

The interface I made was pretty simple due to the amount of time I had left that summer.
For the different forms I used Flask_WTF and the first page for the "base creator" different info got saved for the model:
- part_id
- size (base material)
- add_hold (True or False) - if you wanted to add the holder

These things were saved both for the extender (second step) but also for the idea for a future implementation. Also because of the auto size I added preconfigured labels for bounding boxes so no labeling would be needed unless you wanted to have segmentation.

<img src="/GKN/BaseForm.png" alt="baseForm" style="max-width: 500px; width: 100%;">

Since info about the parts got saved the extender had the information so all you needed to do was to pick it as well as upload the labeled dataset and it would then create the extended version with the other textures. You also got to pick the split you wanted for the model training.

<img src="/GKN/extenderform.png" alt="ExenderForm" style="max-width: 500px; width: 100%;">


The full flow of the creator and the extender:

***Creator***
<img src="/GKN/BaseCreator.png" alt="BaseCreator" style="max-width: 500px; width: 100%;">


***Extender***
<img src="/GKN/extender.png" alt="Exender" style="max-width: 500px; width: 100%;">


## Future implementation
I had an idea that you could implement this together with the blurring application so everything happened automaticlly.

<img src="/GKN/Future.png" alt="Future" style="max-width: 500px; width: 100%;">
