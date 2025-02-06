[[Satellite Data Analysis]]
03/10/2024
****
**Table of Contents**
```table-of-contents
```

****
## Sensors

Begin by understanding how we can have sensors in space, how they work, technical difficulties that may appear etc
The sensor is always attached to a "platform" (ground, balloon, plane, drone...).

### Ground-based sensing

A **weather radar** is always on the ground, looking at the atmosphere and collecting various information nearby.
	*Same for radars, GPR...*

### Ground-based images

Often retrieved from online sources (social medias, StreetView, Mapillary...)
	*Usually just photos of buildings from the street, a drone, a plane... That's what we use for OSINT in general, but it can still be useful for our field*

### HAPs

**HAPs** = High Altitude Platforms. 
Usually in standby at a **height greater than airplanes, but lower than satellites**. They have engines on the side that keeps them in position. 
	They can provides continuous data (videos), unlike satellites.

### Spaceborne sensing
*This course will focus on **Spaceborne Remote Sensing**, so this is of interest.*

They provide **periodic acquisitions**, not continuous (there is a break between each capture...)

Biggest advantage of this solution is that satellites—even though they are expensive to build and put into orbit—lives there for an extended period of time, ending in a **low per-datum cost** as we can collect thousands of photos for 10 to 15 years.

**Space segment** (satellite) possess a very **low storage capacity**. Data from the sensors is volatile and must be transferred to the ground segment quickly. 
**Ground segment** (mission control on earth) is mostly here to remotely supervise the satellite 
	*back up data, know where the satellite is, where it is taking the images from, how to orientate it ...*


****
## Orbit

There is the circular orbit, which is the most common (always the same distance from the Earth). But it is also possible to go for an elliptical orbit if we want to take photos from a further distance.

The angle (based on the equator) defines the degree of the orbit (can go from North to South, East to West, etc).

Orbits can be classified by three types (depending on their height):
- Low earth orbits (150 – 2000 km)
- Mid earth orbits (2000 – 35780 km)
- High earth orbits (≥ 35780 km)

Low and Mid can use any degree, while high ones can only be on the 0° (equator plane) as it helps to maintain the distance on a 24h basis. We call it **Geostationary**.
	*Satellite always face the same point on earth*

It is also possible to synchronise the movement of the satellite with the sun. This **Sun-synchronous orbit** allows to keep the same shadows and lighting between different photos it takes
	*this does not, however, prevents the atmosphere changes between images... obviously*

We call the **nadir point** the point of the Earth which is right under the satellite (vertically):
![[nadir.png|399]]

**Swath** represents the surface strip that a satellite can cover as the earth moves. This succession of nadir points provides a continuous zone that can be captured.
	Where **azimuth** is the motion direction
![[swath.png|400]]
