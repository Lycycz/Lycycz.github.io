---
layout: post
title:  "Couldn’t open audio device: No available audio device!"
date:   2021-05-11 14:00:00 +0800
---
[Couldn't open audio device: No available audio device](https://discourse.libsdl.org/t/couldnt-open-audio-device-no-available-audio-device/18499)

### built your own SDL
If you built your own SDL, you probably didn’t have development headersfor PulseAudio (or ALSA), so it’s trying to use /dev/dsp, which doesn’texist on many modern Linux systems (hence, SDL_Init(SDL_INIT_AUDIO)succeeds, but no devices are found when you try to open one). “`apt-get install libasound2-dev libpulse-dev`” and rebuild SDL…let the configurescript find the new headers so it includes PulseAudio and ALSA support.

### didn’t build your own SDL
If you didn’t build your own SDL, maybe you can force it to use adifferent audio path:

{% highlight shell %}
SDL_AUDIODRIVER=pulse ./mytestprogram
{% endhighlight %}

or

{% highlight shell %}
SDL_AUDIODRIVER=alsa ./mytestprogram
{% endhighlight %}

One of those two solutions will (probably) fix your problem.