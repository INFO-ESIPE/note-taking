[[Satellite Data Analysis]]
03/10/2024
****
**Table of Contents**
```table-of-contents
```

****
## Electromagnetic Waves

Remote sensing is about observing matter from afar. When this remote sensing comes to earth observation, we mostly uses satellites as a solution.

Satellites uses **Electromagnetic (EM) radiations** as a carrier of the information we are interested in—to us, the measurer—as it is the quickest propagation possible in the void, without the need of any physical support (wire, storage).
	*This offers us small delay.*

Long story short, an EM Radiation is a **wave that propagates in one specific direction**. This wave is made out of two fields:
- Electric Field (E); This field works even if there is no matter (charge) around
- Magnetic Field (M), in phase with E

Those waves are oscillating together, and are related. If we change the size of E, the size of M is affected.


The **wavelength** `𝜆` (lambda) of a radiation determines its behaviour when it interacts with matter.
	*Long wavelength, low frequency, low energy
	Short wavelength, high frequency, high energy*

All possible frequencies defines the **EM Spectrum**, even thought only a subpart of it is exploited:
- **UV** (Ultraviolet)
	- *Lower than 0,4 μm (micrometers)*
	- After Purple (Violet in French)
- **VIS** (Visible spectrum), the only part visible to our naked eye
	- *Purple being near 0,4 μm (ultraviolet)
	- Red close to 0,7 μm (infrared)
- **IR** (Infrared)
	- Above 0,7 μm *(Under Red)*
	- If the value is between 0,7 and 3 μm, the Infrared is **reflected**
	- If the value is between 3 and 15 μm, the infrared is **emitted** 
- **MW** (Microwave)
As a human can only see the visible spectrum, we build sensors in order to detect EM radiations which are outside of the visible spectrum. 


****
## What to capture

The ozone layer absorbs UV. The atmosphere—and the various molecules it is composed of—absorbs most of the EM radiations wandering around (this absorption varies depending on the wavelength).
	*For instance, UVs are fully absorbed, while EM waves on the VIS spectrum aren't.*

For this reason, you will never have UV censors for earth.
	*There are, however, UV sensors **outside the atmosphere** in order to observe other planets, as the EM aren't absorbed.*

Analysing matter from VIS waves is not a good idea, as we can't consistently recognise a material from its colouring. In general, we combine the result from several wavelengths in order to obtain a better approximation of the matter we attempt to analyse.
	*That is especially the case for analysing exoplanets—or say, Mars' surface for instance—as everything appears of an unique colour (red for Mars), thus not allowing to clearly understand the matter around engines exploring the planet.*


****
## EM Emittance

Any entity with a temperature above the absolute zero (> 0 Kelvin) emits radiations (due to microscopic thermal motion. This emittance is called **exitance**.
	*The higher the temperature, the greater the power emitted and the greater the movement.*

Planck law allows us to calculate the spectral emittance of an entity.
	The results heavily depends on the wavelength. If the wavelength (lambda) increases, the spectral emittance decreases.

Reflected infrared actually has two subsets, but only the **Near infrared** is interesting us as **its the only of the two we can retrieve data from** (the atmosphere absorbs waves on the other).


****
## Propagation Chain

An EM is in a course from the source to the target, then to the sensor:

![[propagation.png|400]]

We can distinguish at least 4 relevant phenomena from this

### Transmission

Radiation passes through a medium without any absorption or scattering
![[transmission.png|200]]

Materials may be transparent for some wavelengths and opaque for others (e.g., water is transparent to visible light but opaque to infrared).

### Scattering

Random redirection of radiation due to microscopic irregularities in the medium, such as air molecules, aerosols, or cloud droplets.
	*Scattering effect reveals all the elements (particles) wandering around in the air that might obfuscate waves. This effect is not visible when in plain light, as this haze effect is visible everywhere (scattering effect works better when, for instance, a small sunlight appears in a dark area, like a cavern).*

![[cavern.png|300]]

This effect generally reduces contrast in remote sensing data and introduces wavelength-specific distortions.

### Absorption

Radiation is absorbed by a medium or object and converted into another form of energy (e.g., heat) or re-emitted at a different wavelength.
![[absorption.png|200]]

It is useful for gathering information about the target’s material properties, as absorption is linked to its physical and chemical characteristics.
	*Some wavelengths are absorbed more readily than others (e.g., water absorbs infrared light but not visible light).*


### Reflection

Radiation bounces off the surface of an object and travels back toward the sensor.
![[reflection.png|200]]

> [!important]
> Reflection is critical in remote sensing as it provides the primary data about the target.**

The type and roughness of the material determines how the reflection behaves, which can be of several type:
- **Mirror Reflection**: Occurs on smooth surfaces; all radiation reflects in a single direction, often away from the sensor, leading to information loss.
![[mirror.png|400]]

- **Diffuse Reflection**: Happens on rough surfaces; radiation reflects in multiple directions, making it ideal for remote sensing as it carries information about the surface.
![[diffuse.png|400]]

- **Mixed Reflection**: A combination of mirror and diffuse reflection, common in real-world surfaces.
![[real.png|400]]

> [!tip]
> Rough material = Diffuse reflection
> Smooth material = Mirror-like reflection


Here is an example for vegetation, we see that wavelength strongly affects the reflection percentage due to its chemical composition:
![[vegetation.png|500]]


****
## Issue

As we have seen, reflectance is the basis for identification through remote sensing.
But, in the real world, **noise** produces inaccuracy in our measurements, as it affects the EM radiations.
	*Considerable disturbance factor, among few others*
