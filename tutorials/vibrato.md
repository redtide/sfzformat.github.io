---
title: Vibrato
---
Emulating vibrato for solo instruments or voices with LFOs is not difficult. The challenges
are understanding the key aspects of vibrato on the real instrument, and keeping the number of
parameters from growing too large to be easy to use. The examples here use SFZ 2 spec [numbered
LFOs](/modulations/lfo.html), rather than the [dedicated pitch, volume and filter LFOs](/opcodes/amplfo_depth)
and envelopes of SFZ 1. A lot of this can be done under the SFZ 1 specification as well, but
there are some limitations.

The most basic, typical vibrato is pitch vibrato - just an LFO modulating pitch. Making the
minimum and maximum rates and depths that would be used by players in real life is important,
of course. The numbers here are examples which would be decent for bowed strings - it has a
rate of 2-10 Hz and a maximum depth of 35 cents. Wider and slower are certainly possible
on real instruments, but isn't commonly used in performance.
```
lfo01_pitch_oncc111=35
lfo01_freq=2
lfo01_freq_oncc112=8
```
In real life, however, players and singers will often start a note without vibrato,
and add vibrato a fraction of a second later. This is where modulating the [LFO delay parameter](/opcodes/lfoN_delay)
becomes useful, and possibly [LFO fade](/opcodes/lfoN_fade) as well. Delay seems like enough for most wind
instruments and vocals, but having both delay and fade seems effective with bowed strings.
```
lfo01_pitch_oncc111=35
lfo01_freq=2
lfo01_freq_oncc112=8
lfo01_delay_oncc115=0.500
lfo01_fade_oncc116=0.500
```
Vibrato on many instruments doesn't just affect pitch; on bowed strings, it seems to have
a subtle effect on volume and timbre as well. We can have the same MIDI CC parameter control
how much the vibrato LFO affects pitch, volume, and an EQ band, with the latter two being
quite subtle.

This is a good a time as any to note that not all vibrato is equal - it's not really
practical to add vibrato to the lowest note playable on a cello, for example, or to a
natural harmonic. With pizzicato, vibrato is possible, but probably should not affect the
EQ band, and either only affect pitch or pitch plus a subtle effect on volume. The below
will work reasonably for most bowed notes, however.
```
lfo01_pitch_oncc111=35
lfo01_volume_oncc111=1
lfo01_freq=2
lfo01_freq_oncc112=8
lfo01_delay_oncc115=0.500
lfo01_fade_oncc116=0.500
eq1_freq=2000
eq1_bw=2
lfo01_eq1gain_oncc111=2
lfo01_eq1freq_oncc111=500
```
Vibrato can also be humanized, by varying the rate of the vibrato LFO. This can be done
by modulating the rate of the vibrato LFO with another LFO. The ARIA sample & hold
[waveform](/opcodes/lfoN_wave.html) can be used here, and the modulation depth controlled by MIDI CC, so when that's
at zero, no humanization happens.
```
lfo02_freq=1
lfo02_wave=12 //Sample & hold LFO waveform number
lfo02_freq_lfo01_oncc117=1
```
Or, to stay in the SFZ 2 spec and not use ARIA extensions, a sine wave with randomized
[starting phase](/opcodes/lfoN_phase) will also work:
```
lfo02_freq=1
lfo02_phase_oncc135=1
lfo02_freq_lfo01_oncc117=1
```
However, when playing multiple layers, such as sustain samples with crossfaded dynamics
or multiple mic positions, this can cause each layer's vibrato to drift out of sync and
sound like separate instruments. This is generally not desirable, so it is possible to
pseudo-randomize the starting phase using a non-random [CC, such as velocity](/extensions/midi_ccs.html)
(which is often otherwise unused in sustain sounds with crossfaded dynamics). If the SFZ
player can have a global sample and hold LFO which does not retrigger for each note, this
would also be a solution, though ARIA does not allow this.
```
lfo02_freq=1
lfo02_phase_oncc132=0.7
lfo02_freq_lfo01_oncc117=1
```
For additional complexity, it's also possible to have the random LFO itself modulate pitch,
which will create some pitch drift, and have more than two LFOs involved. Here is a fairly
sophisticated example.
```
//Vibrato
lfo01_pitch_oncc21=29 //Vibrato LFO
lfo01_freq=2 //Any slower than this sounds really lousy
lfo01_freq_oncc112=6 //8 Hz is about as fast as vibrato on cello can go
lfo01_delay_oncc115=0.500
lfo01_fade_oncc116=0.500
//This LFO also does tremolo
lfo01_volume_oncc21=1 //Not much - just a subtle effect on volume
eq1_freq=2200 //EQ band for vibrato
eq1_bw=2
lfo01_eq1gain_oncc21=3 //Again, pretty subtle

lfo02_wave=1 //Second LFO to make things wobblier
lfo02_phase=0
lfo02_phase_oncc131=0.7 //Phase affected by velocity, to pseudo-randomize while keeping both mics' LFOs in sync
lfo02_freq=0.01 //Basically no movement at very slow speeds, just randomization
lfo02_freq_oncc117=1 //Max rate is not very high, so it doesn't sound too obvious
lfo02_pitch_oncc117=6 //Slight pitch wobbliness
lfo02_freq_lfo01_oncc117=1 //Affect the rate of the other LFO for unsteady vibrato

lfo03_wave=1 //And a third LFO for secondhand complex wobbliness
lfo03_phase=0.4
lfo03_phase_oncc131=0.479 //Different phase response to velocity than the second LFO
lfo03_freq=0.5
lfo03_freq_oncc117=-0.4
lfo03_freq_lfo2_oncc117=1
lfo03_pitch_oncc117=-4
```
Something similar to the above will work fairly well for a range of strings and voices.
However, there are cases where vibrato should only go in one direction - for example,
bending guitar strings only moves the pitch upwards, while on saxophone it's possible
to play vibrato centered around the pitch, but most of the time players will go only
below the pitch. Let's use saxophone vibrato as an example.

