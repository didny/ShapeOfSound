



# P5 Sound USB-MIDI tutorial

## 
In this tutorial, we will use [p5.sound](https://p5js.org/reference/#/libraries/p5.sound), the WebAudio framework of p5.js, to create a simple sound playback mechanism triggered by a MIDI device. This mechanism can be extended to create a more complex USB-MIDI instrument that combines sound synthesis and sensor inputs. Let's start by creating a sound that can be programmatically triggered and controlled.

## P5.sound

To use [p5.sound](https://p5js.org/reference/#/libraries/p5.sound) library with p5.js, insert following line to index.html to
load the library. If you are using editor.p5js.org site, its already included. 

```html
  <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.4.1/addons/p5.sound.min.js"></script>

```
---
## 01. Playback Audio file

<!--SoundFile 01 p5.js -->

https://editor.p5js.org/didny/sketches/BOHN1rJCl


```js
let snd = loadSound("soundfile.mp3"); //load a soundfile
snd.play(); // play loaded sound file
```

Loading large sized audio files may take a while, so it is advisable to preload files before the sketch is started using `preload()` function, or use a callback to process them after the loading has completed.

```js
let snd; 

function preload(){
 snd = loadSound("soundfile.mp3");
}

function setup(){
 snd.play();   
}

```
### Basic operations of sound files

```js
snd.play(); // play sound file
snd.loop(); // loop sound fine

snd.pause();
snd.stop();

snd.setVolume(vol); // set loudness
snd.rate(rate); //set playback speed
snd.pan(pan) //Panning sound to right or left.

snd.isPlaying() // returns true if the soundfile is playing

```
> Challenge01: 
>  Try loading your own sound file and play.

---
## 02.Playback Loops 

https://editor.p5js.org/didny/sketches/_vxZeCHcJ

If snd.play() or snd.loop() is called during playback, more sounds will be played over the other sounds. If you want to play only a single sound, you need to check with isLooping() whether a sound is playing and stop() to stop the sound playing.

```js
      // toggle drum loop
      if (drums.isLooping()) { 
        drums.stop();
      } else {
        drums.loop();
      }
```


---
## 03.Keyboard Piano 

https://editor.p5js.org/didny/sketches/e7EYhhewZ

If the frequency of a sound file is known, the scale can be played by adjusting the playback speed by a ratio to the target frequency. 

```js

      let notes = {
        //note to frequency table
        C4: 261.63,
        D4: 293.66,
      };

      mySound.rate(notes.C4 / 440.0); //set playback rate to C4
      mySound.play();

```

> Challenge01: 
>  Update the sample code to enable all 12 notes.
> https://gist.github.com/stuartmemo/3766449

---
## 04.Playback Sequence

https://editor.p5js.org/didny/sketches/U73IZ8-lq


```js
let sequence = ["C4", "D4", "E4", "F4", "E4", "D4"];
let step = 0;

let isPlaying = true;
let timer = 0;
let interval = 200;

  // if the amount of time that
  // has passed is greater than the
  // interval, execute the code
  // in the conditional statement
  if (isPlaying) {
    if (millis() - timer > interval) {
      console.log("Step", step);

      let pitch = notes[sequence[step]] / 440.0;
      let volume = (1 - mouseY / height) * 2;
      mySound.rate(pitch);
      mySound.setVolume(volume);
      mySound.play();

      step = ++step % sequence.length;
      // reset the timer to equal
      // the current number of milliseconds
      timer = millis();
    }
  }
```
###  [p5.Part](https://p5js.org/reference/#/p5.Part)
https://editor.p5js.org/didny/sketches/55xSauqCJ

With [p5.Part](https://p5js.org/reference/#/p5.Part), you can control the playback of multiple sequence phrases. However, the tempo timing is not very precise.
If you need to implement exact timing like a step sequencer, Tone.js is a good option.
https://tonejs.github.io/examples/stepSequencer


---
## 05.Sound Effect (Delay)
https://editor.p5js.org/didny/sketches/6XqHr_sVU

In p5.sound, p5.Effect objects can be used to add acoustic effects such as reverberation and delay to the played sound.

For example, the p5.Delay object can create an echo effect.

```js

       //Create an instance of the p5.Delay object 
      delayEffect = new p5.Delay();

      mySound.play(); //play some sound
      // delay.process() accepts 4 parameters:
      // source, delayTime (in seconds), feedback, filter frequency
      delayEffect.process(mySound, delayTime, delayFeedback, delayFilter);

      //alternatively use connect() to connect sound output of mySound
      mySound.connect(delayEffect);
```


---
## 06. P5.Sound + MIDI Input
https://editor.p5js.org/didny/sketches/UgBEU7SJT

> There is a bug in WebMIDI that prevents the MIDI port from closing properly, so after making changes to the sketch, press the reset button on the Arduino and wait until the Arduino restarts & connects back.

In the Chrome browser on Windows and Android, 
[Web MIDI API](https://webaudio.github.io/web-midi-api/) can be used to receive MIDI signals from external MIDI devices. This allows the sound of p5.sound to be controller from the sensor data from Arduino via USB-MIDI.


1. request MIDIAccess and regiseter callback events for midi messages.
```js

  // check for Web MIDI support of the browser
  if (navigator.requestMIDIAccess) console.log('This browser supports WebMIDI!')
  else console.log('WebMIDI is not supported in this browser.')

  // ask for MIDI access
  navigator.requestMIDIAccess()
    .then(onMIDISuccess, onMIDIFailure);


//callback function for MIDI access
function onMIDISuccess(midiAccess) {
  // console.log("midiAccess",midiAccess);
  const midi = midiAccess;
  const inputs = midi.inputs.values();
  
 //enable all the midi inputs and register onMIDIMessage callback event 
 for (var input of midiAccess.inputs.values()){ 
    console.log("midi input :",input.name)

   input.onmidimessage = onMIDIMessage;
 }
 
}
```

2. Separate MIDI message array consisting of command, note and velocity, and
Extracting events such as noteOn by MIDI command and dispatching event callbacks.

```js
//callback function for incoming MIDI messages 
function onMIDIMessage(message) {
  const data = message.data // [command/channel, note, velocity]

  const cmd = data[0] >> 4
  channel = data[0] & 0xf
  type = data[0] & 0xf0
  note = data[1]
  velocity = data[2]

  //select MIDI message type and call event function.
  // List of MIDI status message codes
  //http://www.opensound.com/pguide/midi/midi5.html
  switch (type) {
    case 144: // noteOn message type (always 144 no matter what channel)
      noteOn(channel, note, velocity)
      break
    case 128: //noteOff message type (always 128)
      noteOff(channel, note, velocity)
      break
  }
}
```  

3. The NoteOn event callback processes the received note number and generates a sound.

```js 
function noteOn(channel, note, velocity) {
  // if (channel !== 4) return
  let pitch = midiToFreq(note) / 440.0;
  mySound.rate(pitch)
  mySound.play();

}

function noteOff(channel, note, velocity) {
// //  background(0, 0, 255)
//   fill(255)
//   textSize(72)
//   text(note, 200, 200)
}


function onMIDIFailure(e) {
  console.log('Could not access your MIDI devices: ', e)
}
```
---
## Challenge 

> ### Combine the MIDI input sample with the sound file playback example above to create your own sound instruments!

---

## Next Step

In this tutorial, we have done a simple sound file playback in a browser triggered by a USB-MIDI input.

This next step could be for example, 
  - Explore more p5.sound capabilities such as 
    SoundSynthesis, SoundAnalysis 
    https://p5js.org/reference/#/libraries/p5.sound
　- Send multiple sensor input values from the Arduino.
　- More complex sound synthesis using p5.Synth
    - Interaction with smartphone's internal sensors (accelerometer, etc.)
       https://github.com/OhJia/p5MobileWebExamples
    - Use Tone.js library for more precise timing control and advanced sound synthesis
　　https://pdm.lsupathways.org/3_audio/


## References

Synthesizing and analyzing sound with p5
https://creative-coding.decontextualize.com/synthesizing-analyzing-sound/

p5.sound
https://p5js.org/reference/#/libraries/p5.sound

frequency to note
https://pages.mtu.edu/~suits/notefreqs.html

MIDI Tutorial for Programmers
https://www.cs.cmu.edu/~music/cmsip/readings/MIDI%20tutorial%20for%20programmers.html

Arduino-> serial to midi
https://projectgus.github.io/hairless-midiserial/

Tone.js Tutorial
https://pdm.lsupathways.org/3_audio/