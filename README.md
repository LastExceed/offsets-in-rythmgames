# An in-depth guide to offsets in rythm games

I have yet to find a good explanation of the matter on the internet, so I decided to write one myself. The goal is to not only get the system properly configured, but also understand what is actually happening.

## Terminology:
- **"latency"** refers to the amount of time that passes between cause and effect
- **"offset"** refers to the amount of time by which we shift a predictable event in time to compensate latency
- **"lag"** is a very vague term that can mean many things which is why you should try to avoid using it

These terms are often used synonymously, but it is important to understand that "latency" and "offset" actually describe problem and countermeasure respectively.

## Latency types

"Input lag" has become a big topic in competitive gaming lately. Unfortunately the chosen term to describe it is vague and confusing, especially considering that the similar sounding "input latency" is only a tiny portion of that.

There are 4 main types of latency:

- **"input latency"** is the time between you making an input (e.g. by physically pressing a key on your keyboard) and the game recognizing that you did
- **"audio latency"** is the time between the computer telling the speaker to output a sound and you hearing it
- **"video latency"** is the time between the computer telling the monitor to display something and you seeing it
- **"processing latency"** is the time the software takes to react to a signal

![](https://i.imgur.com/zPkJvRR.png)


When people talk about "input lag", they are usually referring to end-to-end latency (in most cases input-to-video):

* **"input-to-video latency"** is the time between physically performing an input and seeing a reaction on screen
* **"input-to-audio latency"** is the time between physically performing an input and hearing a reaction from the speakers

Initially I wanted to go more into detail about what the individual components of each latency type are and how to minimize them, but the more time I spent researching this topic the more I had to realize that it would be WAY beyond the scope of this guide. If you want to get an idea of how crazy complex this topic is, check out [NVIDIA's breakdown of input-to-visual latency](https://www.nvidia.com/en-us/geforce/news/reflex-low-latency-platform/#next-level-system-latency-advanced-section), but be aware that even that is only an overview. The rabbit hole goes deeeeeep

So instead I'm going to talk about what we can do to compensate latency:

## Offsetting

The idea is that if we know in advance that a certain thing needs to happen at a set point in the future and that the information that it happened will reach the player late because of latency, we can simply make it happen that much earlier than it actually should so the information of it happening reaches the player in just the right moment. 

Here's an example:

Let's say we play some music in a rythm game. We know that the beat will kick 70ms from now, and that the player is supposed to perform an input in that moment. When the player presses a key, the game plays a hitsound and flashes a light. The system the game is running on on has 40ms input-to-audio latency, and for the sake of this example we'll assume that the player has perfect aim.

If the player aims by audio then they will hear the beat 110ms from now and perform the input in that moment, then get frustrated that the hitsound plays at 150ms from now and that the game says they hit 40ms too late even though they hit perfectly on time according to their perception.

We can fix this by using an audio offset. The computer already knows the entire song in advance, and thus when the beat will kick. we simply play the music 40ms too early while leaving the note where it is, then let the audio latency have its effect which nullifies this shift and get a resulting gameplay experience where beat and note are perfectly aligned.

But what about the hitsound ? Remember how i said "predictable event" in the terminology section? We don't know in advance when exactly the player will actually press the key (or if at all) so we can't predict at which point in time we need to play the hitsound in order to synchronize it with the input. All we can do is play it as soon as we get the signal that the player made an input, at which point we're already doomed to be 40ms late. Audio offset cannot affect hitsounds, which is why many rythm gamers turn off hitsounds and instead listen to their physical keytaps which aren't affected by audio latency in the first place.

The same of course applies to aiming by video: the beat kick corresponds to the visuals indicating that you need to perform an input, and hitsounds correspond to the flashing lights.

## Audio offset (a.k.a. "global offset")

*This option can be found in most rythm games*

global audio offset shifts all music in time. It is the option you should use to compensate audio latency. It's value should always be negative, I can't think of any scenario where using a positive global offset makes sense.

Keep in mind that hitsounds are unaffected as they aren't predictable, so if you're the kind of player that aims by hitsounds instead of keytaps then you shouldn't touch this option.

Fully keysounded maps are completely unaffected by this option.

## local offset

*Games that have this option include: osu!, StepMania, Quaver, and more*

local offset is configured for each song individually. It shifts  the notes in time and should be used if you think the mapper timed the beatmap incorrectly. If you find yourself changing it on every song you play, you should probably change your global offset instead.

## Video offset

*So far the only game i've seen with this option is StepMania*

Since the arrows move at a constant speed we can predict where they will be in the future, and thus we can offset any visual latency we may have.

This option too has a catch just like audio offset does with hitsounds: We can't predict when (or if at all) the player will press a key causing flashy light to appear and notes to disappear. These effects are therefore unaffected by visual offset. [Here's an example of what that looks like](https://youtu.be/iaqs6uQg1JI)

## Input offset

*I know that RoBeats has this pre-configured, and I think StepMania has a setting for this somewhere as well*

Actual input latency is the worst, because it brings forth both types of artifacts mentioned above. Since all user input is unpredictable we can't use any kind of prediction here. Instead, we shift the time windows that the player needs to hit.

*I really can't recommend using this, EVER. Do yourself a favor and get a proper input device instead*

## Measuring latency

Measuring each type of latency individually is difficult. However for the purpose of synchronizing our gameplay that isn't really necessary. In the end we time our perceived input to perceived output, so we're gonna measure and offset the delay between these two events (end-to-end latency). The perceived input is the finger pushing a button, the output is either the music you hear or the visuals you see depending on how you aim:

**aiming by audio:**    
This is how rythm games are supposed to be played. All we need for our measurement here is a microphone (headset will do) and an audio editor (e.g. [Audacity](https://www.audacityteam.org) is free and perfectly suitable).

Since latency depends on both hardware and software, everyone needs to make these measurements themselves, for every game, on every setup. Again, I will use osu! for an example.

Choose a skin with a clicky hitsound, enable NoFail and enter any mania or taiko map. turn music volume to 0% and hitsounds to 100%. Then put your microphone between speaker and keyboard and start recording. Slam a key and make sure that that both the physical keypress and the digital hitsound can be heard on the recording. Also hold the key briefly before releasing so that keyDown and keyUp are clearly distinguishable on the recording. Then look at the waveform and measure the time that passed between keyDown and hitsound. That number (in negative) should be your global offset. ([example](https://i.imgur.com/RdUqlov.png))

**aiming by video:**    
This is how competitive players often play. For this scenario we basically do the same thing as above but with a highspeed camera instead of a microphone.

Record pushing a button and getting a visual reaction on screen, then count the frames between the two events and set your video offset to the elapsed time. ([example](https://youtu.be/P1FJGE0_1Tg))

## Notes

- Before offsetting latency, try to reduce it first. check "turn off all enhancements" in windows sound device properties, update drivers, play in exclusive fullscreen, maximize fps, disable vsync, yadda yadda

- Don't bother getting millisecond precision. Latency fluctuates a bit and beatmaps aren't timed that accurately anyway. especially sub-ms precision is pointless because the computer mostly doesn't even read or process user input that accurately, and some games (e.g. osu!) round all timings to full ms anyway

- And always remember rule #1 for self-improvement: Play more
