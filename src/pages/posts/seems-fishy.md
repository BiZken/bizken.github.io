---
layout: ../../layouts/Post.astro
title: "SeemsFishy"
description: "Find the location and object from a single picture."
type: writeup
platform: bellingcat
tags: ["OSINT"]
backLink: /writeups
backLabel: Back to writeups
---

## Challenge Overview

Starting of we have a singel picture of a lobster statue, first thing I tested was a reverse image search to see if I would get any locations but it would not return any useful info.

<img src="/SeemsFishy/Init.jpg" alt="Res1" style="max-width: 500px; width: 100%;">



## Step 1 – Visual Analysis
Since the rev search didn't provide anything I manually inspected the image for any geographic indicators.

<img src="/SeemsFishy/InitalView.png" alt="Res1" style="max-width: 500px; width: 100%;">


1. Van
2. Boats - so water close by, probably a harbor
3. Street signs - nothing special, looks like the once Im used to
4. Specific types of poles
5. Red Tiles for sidewalk aswell as Yellow markings on road
6. Blue Parking Spot
7. White buildings (close to water)



Initally just from looking I thought it would be some mediterainan country. I have been to a few countires and it felt like it would be greece or croatia. Especially Greece since I thought it was the classic greece style buildings you always see on the ads for the country.

---

## Step 2 – Reverse Image Search

### Vehicle identification
A reverse image search on the vehicle in the frame returned a match for a **Fiat Ducato**.

<img src="/SeemsFishy/Van.png" alt="Res1" style="max-width: 500px; width: 100%;">


> Note: The license plate was mounted in its standard position, confirming the image is **not mirrored**.

### Building architecture
Running a reverse search on the buildings produced the following result:

<img src="/SeemsFishy/Building.png" alt="Res1" style="max-width: 500px; width: 100%;">

> *"According to rev image; This type of architecture is characteristic of cities in the Apulia region of southern Italy."*

This initially pointed toward Italy, but it didn't align consistently with the other clues so it was noted but not treated as definitive.

---

## Step 3 – Narrowing Down the Country

To strat narrowing down the search i first looked into coutries that were left side drive. Since the image wasn't mirrored I knew that the place was right side drive based on the parking aswell as the signs. A quick search for coutries that drive on the left side could help me exclude:
    Botswana
    Eswatini
    Kenya
    Lesotho
    Malawi
    Mauritius
    Mozambique
    Nambia
    Cyprus
    Guernsey
    Jersey
    Malta
    Isle of Man
    Ireland
    United Kingdom
    
### Poles
After that I started to look at the different elements in the image

