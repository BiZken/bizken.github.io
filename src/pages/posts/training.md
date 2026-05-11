---
layout: ../../layouts/Post.astro
title: "Training Time"
description: "Find the Specific room of an event based on a picture"
type: writeup
platform: bellingcat
tags: ["OSINT"]
backLink: /writeups
backLabel: Back to writeups
---

## Challenge Overview

In December 2017, Christiaan Triebert hosted a workshop as a representative of Bellingcat, find the name of the room he is sitting in when the image was taken.

<img src="/TrainingTime/Init.jpeg" alt="Res1" style="max-width: 500px; width: 100%;">



## Step 1 – Visual Analysis
First I did an inital analysis of the image and focused on areas that could help identify the location
<img src="/TrainingTime/initview.png" alt="Res1" style="max-width: 500px; width: 100%;">


1. Walls - The walls have a specific design that with green rectangles on a "cream" base
2. Carpet - the carpet is pretty much fully covered so I assumed that its not a nordic country, also because its December and they are not wearing anything big/warm, I assumed some musim country just because of the style.
3. Door - the door has a special style, big with lots of small details/carvings
4. water bottle - one thing I wanted to see was if I could identify the waterbottle to see if I could focus on certain areas, some brands are tied to specific regions


Initally just from looking I thought it would be some muslim/middle eastern country. They style of the door, carpet aswell as the walls are not something Im used to or have seen in real life, looks like something from a movie.


---

## Step 2 – Reverse Image Search

### Bottle Identification
A reverse image search on the bottle returned a match for a **Nestle** waterbottle, not the results I was hopping for as its a global brand.
I identified with based on the logo(1) aswell as the red part(2)

<img src="/TrainingTime/bottle.png" alt="Res1" style="max-width: 500px; width: 100%;">
<img src="/TrainingTime/BottleFound.png" alt="Res1" style="max-width: 500px; width: 100%;">


### walls and floor/carpet
Running a reverse search on the walls aswell as the carpet didn't give any result that could help me identify the romm/place. I tried each individually and together but nothing:

<img src="/TrainingTime/carpet.png" alt="Res1" style="max-width: 500px; width: 100%;">
<img src="/TrainingTime/wall.png" alt="Res1" style="max-width: 500px; width: 100%;">
<img src="/TrainingTime/wall+floor.png" alt="Res1" style="max-width: 500px; width: 100%;">

---

## Step 3 Moving On
After the images didn't give me anything I started looking at the information given in the challange:
`In December 2017, Christiaan Triebert hosted a workshop as a representative of Bellingcat. The image above shows Christiaan at that event.`

So what I decided to focus on:
 - December 2017
 - Christiaan Triebert
 - hosted a workshop
 - representative of Bellingcat

 With these snippets of info I started to look over search engines for old bellingcat events and found on their website event upcoming and past. The past section however only went back to 2021 so I check with wayback machine but again noting from 2017.
 Than I started to see workshops in general with Christiaan and found some but most were from after 2017, around 2019. I found however one link: 
 `https://gijn.org/stories/2017s-award-winning-journalism-from-around-the-globe/` with arabic text so I knew I was on the right track based on my previous hunch.
 Looking at more links I found an x link about something called `ARIJ17` with Christiaans name on it.

<img src="/TrainingTime/clue1.png" alt="Res1" style="max-width: 500px; width: 100%;">




## Checking event
I decided to take a close look at this _ARIJ17_ event and found this link 
`https://arij17.arij.net/%d8%a7%d9%84%d9%85%d8%aa%d8%ad%d8%af%d8%ab%d9%88%d9%86-%d8%a7%d9%84%d9%85%d8%aa%d8%af%d8%b1%d8%a8%d9%88%d9%86/index9ed2.html?lang=en`
And searching for `chri` I can easily see that he is part of the event, tied to _Bellingcat_

<img src="/TrainingTime/event.png" alt="Res1" style="max-width: 500px; width: 100%;">

However going to this profile on the events website you can see where he will have his workshop (_workshop D_) but this room is not the correct rom in the image. Meaning that either the room changed or the picture is just taken somewhere else.

<img src="/TrainingTime/workshop.png" alt="Res1" style="max-width: 500px; width: 100%;">

I started to look at images of the event to see if I could see something that looked like the inital picture and eventually I came across one that had similar walls, no exactly the same I think but close enough for me to react on it, si i figured that it might be another room at the event.

<img src="/TrainingTime/Room_.png" alt="Res1" style="max-width: 500px; width: 100%;">



# Final
On the event webpage I found the location of the actual event, it was being held at `Mövenpick resort & Spa Dead Sea`.
Searching on the hotel together with "events" you get a webpage that showcases different rooms the resort have to offer. At the bottom of the page you have the `The Grand Ball Room` Which got the same walls also the same style of carpet. Entering this into the answer and it comes as correct, so the image wasn't from his actual workshop but instead maybe at a break or a gathering.


<img src="/TrainingTime/rättplats.png" alt="Res1" style="max-width: 500px; width: 100%;">
<img src="/TrainingTime/finalcompare.png" alt="Res1" style="max-width: 500px; width: 100%;">




**answer:** The Grand Ball Room  

---

This challange wasn't super difficult, it didn't require super much details to be able to find the location. It did make it alittle harder because of no reverse image search but when they specified "christiaan" it made it alot easier as you could easily narrow it down.
