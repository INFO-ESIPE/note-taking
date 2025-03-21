[[Satellite Data Analysis]]
15/10/2025
****
**Table of Contents**
```table-of-contents
```

****
## Sensors

We call a **sensor** a device that can record incoming radiance. This radiance is an amount of energy emitted by the surface, or energy reflected by another source, such as another sensor.

Sensors can be classified as **active** or **passive**:
- **Passive:** Collecting incoming EM radiations (information that is not generated by the sensor itself). It relies on an **opportunity emitter** (The sun for instance)
	*Small energy consumption, and has a behaviour that relies heavily on the source
	This includes photo cameras, video cameras, radiometers, thermal remote sensors...*

- **Active:** Sensor that generates their own EM waves to illuminate the target and, ultimately, trigger a reaction it will collect
	*Requires more power than a passive sensor, but work 24/7 in general
	This includes radars, LIDAR, sonar...*

Active sensors are generally less dependent on external conditions, which makes their data more consistent over time.


****
## Passive Sensors

Passive sensors often use **pass-band filters** to focus on a specific **central wavelength** within the EM spectrum. However, these filters also capture a range of wavelengths around the central wavelength, known as the **bandwidth**.
*Capturing a single wavelength is often insufficient for meaningful data analysis. A range of wavelengths provides more comprehensive information.*

> [!example]
> Imaging spectrometers generate images by capturing multiple spectral bands, allowing for detailed analysis of the target area.

Thermal sensors are measuring the thermal radiations. 
The wavelength of thermal radiation is relatively long because the Earth does not emit a significant amount of heat. To ensure accurate measurements, thermal sensors often use a wide wavelength range to capture as much thermal energy as possible, despite the presence of noise.


****
## Active Sensors

### LIDAR

LIDAR (Light Detection and Ranging) is a laser that is put on a platform (often an airplane).
It is a **point-wise measurement** that focuses on one point of the surface of the earth, then another, etc.
This allows to detect—and determine distance/range of—entities compared to the laser's position via **pulses**.
	*Unlike a laser pointer we can buy, this laser does not work on the VIS wavelength (as it would be dangerous) but in the infrared one.*

How it works: LIDAR emits laser pulses and **measures the time it takes for the pulses to return after reflecting off a surface**. This allows it to calculate the distance to the object.

Airborne LIDAR is used to obtain a **Point Cloud** of an area (3D-Shaped fuzzy render of points in different positions). 
	*This render is often colour-coded to clearly indicate elevation: blue represents the lower parts (grass for instance), and red are the tall ones (skyscrapers for instance).*
![[lidar.png]]

From the point cloud, two important models can be derived:
- **Digital Surface Model (DSM):** Represents the elevation of the surface, including objects like buildings and vegetation.
- **Digital Terrain Model (DTM):** Represents the elevation of the bare ground, excluding objects like buildings and vegetation. This is achieved by analysing the second return of the laser pulse, which often penetrates vegetation and reflects off the ground.
(more on that [[05 - Data Formats and Visualisation#DEM/DSM/DTM|here]])

Accuracy of LIDAR measurements is typically within a radius of 0.5 meters, which is sufficient for detailed 3D modelling. The sensor captures the average height of objects within this radius, rather than the height of a single point.

### Altimeters

**Altimeters** are sensors used to measure altitude, or height above a reference surface, such as sea level. They are commonly used in aviation, space exploration, and oceanography.
- **How they work:** Altimeters measure the time it takes for a signal (usually radar or laser) to travel to the ground and back. This time delay is used to calculate the distance to the surface.

> [!info]
> In fact, LIDAR is a type of Altimeter known as laser altimeter.
