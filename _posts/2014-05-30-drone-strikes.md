---
layout: Post
title: US Drone Strike Map
---

# US Drone Strike Map


## Background

Over the last few months I’ve gotten more interested in plotting events on a map. This curiousity merges well with my anger over America’s covert wars in Yemen, Afghanistan, and Pakistan.

In particular, the use of “_unmanned combat air vehicles_” known more commonly as [drones](https://en.wikipedia.org/wiki/Combat_drone) to conduct, “drone strikes”. Drones have played a signifiant role in America’s covert wars. Their use on the battlefield has either been directly responsible, or aided air strikes in remote parts of Yemen, Pakistan, and Somalia.

## The Map

The map below, created with [TileMill](tilemill.com), [Mapbox](mapbox.com), and data provided by [dronestream](dronestre.am) paints an alarming picture.

Each point marks a reported strike. Points in brightest red, draw attention to strikes whose numbers killed exceeded 20 people.

The map is interactive. Clicking a point will provide you with the number of civilian deaths reported, as well as how number of children killed as a result of these attacks.

## Data & Methodology

This map was created with free, and publicly available data and tools. [Dronestream](dronestre.am) provides “Real-time and historical data about every reported United States drone strike”. The information is available as a `.json` API.

I used the [Raw JSON Feed](http://api.dronestre.am/data) dronesteam provided and converted the `.json` file a `.csv` using konklone’s [JSON to CSV coverter](http://konklone.io/json/).

After that, it was as easy as using a [TileMill’s tutorials](https://www.mapbox.com/tilemill/docs/crashcourse/point-data/) to plot the points a map.

Acquiring the tiles, of Yemen, Pakistan, and Somalia wasn’t relatively easy. I used a shapefile of roads provided free by [Diva GIS](http://www.diva-gis.org/Data).

* * *

If you have questions or comments about this map, please feel free to contact me via email at `hey.bernard [at] gmail [dot] com`.
