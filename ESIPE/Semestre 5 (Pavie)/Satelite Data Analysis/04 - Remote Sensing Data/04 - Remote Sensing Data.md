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

Out of a spectral component we *can* measure :
- **Amplitude A**, which indicates the voltage (how much is reflected to us)
![[amplitude.png|400]]

- Intensity. Not important, it's similar to Amplitude even though the returned value is not exactly the same, it corresponds to the same kind of value

- **Phase Phi**, helps us estimating the delay of reception of the EM wave.
![[phase.png|400]]

- **Polarisation**, which indicates the oscillation axis/degree (is it horizontal, vertical, blabla...) of both the electric field and the magnetic field.
	*As this degree can be random, we have to ensure it's value.*
![[polarization.png|400]]

We can break down the incoming signal in several spectral components that we measure separately. This allows us to obtain a vector of values which represents the amplitude for each part of the EM wave.

The resolution of data is based on the smallest scale we are able to compute.
	*For instance, we measure euro currency resolution on cents. We measure time in nanoseconds, picoseconds (or milliseconds if we are not able to compute such small values)...*
We analyse:
- Temporal resolution (time)
- Spectral resolution (wavelength)
- Spatial resolution (location and size)
- Radiometric resolution (amplitude/intensity)


Mono-spectral means there is only one band, unlike multi-spectral.
A high Spectral resolution provides more data, but requires a lot of storage capacity. This may also impact the availability of the data.


The spatial/geometric resolution (spatial detail) provides us a matrix. The more accurate and precise the spatial resolution is, the more detailed and large this matrix will be. This precision depends on the **Ground Instantaneous Field Of View (GIFOV)**. If a detail is smaller than the GIFOV, it will not be captured


**Quantisation** is the way we store a finite approximation of the continuous range of amplitude the sensor could measure.
	*This is like an analog-to-digital convert function, as we have to turn those values into bits, like an audio*

The quantisation step is prone to errors, like any approximation
Furthermore, the quality also depends on how precise we want our render to be
	*Colours render on 3 bytes will give [[02 - (Digital) Images#Representation|true colour]] output but will be heavy. Render on one byte is more lightweight but we lose details...*


**Temporal resolution** is the latency between the measures.
For instance, high temporal resolution are measurements separated by less than 3 days. Low temporal resolutions are the ones separated by 16 days, which is kind of a lot (typical for satellites).

