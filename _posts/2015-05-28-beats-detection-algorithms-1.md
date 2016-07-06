---
layout: post
title: Beat Detection Algorithms (Part 1)
description: Two simple beat detection algorithms and their Scala implementation
keywords: sound processing, BPM, audio, scala, tempo, algorithms
---


You can think to the beat of a song as the rythm you tap your foot at while 
listening to it. The tempo of a song is usually measured in beats per minute. 
The tempo of a song, its beats, are usually felt by a listener and drive him, 
for instance, to dance according to the song's rythm. As it is often the case, 
things that a human can feel are not easy to detect through a computer 
program. This is the first of 2 posts on beat detection algorithms in which I introduce 
two simple algorithms I implemented in Scala [scala-audio-file](https://github.com/mziccard/scala-audio-file) library.  
Please notice that so far the library is only able 
to process WAV files.

###Sound Energy Algorithm 

The first algorithm I will talk about was originally presented 
[here](http://archive.gamedev.net/archive/reference/programming/features/beatdetection/index.html). I implemented it in the class 
`SoundEnergyBPMDetector`.
The algorithm divides the data into blocks of samples and compares the energy 
of a block with the energy of a preceding window of blocks. 
The energy of a block is used to detect a beat. If the energy is above a certain 
threshold then the block is considered to contain a beat. 
The threshold is defined starting from the average energy of the window of 
blocks preceding the one we are analyzing.  

If a block _j_ is made of 1024 samples and the song is stereo, its energy can be computed as:

\\[ E _ j = \sum _ {i=0} ^ {1023} left[i]^2 + right[i]^2 \\]

The block's energy is then placed in a circular buffer 
that stores all the energy values for the current window. Let us assume that 
the current window is made of 43 blocks (\\(43 \cdot 1024 = 44032\\) ~ \\(1s\\) with a sample rate of \\(44100\\)), the average window energy can be computed as: 

\\[ avg(E) = \frac{1}{43} \sum _ {j=0} ^ {42} E _ j \\]

We detect a beat if the instant energy \\(E _ j\\) is bigger than \\(C \cdot avg(E)\\). 
In the [original article](http://archive.gamedev.net/archive/reference/programming/features/beatdetection/index.html), a 
way to compute \\(C\\) is proposed as a linear regression of the energy variance in the corresponding window. 
In the `SoundEnergyBPMDetector` 
[class](https://github.com/mziccard/scala-audio-file/blob/master/src/main/scala/me/mziccard/audio/bpm/SoundEnergyBPMDetector.scala) 
I use a slightly modified equation that lowers the impact of variance: 

\\[ C = -0.0000015 \cdot var(E) + 1.5142857 \\]

In general, however, the bigger the variance the more likely we consider a 
block to be a beat. The variance inside a window of blocks is defined as: 

\\[ var(E) = \frac{1}{43} \sum _ {j=0} ^ {42} (avg(E) - E _ j)^2 \\]

This first algorithm is very easy to implement and fast to execute. 
However, the results computed are rather imprecise. To evaluate it add the 
[scala-audio-file](https://github.com/mziccard/scala-audio-file) library 
to your project and instantiate the `SoundEnergyBPMDetector` class as: 

```scala
val audioFile = WavFile("filename.wav")
val tempo = SoundEnergyBPMDetector(audioFile).bpm
```

###Low Pass Filter Algorithm

This second algorithm is based on a 
[post](http://joesul.li/van/beat-detection-using-web-audio/) 
originally hosted on Beatport's engineering blog by Joe Sullivan. 
The original article uses the 
[Web Audio Api](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API) 
to implement browser-side beat detection. A similar but more extensible 
implementation of the algorithm is provided by the class `FilterBPMDetector`. 
First, the algorithm applies to sampled data a biquad low-pass filter 
(implemented by the class `BiquadFilter`). Filter's parameters are computed 
according to the "Cookbook formulae for audio EQ biquad filter coefficients" 
by Robert Bristow-Johnson. 
Filtered audio data are divided into windows of samples (whose dimension is 
the sample frequency, i.e. 1s). A peak detection 
algorithm is applied to each window and identifies as peaks 
all values above a certain threshold (defined as \\(C \cdot avg(window)\\) where \\(C\\) 
is for the user to be configured, e.g. \\(0.95\\)). 
For each peak the distance in number of samples between it and its neighbouring 
peaks is stored (10 neighbours are considered). 
For each distance between peaks the algorithm counts how many times it has been detected across all windows in the song. 
Once all windows are processed the algorithm has built a map `distanceHistogram` 
where for each possible distance its number of occurrences is stored:

```scala
distanceHistogram(distance) = count
```

For each distance a theoretical tempo can be computed according to the 
following equation:
\\[ theoreticalTempo = 60 / (distance / sampleRate) \\]
Here the algorithm builds on the assumption that the actual tempo of the song 
lies in the interval [90, 180]. Any bigger theoretical tempo is divided by 2 
until it falls in the interval. Similarly, any smaller tempo is multiplied by 2. 
By converting each distance to the corresponding theoretical tempo 
a map `tempoHistogram` is built where each tempo is 
associated the sum of occurrences of all distances that lead to it:

```scala
tempoHistogram(tempo) = count
```

The algorithm computes the `tempoHistogram` map and selects as 
the track's tempo the one with the highest count.  

This second algorithm is also very fast and provides better results than 
the first one. To evaluate it, add the 
[scala-audio-file](https://github.com/mziccard/scala-audio-file) library 
to your project and instantiate the `FilterBPMDetector` class as: 

```scala
val audioFile = WavFile("filename.wav")
val filter = BiquadFilter (
      audioFile.sampleRate,
      audioFile.numChannels,
      FilterType.LowPass)
val detector = FilterBPMDetector(audioFile, filter)
val tempo = detector.bpm
```

### Conclusion

To sum things up, both algorithms are very fast to implement and execute. The 
second one is a bit slower but provides better approximations of the actual 
tempo. More time consuming algorithms that compute almost exact 
results can be developed by exploiting the Fast Fourier Transform or the Discrete Wavelet Transform. 
I will discuss such algorithms in the next post.