To have vibrato which will go below the main pitch is simple - the LFO phase can be
set so the wave starts at the top, and the note [tuned](/opcodes/tune) down by the vibrato depth amount.
```
lfo01_pitch_oncc111=20
lfo01_phase=0.25
lfo01_freq=1.5
lfo01_freq_oncc112=6
pitch_oncc111=-20
```
This will work fine, as long as we don't try to apply delay or fade to the LFO, which
would result in the note starting out flat with no vibrato. To solve that problem, we
can combine the LFO with a pitch [envelope](/modulations/envelope_generators.html). Here is an example with just delay:
```
lfo01_pitch_oncc111=20
lfo01_freq=1.5
lfo01_freq_oncc112=6
lfo01_phase=0.25
lfo01_delay_oncc116=1
pitcheg_delay_oncc116=1
pitcheg_depth_oncc111=-20
```
To have the choice of idiomatic sax vibrato and violin-style vibrato centered around the pitch
can be done separate LFOs and separate depth controls. It's also possible to duplicate all the
regions and use [loccN/hiccN](/opcodes/loccN.html) to select between ones with different styles of vibrato.
```
lfo01_pitch_oncc111=20 //Sax vibrato LFO - goes down from the main pitch
lfo01_freq=1.5
lfo01_freq_oncc112=6
lfo01_phase=0.25 //Starts at top
lfo01_delay_oncc116=1
pitcheg_delay_oncc116=1 //Pitch envelope to drop the central pitch when sax vibrato kicks in
pitcheg_depth_oncc111=-20

lfo02_pitch_oncc114=20 //Violin vibrato LFO - goes below and above main pitch
lfo02_freq=1.5
lfo02_freq_oncc112=6 //Same rate as the first LFO
lfo02_phase=0.5 //Starts in the middle, goes down first before going up
lfo02_delay_oncc116=1 //Same delay, too
```
This covers jaw vibrato, but sax players also use diaphragm vibrato, which changes volume
and has no effect on pitch, which means there are now three vibrato depths. Having the
volume modulated by the second LFO is a little easier, as the phase setting of the first
LFO would mean having to apply a volume envelope as well.
```
lfo01_pitch_oncc111=20 //Sax vibrato LFO - goes down from the main pitch
lfo01_freq=1.5
lfo01_freq_oncc112=6
lfo01_phase=0.25 //Starts at top
lfo01_delay_oncc116=1
pitcheg_delay_oncc116=1 //Pitch envelope to drop the central pitch when sax vibrato kicks in
pitcheg_depth_oncc111=-20

lfo02_pitch_oncc114=20 //Violin vibrato LFO - goes below and above main pitch
lfo02_freq=1.5
lfo02_freq_oncc112=6 //Same rate as the first LFO
lfo02_delay_oncc116=1 //Same delay, too
lfo02_phase=0.5 //Starts in the middle, goes down first before going up
lfo02_volume=0 //This LFO also does tremolo
lfo02_volume_oncc113=3
```
There is one additional consideration with diaphragm vibrato - when
the volume of the note drops down, the breath noise can become more prominent, especially on
quiet notes or when using the subtone technique. If the volume of the breath noise can be
modulated separately, the noise regions should not be affected by pitch vibrato, and be
affected by the diaphragm vibrato in an opposite direction to the notes. So, if the above
vibrato settings are set under a [global header](/headers/global.html), the breath noise
sample regions could have settings similar to this.
```
lfo01_pitch_oncc111=0 //LFOs do not affect pitch
pitcheg_depth_oncc111=0
lfo02_pitch_oncc114=0
lfo02_volume_oncc113=-3 //Diaphragm vibrato affects volume in the opposite direction
```
Humanization and having the diaphragm vibrato affect timbre can be done similarly as with
the strings above, ensuring both LFOs are humanized in sync with each other, so they do not
drift apart.

Some instruments will have vibrato types which require special treatment, for example guitar
tremolo bridges will bend each string's pitch by a different amount when playing chords. This
requires different pitch modulation depths for each string. Vibrato can also be used to
modulate [filter cutoffs](/opcodes/cutoff), which is commonly used in synthesizers to create
evolving pads or wobble basses. This is not difficult to implement. Here is an example of a
synthesizer style vibrato with a typical lowpass filter, and vibrato which can affect pitch,
volume or filter cutoff.
```
//Filter
//Lowpass filter
cutoff=120
cutoff_cc120=13200
fil_keytrack=100
resonance=0
resonance_cc121=12

//Vibrato
lfo01_freq=1
lfo01_freq_oncc112=11
lfo01_delay_oncc115=0.500
lfo01_fade_oncc116=0.500
lfo01_pitch_oncc111=22 //Vibrato LFO affects pitch
lfo01_volume_oncc114=6 //Volume tremolo
lfo01_cutoff=0 //Filter wobble
lfo01_cutoff_oncc113=3600
```
This by no means exhausts all the possibilties of vibrato. It does provide a decent combination of control and realism for a lot of common instrument types.