Cross-referencing the distinctive poles in the image against [geohints.com/meta/bollards](https://geohints.com/meta/bollards) produced a strong match for **Greece**, and the red/pink road colouring was also in the images with this.

### License plates
Checking [geohints.com/meta/licensePlates](https://geohints.com/meta/licensePlates) for Greece showed a reasonable match though the format is shared with several other countries, so this alone wasn't conclusive.


So at this point I was sure it was Greece so I started looking at maps on greece and congrats the amount of coast line I had to go through.

<img src="/SeemsFishy/Grekland.png" alt="Res1" style="max-width: 500px; width: 100%;">

What I did was that I also check for searches like:
- _Lobster statue greece_
- Also searched for `Greece` along with the image on rev search but no matches

After no matches after looking for a while I decided to move on, go over the other leads.

### Blue parking markings
Blue road markings for parking are used across several countries:

| Country | System | Notes |
|---------|--------|-------|
| **Italy** | *Strisce Blu* | Paid parking via meter (*Parcometro*) or mobile app |
| **Spain** | *Zona Azul / Zona ORA* | Short-term paid parking (1–2 h) |
| **France** | *Zone Bleue* | Free short-term parking with a parking disc |
| **Switzerland** | *Blue Zone* | Free short-term parking (1 h) with a parking disc |
| **Austria** | *Kurzparkzone* | Paid or parking disc, common in Vienna and Salzburg |

It was with this I started to get closer. I went between two coutries now, Italy and Spain. I have never been to Italy so wasn't sure about it I only went by search for buildings previously, however I have been to spain a few times and after a search for `spain blue parking spot` and i found a pretty good match on `mallorca`.

<img src="/SeemsFishy/mallorcaParklikng.png" alt="Res1" style="max-width: 500px; width: 100%;">



## Checking Mallorca
Being the dumbass I am I decided that I would check the whole coast line of mallorca and zoom in when I found a harbor that would match. I check it but could'n find any, Wasn't the best quality pictures.
I needed to be sure that the boats were positioned towards land and not along it.
When I did go to the street view I found the yellow lines, proving that I'm probabily correct, I just need to find the correct location won mallorca.

<img src="/SeemsFishy/MallorcaFollowcostline.png" alt="Res1" style="max-width: 500px; width: 100%;">



# Final
So after checking the maps I decided to go back to the one thing I got, the image of the lobster and this time I check it with reverse image search with `Mallorca` with it and this time I got a match. The match was from a instagram reel that along with other statues of animals, one fish, one octupus and the lobster.

<img src="/SeemsFishy/images.jpg" alt="Res1" style="max-width: 500px; width: 100%;">
<img src="/SeemsFishy/image2.jpg" alt="Res1" style="max-width: 500px; width: 100%;">
<img src="/SeemsFishy/image3.jpg" alt="Res1" style="max-width: 500px; width: 100%;">


However when I tried to see the reel i couldn't find it, it was redirected to some other post. The captions and the lighting of the three images suggests that it is taken at the same time and they look to be in the same style so I assumed that they are places someware together. So I know it is Mallorca, now i just need to know where. Doing searches with the other animals from the picture.
I even searched in spanish for `¿dónde puedo encontrar esculturas de langosta y pulpo? mallorca` (Where can I find lobster and octopus sculptures? mallorca) but still nothing.

Looking for statues of sea life in mallorca I came a cross this link where someone was asking where a fish statue was locates: `https://www.facebook.com/groups/mallorca.majorca/posts/2671893002997892/`

And checking the comments to it I got a name, aswell as some other images, one being the fish from the three images I got before.

***location: Puerto de Portocolom***

Search for this location I fould this link:
`https://www.komoot.com/highlight/4861111

IT shows the statues without the lobster but it have the other animals. 

---

Checking the location on maps I it is  clear that it is there, I just check it but over looked it when I did my inital coastline check. Looking at the image after I can see that there are trees behind the boats so instead of checking the entire coastline I could just have looked for a bay:

<img src="/SeemsFishy/Found.png" alt="Res1" style="max-width: 500px; width: 100%;">


- **Location:** Puerto de Portocolom, Mallorca, Spain
- **Coordinates:** `39.42402800667226, 3.2607722763750844`


<img src="/SeemsFishy/Malloericaconfirm.png" alt="Res1" style="max-width: 500px; width: 100%;">
<img src="/SeemsFishy/found2.png" alt="Res1" style="max-width: 500px; width: 100%;">

To see what animal the lobster replaced I checked with streetview but there is no picture of either the lobster or any other animal except the fish and the octupus aswell as a praw. Looking at the stone of the prawn its not that. However on the other picture on the `https://www.komoot.com/highlight/4861111` site you can see a sea horse and on the streetview the stone the lobster is  on theres nothing. Then the single neuron I have started and thought that the street view was taken between the switch between the lobster and the seahorse and submitting `seahorse` gives us a ***congratulations***

<img src="/SeemsFishy/InitalView.png" alt="Res1" style="max-width: 500px; width: 100%;">
<img src="/SeemsFishy/found3.png" alt="Res1" style="max-width: 500px; width: 100%;">
<img src="/SeemsFishy/found5.png" alt="Res1" style="max-width: 500px; width: 100%;">


---

**Country:** Spain  
**Island:** Mallorca  
**Location:** Puerto de Portocolom  
**Flag:** `seahorse`
