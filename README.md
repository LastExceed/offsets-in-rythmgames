# An in-depth guide to offsets in rythm games

## Terminology:
- **"latency"** refers to the amount of time that passes between cause and effect
- **"offset"** refers to the amount of time by which we shift a predictable event in time to compensate latency
- **"lag"** is a very vague term that can mean many things which is why you should try to avoid using it

These terms are often used synonymously, but it is important to understand that they actually describe problem and countermeasure respectively.

## Latency types

There are 4 main types of latency: *(simplified)*
- **"input latency"** is the time between you making an input (e.g. physically pressing a key on your keyboard) and the game recognizing that you did
- **"audio latency"** is the time between the computer telling the speaker to output a sound and you hearing it
- **"video latency"** is the time between the computer telling the monitor to display something and you seeing it
- **"processing latency"** is the time the software takes to react to a signal

*Note: I am simplifying a lot here. Initially I wanted to get more into detail about latencies and how to minimize them, but turns out that's a crazy complex topic. [NVIDIA has a great overview of the individual components that affect input-to-visual latency](https://www.nvidia.com/en-us/geforce/news/reflex-low-latency-platform/#next-level-system-latency-advanced-section), but be aware that even that is just scratching the surface. The rabbit hole goes deeeeeep*

"Input lag" has become a big topic in competitive gaming lately, but the chosen term to describe it is less than ideal. What people are usually referring to is actually the latency between the input (eg a mouse-click) and the effect showing on screen, which is the combination of input-, processing-, and video-latency. In the end this is what the player experiences and thus wants to optimize, but for that it is important to understand the individual components that add up to the end result, especially in rythm games where "end-to-end" can mean either "input-to-visual" or "input-to-sound" depending on your play style.

So instead I'm going to talk about what we can do to combat latency:

## Offsetting

The idea is that if we know in advance that a certain thing needs to happen at a set point in the future and that the information that it happened will reach the player late because of latency, we can simply make it happen that much earlier than it actually should so the information that it happened reaches the player in just the right moment. Example:

Suppose we play some music in a rythm game. We know that the beat will kick 70ms from now, and that the player is supposed to hit a note at that point. When the player presses a key, the game plays a hitsound. However the system the player plays on has 50ms audio latency. For simplicity sake we will assume that all other latencies are 0ms and that the player has perfect aim.

If the player aims by audio then they will hear the beat 120ms from now and do the input in that moment, then get frustrated that the hitsound plays at 170ms from now and that the game says they hit 50ms too late even though they hit perfectly on time according to their perception.

We can fix this by using an audio offset. We already know what the rest of the song is and when it is supposed to play, so we simply play the music 20ms from now (50ms too early) while leaving the note where it is, then let the audio latency have its effect and get a resulting gameplay experience where music and note are perfectly synchronized.

But what about the hitsound ? Remember how i said "predictable event" in the terminology section? We don't know in advance when exactly the player will actually press the key (or if at all) so we can't predict at which point in time we need to play the hitsound in order to synchronize with the input. All we can do is play it the moment we get the signal that the player made an input, at which point we're already doomed to be 50ms late. Audio offset cannot affect hitsounds, which is why many rythm gamers turn off hitsounds and instead listen to their physical keytaps, which aren't affected by audio latency in the first place.

## Audio offset: global vs local

*I will talk about osu! here, but the concepts apply to all rythm games*

"global offset" shifts music in time and applies to all beatmaps. it is the option you should use to offset audio latency. It's value should always be negative, I can't think of any scenario where using a positive global offset makes sense.

"local offset" shifts notes in time and is beatmap specific. only use this if you think the mapper timed the beatmap incorrectly. If you find yourself changing it on every map you play, you should probably change your global offset instead.

## Visual offset

*So far the only game i've seen with this option is StepMania*

Since the arrows move at a constant speed we can predict where they will be in the future, and thus we can offset any visual latency we may have. However here too is a catch just like with the hitsounds in the previous example: We can't predict when (or if at all)the player will press a key causing flashy light to appear and notes to disappear. These effects are therefore unaffected by visual offset. here's an example of what that looks like: https://youtu.be/iaqs6uQg1JI

## Input offset

*I know that RoBeats has this pre-configured, and I think StepMania has a setting for this somewhere as well*

Actual input latency is the worst, because it brings forth both types of artifacts mentioned above. Since all user input is unpredictable we can't use any kind of prediction here. Instead, we shift the time windows that the player needs to hit to be after the note.

*I really can't recommend using this, EVER. Do yourself a favor and get a proper input device instead*

## Measuring latency

Measuring each type of latency individually is difficult and requires special equipment. However for the purpose of synchronizing our gameplay that isn't really necessary. In the end we time our perceived input to perceived output, so we're gonna measure and offset the delay between these two events. The perceived input is the finger pushing a button, the output is either the music you hear or the visuals you see depending on how you aim:

**aiming by audio:**    
This is how rythm games are supposed to be played. All we need for our measurement here is a microphone (headset will do) and an audio editor (eg [Audacity](https://www.audacityteam.org) is free and perfectly suitable).

Since latency depends on both hardware and software, everyone needs to make these measurements themselves, for every game, on every setup. Again, I will use osu! for an example.

Choose a skin with a clicky hitsound, enable NoFail and enter any mania or taiko map. turn music volume to 0% and hitsounds to 100%. Then put your microphone between speaker and keyboard and start recording. Slam a key and make sure that that both the physical keypress and the digital hitsound can be heard on the recording. Also hold the key briefly before releasing so that keyDown and keyUp are clearly distinguishable on the recording. Then look at the waveform and measure the time that passed between keyDown and hitsound. That number (in negative) should be your global offset. Example: https://i.imgur.com/RdUqlov.png

**aiming by video:**    
This is how competitive players often play. For this scenario we basically do the same thing as above but with a highspeed camera instead of a microphone.

Record pushing a button and getting a visual reaction on screen, then count the frames between the two events. Example: https://youtu.be/P1FJGE0_1Tg

## Notes

- Before offsetting latency, try to reduce it first. check "turn off all enhancements" in windows sound device properties, update drivers, play in exclusive fullscreen, disable vsync, yadda yadda

- Don't bother getting millisecond precision. Latency fluctuates a bit and beatmaps aren't timed that accurately anyway. especially sub-ms precision is pointless because the computer mostly doesn't even read or process user input that accurately, and the .osu fileformat rounds all timings to full ms anyway

- And always remember: Play more
