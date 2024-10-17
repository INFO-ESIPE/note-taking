[[Web And Multimedia Technologies]]
17/10/2024
****

Perception of sound is due to variations in air pressure around the tympanum.
	*When an object vibrates, it produces shifts of nearby air molecules that progressively propagate and arrive at our tympanum, which in turn vibrates.*

In time, the pressure variations produces a **sound wave**.

Audio is defined by 3 parameters (like colours) :
- **Volume** : Defined by the extent of pressure variation of the sound wave
- **Pitch** : This value is given by the frequency of the sound wave (how fast the pressure varies)
- **Timbre** :Given by the spectrum, it is the shape of the wave


****
## Digital 

In order to be processed numerically (via a computer), the sound wave must be :
- **Sampled** : Measure volume at regular — and short — time intervals, called **Sampling frequency** (measured in hertz)
- **Quantized** : Finding correspondence between each measure and a volume
from a finite set of possible volumes

A digitised sound is simply a sequence of bits.


An Audio CD has a sampling frequency of 44100 Hz (volume is measured 44100 times per second).


As digital signal is only composed of binary values, it is very rarely altered by noise (as this noise will not reach the other value). Unlike analog signal where noise affects the audio quality a lot. 
![[digital.png]]

*Noise is visible in red. As we can see, the value will still be considered like the original intended values as the noise is not strong enough to reach the opposite binary value.*


****
## Audio Formats

MPEG-1/MPEG-2 Audio Layer III (MP3) is the most common audio format, as it is very suitable for the internet and allows audio encoding on small file sizes.

MP3 audio quality is alike CD quality. It offers a lossy data compression based on 


Advanced Audio Coding (AAC) was designed to be a successor of MP3. It is part of the MPEG-2 and **MPEG-4** specifications.
