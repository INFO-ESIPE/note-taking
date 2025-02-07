[[Satellite Data Analysis]]
17/10/2024
****
**Table of Contents**
```table-of-contents
```

****
## Representation

In general, remote sensed data **coming from an imaging sensor** is represented as a numerical matrix. We can map the numerical values of the matrix to grey-values/coloured pixels if we need a more visually appealing result for our data.

![[matrix.png|350]]


***
## Spectral components

From a spectral component, we can measure several key properties:
- **Amplitude A**, which indicates the strength of the reflected signal, often represented as voltage. It tells us how much energy is reflected back to the sensor.
![[amplitude.png|400]]

- **Intensity**. Not important, it's similar to amplitude. 

- **Phase (Φ)**, helps us estimate the delay in the reception of the EM wave. The phase can provide information about the distance to the target and the properties of the reflecting surface.
![[phase.png|400]]

- **Polarisation**, which indicates the orientation of the oscillation of the electric and magnetic fields of the EM wave. Polarisation can be horizontal, vertical, or at any angle in between. 
	*As this degree can be random, we have to ensure it's value.*
![[polarization.png|400]]

By breaking down the incoming signal into several spectral components, we can measure each component separately. This allows us to obtain a vector of values representing the amplitude for each part of the EM wave.


***
## Resolution of Data

The resolution of data is based on the smallest scale we are able to compute.
	*For instance, we measure euro currency resolution on cents. We measure time in nanoseconds, picoseconds (or milliseconds if we are not able to compute such small values)...*

Different types of resolution are important in remote sensing:
1. **Temporal Resolution:** Frequency at which data is captured over time. For example:
    - High temporal resolution: Measurements taken less than 3 days apart.
    - Low temporal resolution: Measurements taken 16 days apart (typical for some satellites).
    
2. **Spectral Resolution:** Ability of the sensor to distinguish between different wavelengths of the EM spectrum. Higher spectral resolution provides more detailed data but requires more storage capacity and can impact data availability.
    
3. **Spatial Resolution:** Size of the smallest object that can be resolved in the image. It is determined by the **Ground Instantaneous Field Of View (GIFOV)**, which is the area on the ground that corresponds to a single pixel in the image. If a detail is smaller than the GIFOV, it will not be captured.
    *Higher spatial resolution results in a more detailed and larger matrix of data.*
    
4. **Radiometric Resolution:** Ability of the sensor to distinguish between different levels of intensity or amplitude. Higher radiometric resolution allows for more precise measurements of the signal strength.


***
## Mono-spectral vs. Multi-spectral

**Mono-spectral:** Refers to data captured in a single spectral band.
**Multi-spectral:** Refers to data captured in multiple spectral bands. Multi-spectral data provides more detailed information but requires more storage capacity.


***
## Quantisation

**Quantisation** is the way we store a finite approximation of the continuous range of amplitude the sensor could measure.
	*This is like an analog-to-digital convert function, as we have to turn those values into bits, like an audio*

The quantisation step is prone to errors, like any approximation.
Furthermore, the quality also depends on how precise we want our render to be
	*Colours render on 3 bytes will give [[02 - (Digital) Images#Representation|true colour]] output but will be heavy. Render on one byte is more lightweight but we lose details...*
