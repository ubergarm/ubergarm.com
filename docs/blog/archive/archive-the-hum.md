How It Starts
---

"Do you hear that? That noise... Kind of like a diesel truck running off in the distance... Maybe the neighbors are mowing with their tractor?"

"Wait be quiet. Hrmm. No... Wait, you mean the fridge that just came on?"

"No... That's strange you can't hear it. I'll google to see if there is a factory nearby or something. Huh, apparently its all over the world and known as 'The Hum'."

What is "The Hum"?
---
So apparently 2% of the population (often consisting of middle aged folks) describes hearing a low distant rumbling droning humming sound even though they live in a quiet rural area. 

I just moved to Fortunes Cove down in Nelson County, VA. I first started noticing it while sitting zazen (seated meditation) and now its hard to *not* notice it. Fortunately in my case it isn't too distracting unless I focus on it. It doesn't get in the way of watching my stories on netflix or anything horrible like that!

The [wikipedia article on The Hum](https://en.wikipedia.org/wiki/The_Hum) covers the idea in more detail and mentions famous locations associated with the hum including the Taos Hum and Bristol Hum. 

There's even a web site where you can report occurrences of this phenomenon: [The World Hum Map and Database Project](http://www.thehum.info/) with a few hits near my area.

The Fortunes Cove Hum
---
I set out this morning with my limited budget audiophile home studio equipment to record "The Hum".

[It sounds like this!](http://ubergarm.com/hum/the-fortunes-cove-hum-80hz-48db-lowpass.mp3)
*Note*: You need good headphones or stereo to hear the low frequencies

## Location-Time
* Fortunes Cove Nelson County, VA
  * Near [Mountain Cove Vineyards](http://www.mountaincovevineyards.com/)
  * Audible from [Fortunes Cove Nature Preserve](http://www.nature.org/ourinitiatives/regions/northamerica/unitedstates/virginia/placesweprotect/fortunes-cove-preserve.xml) hiking trails
* Monday April 18th, 2016 ~10 AM.

## Setup
* Lenovo ThinkPad W500
* Debian Linux 3.10 64bit kernel
* Audacity 2.0.5-1 audio editor/recorder
* Alesis MultiMix 4 USB Mixer
* RoadHog 24awg XLR Microphone Cable
* Audio-Technica AT2020 Cardiod Condenser Microphone
* Jolida Integrated Hybrid Stereo Amplifier JD1501RC
* Paradigm Reference Studio 60 v3 Loudspeakers
* Sennheiser HD380 Pro headset

## Procedure
1. Place microphone in window.
2. Setup mixer with flat eq, flat mix, and 75% pre-amp
3. Setup audacity to record 44.1khz 32-bit float from USB input
4. Record track with fridge running for baseline
5. Record track with no fridge, HVAC, water pumps, jet planes, etc.
6. Apply 80hz cutoff low pass filter with 48db per octave roll-off
7. Apply ~30db amplification

## Results
1. [Hum Raw w/ Fridge](http://ubergarm.com/hum/the-fortunes-cove-hum-plus-fridge-raw.wav)
1. [Hum Raw no known background noises](http://ubergarm.com/hum/the-fortunes-cove-hum-raw.wav)
1. [Hum 80hz lowpass filter](http://ubergarm.com/hum/the-fortunes-cove-hum-80hz-48db-lowpass.wav)

## Analysis
Raw spectrum analysis - Note peaks at ~12hz and 60hz (Electric AC noise?)
![](/content/images/2016/04/spectrum-peak-12hz.png)
Low Pass spectrum analysis
![](/content/images/2016/04/spectrum-low-pass-80hz-48db-per-octave.png)
Low Pass Waveform
![](/content/images/2016/04/spectrum-low-pass-80hz-48db-per-octave-waveform.png)

## Conclusion
The naive conclusion is that "it worked". The recording sounds like the noise that I personally hear. However, typical human hearing is in the 20Hz-20kHz range and much of the energy of this waveform appears to be in the subsonic range. The frequency response of this microphone and most audio equipment are not rated to go below 20hz.

So is this just over amplified background or electrical noise, or is this actually a very low frequency acoustic phenomenon which can be experienced as well as captured accurately?

Have you heard "The Hum"? Do you live near Fortunes Cove and experience anything similar? Please leave a comment!

## References
There are numerous reports and articles on "The Hum". This one is newer and has pretty photos:

[Taos Hum](https://newrepublic.com/article/132128/maddening-sound)
