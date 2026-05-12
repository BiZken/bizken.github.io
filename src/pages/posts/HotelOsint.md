---
layout: ../../layouts/Post.astro
title: "No Room for Error"
description: "Find the Hotel based on the image taken from the room"
type: writeup
platform: bellingcat
tags: ["OSINT"]
backLink: /writeups
backLabel: Back to writeups
---

## Challenge Overview

An image was taken from a hotel room and it's our task to find which hotel it is. At first glance it looks like a somewhat difficult task since there is not much to go on, but even though it's not the number of MP I would prefer, you can see some things. I also think I managed to figure out not only the hotel but the hotel room.

<img src="/NoRoomForError/Init.jpeg" alt="Res1" style="max-width: 500px; width: 100%;">



## Step 1 – Visual Analysis
First I did an initial analysis of the image and focused on areas that could help identify the location. At first I didn't really see anything but it felt like Scandinavia — being from Sweden it felt like either Stockholm, Malmö or Göteborg.

<img src="/NoRoomForError/initview.png" alt="Res1" style="max-width: 500px; width: 100%;">

To see some of the details you really needed to zoom in. Some of the first things I noticed:
1. Floor/carpet - The carpet is specific, having multi-color hexagons on it
2. Painting - Knowing that the image is from a hotel room, the combination of the painting and the carpet can help me identify the correct hotel since most hotels tend to style their rooms the same.
3. Hotel - This hotel room looks like a classic hotel room from Scandic in Sweden, worth noting
4. Street signs - Looking at the street signs you can see that they drive on the right side of the road
5. Christmas lighting - Now for the one thing that stood out: in Sweden we have these things called "adventsstjärnor" which are lamp covers we put up around Christmas.
With the other info I felt that this was either Sweden, Norway, Denmark or Finland. Since this is really specific it's worth looking into.
6. Buildings/towers - In the distance you can see one tall tower with lights on it and another without, which could help when looking at maps
7. Parking sign - The parking sign looks Nordic, worth noting


---

## Step 2 – Reverse Image Search

### Room Identification
A reverse image search on the different things in the room returned nothing useful, nothing that could help me identify the location.

<img src="/NoRoomForError/Painting.png" alt="Res1" style="max-width: 500px; width: 100%;">
<img src="/NoRoomForError/Floor.png" alt="Res1" style="max-width: 500px; width: 100%;">
<img src="/NoRoomForError/ROOM.png" alt="Res1" style="max-width: 500px; width: 100%;">


### Environment
Running a reverse search on the outside could return results for images from when the building was built. I also decided to reverse search things like the street signs, towers and lamp posts, but nothing came up.
The images are not the highest resolution but it's still worth a shot.

<img src="/NoRoomForError/lyktstolpe.png" alt="Res1" style="max-width: 500px; width: 100%;">

<img src="/NoRoomForError/building.png" alt="Res1" style="max-width: 500px; width: 100%;">

<img src="/NoRoomForError/towers.png" alt="Res1" style="max-width: 500px; width: 100%;">

<img src="/NoRoomForError/SKYLT.png" alt="Res1" style="max-width: 500px; width: 100%;">

<img src="/NoRoomForError/street.png" alt="Res1" style="max-width: 500px; width: 100%;">

---

## Step 3 – Moving On
After the images didn't give me anything I started looking at the signs that this is a Nordic country. The "adventsstjärnor", with a quick Google search for which countries have them:
`In addition to Germany and Sweden, the Advent star and similar Advent lighting are found, mainly through export, in countries such as: Finland, the Netherlands, Switzerland and the USA.`

So what I decided to do with this info is look at big cities in the Nordic countries and see if I can find a match for the lamp post.

<img src="/NoRoomForError/lyktstolpe.png" alt="Res1" style="max-width: 500px; width: 100%;">

I started going through Street View and found different styles but none that had that 90-degree bend.

<img src="/NoRoomForError/olsolykta2.png" alt="Res1" style="max-width: 500px; width: 100%;">
<img src="/NoRoomForError/olsolykta1.png" alt="Res1" style="max-width: 500px; width: 100%;">

When I didn't find anything from the Street View images I went back to the photo to inspect it some more, and eventually I just decided to take an image of the whole street and do a reverse search on that, including the buildings.

<img src="/NoRoomForError/streetfull.png" alt="Res1" style="max-width: 500px; width: 100%;">

With that I found this site `https://www.bosthlm.se/1178115/textilgatan-23` and going down on the website you can see a map. Looking at the map you can see a green plot that looks similar to the triangular shaped plant plot in the initial image.

<img src="/NoRoomForError/initmapfind.png" alt="Res1" style="max-width: 500px; width: 100%;">


### Finding the Hotel

With the matching map I decided to look for hotels close by and found one: `Motel L Hammarby Sjöstad`
This hotel/motel shows the same hotel room on their booking site as in the image:
https://www.booking.com/hotel/se/motel-l.sv.html?chal_t=1778525562473&force_referer=https%3A%2F%2Fwww.google.com%2F

**Answer:** Motel L Hammarby Sjöstad

Submitting the name of the place gives a correct answer. Being done with the challenge I decided to take it one step further — I want to know which hotel room the image was taken from.
This proved to be a real challenge, but you know what we say: fuck around and find out.

<img src="/NoRoomForError/WhereImage.png" alt="Res1" style="max-width: 500px; width: 100%;">

### Finding the Room

The first thing I did was check Street View and compare it with the angle of the photo taken. You can see from the image that the photo was not taken from the first floor, but looking at the white "cube" you can see that you won't be able to see a car behind it.

<img src="/NoRoomForError/catwontsee.png" alt="Res1" style="max-width: 500px; width: 100%;">

So looking at the Street View we can determine that it's not the top floor, since we can clearly see it from the Google car. We can also see the fourth floor, but the Google car camera sits on top of the roof which means it's quite high up.

<img src="/NoRoomForError/NotSeeStreen.png" alt="Res1" style="max-width: 500px; width: 100%;">

Knowing that the hotel room doesn't have full visibility of the street, we can determine that it's either the third or fourth floor.

Now we need something to use as a reference, so I took the building across from the hotel room. I tried to find the height of the building but didn't find anything, so instead I just did it visually.

<img src="/NoRoomForError/RefBuildning.png" alt="Res1" style="max-width: 500px; width: 100%;">

Looking at the initial picture we can see that the photo is eye level with the upper floors of the building across the street. Going into Google Earth we can enable 3D mode and look from an angle behind the hotel.

<img src="/NoRoomForError/WhichFlorr.png" alt="Res1" style="max-width: 500px; width: 100%;">

With a rough determination we can see that it's either the fourth or third floor. Now we need some sort of visual cue to know which room on those floors it is.
Looking at the initial picture we can see a "beam" (1) that is turned slightly outwards, and the "cube" shaped building (2) is right on the edge relative to the hotel room.

Getting an overview of the location we can determine that it's most likely the third room from the edge:
<img src="/NoRoomForError/3rdEdge.png" alt="Res1" style="max-width: 500px; width: 100%;">

<img src="/NoRoomForError/RoomIN.png" alt="Res1" style="max-width: 500px; width: 100%;">

---