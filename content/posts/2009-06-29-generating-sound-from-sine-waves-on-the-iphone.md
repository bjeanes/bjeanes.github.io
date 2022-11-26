+++
aliases = ["2009/generating-sound-from-sine-waves-on-the-iphone", "2009/06/generating-sound-from-sine-waves-on-the-iphone"]
date = 2009-06-29T11:31:00Z
slug = "generating-sound-from-sine-waves-on-the-iphone"
title = "Generating Sound from Sine Waves on the iPhone"
updated = 2011-08-28T21:24:26Z
[taxonomies]
tags = ["obj-c"]
+++

A few months ago I saw this extremely addictive flash game/toy called
[ToneMatrix](http://lab.andre-michelle.com/tonematrix).

My immediate thought was: "This would make quite possibly the coolest iPhone application ever."

So my good friend, [Anthony Mittaz](http://twitter.com/sync1983), and I
decided to have a crack at it but hit one immediate hurdle; however
proficient an iPhone developer Anthony is, he hadn’t done anything to do
with sound, and nor had I, in *any* language/framework, let alone on the
iPhone. Let me tell you this: Apple’s documentation in this area is
severely lacking. Tangent: well, almost all Apple documentation on
specialised things is pretty sparse.

I think we spent about two weeks’ worth of nights reading documentation,
source code (Cocoa, iPhone, Java, C, and others), and plain outright
stabbing in the dark code-wise getting not much more than ugly pops and
crackles. Whenever we thought we had a clean note working, we’d change
the frequency to what **should** be another nice clean note, and would
instead get ear-shattering screeches.

Nonetheless we persevered and were even able to come up with a nifty
little piece of code that could take an array of frequencies and play
them together in what could be made to sound like a pleasant chord.

Here is the relevant code:

## Code

**TonePlayer.h**

``` objective-c
#import <Foundation/Foundation.h>
#include <AudioUnit/AudioUnit.h>
#include <math.h>

#define kTonesAvailable 16
#define kOutputBus 0
#define kInputBus 1
#define kNumChannels 2

@interface TonePlayer : NSObject {
  AudioComponentInstance audioUnit;
  AudioStreamBasicDescription audioFormat;
  float phase[kTonesAvailable];
  float frequencies[kTonesAvailable];
}

-(OSStatus)start;
-(OSStatus)stop;
-(void)resetFrequencies;
-(void)cleanUp;
-(void)intialiseAudio;
@end

// Some C function headers to get arount compile errors while I refactor to be more readable
float phaseOffsetFromFrequency(float);
OSStatus RenderCallback(void*, AudioUnitRenderActionFlags*, const AudioTimeStamp*, UInt32, UInt32, AudioBufferList*);
```

**TonePlayer.m**

``` objective-c
#import "TonePlayer.h"

typedef struct {
  @defs(TonePlayer);
} SinewaveDef;

@implementation TonePlayer

static float gSampleRate = 44100.;
static float notes[111]  = {
      8.1758, 8.6620, 9.1770, 9.7227, 10.3009, 10.9134, 11.5623, 12.2499,
      12.9783, 13.7500, 14.5676, 15.4339, 16.3516, 17.3239, 18.3540, 19.4454,
      20.6017, 21.8268, 23.1247, 24.4997, 25.9565, 27.5000, 29.1352, 30.8677,
      32.7032, 34.6478, 36.7081, 38.8909, 41.2034, 43.6535, 46.2493, 48.9994,
      51.9131, 55.0000, 58.2705, 61.7354, 65.4064, 69.2957, 73.4162, 77.7817,
      82.4069, 87.3071, 92.4986, 97.9989, 103.8262, 110.0000, 116.5409, 123.4708,
      130.8128, 138.5913, 146.8324, 155.5635, 164.8138, 174.6141, 184.9972,
      195.9977, 207.6523, 220.0000, 233.0819, 246.9417, 261.6256, 277.1826,
      293.6648, 311.1270, 329.6276, 349.2282, 369.9944, 391.9954, 415.3047,
      440.0000, 466.1638, 493.8833, 523.2511, 554.3653, 587.3295, 622.2540,
      659.2551, 698.4565, 739.9888, 783.9909, 830.6094, 880.0000, 932.3275,
      987.7666, 1046.5023, 1108.7305, 1174.6591, 1244.5079, 1318.5102, 1396.9129,
      1479.9777, 1567.9817, 1661.2188, 1760.0000, 1864.6550, 1975.5332, 2093.0045,
      2217.4610, 2349.3181, 2489.0159, 2637.0205, 2793.8259, 2959.9554, 3135.9635,
      3322.4376, 3520.0000, 3729.3101, 3951.0664, 4186.0090, 4434.9221, 4698.6363
  };

OSStatus    RenderCallback(void                          *inRefCon,
                           AudioUnitRenderActionFlags    *ioActionFlags,
                           const AudioTimeStamp          *inTimeStamp,
                           UInt32                        inBusNumber,
                           UInt32                        inNumberFrames,
                           AudioBufferList               *ioData)
{
  SinewaveDef* def = inRefCon;

  float *outL = ioData->mBuffers[0].mData;
  float *outR = ioData->mBuffers[1].mData;

  float wave, j;
  float freqs[kTonesAvailable];

  memset(freqs, 0.0f, sizeof(freqs));

  for (int i=0; i< inNumberFrames; i++){
    wave = 0.0f;
    j = 0.0f;

    for(int m = 0; m < kTonesAvailable; ++m) {
      if(def->frequencies[m] != 0) {
        if(freqs[m] == 0) freqs[m] = phaseOffsetFromFrequency(def->frequencies[m]);

        wave += 0.5 * sinf(def->phase[m]);
        def->phase[m] += freqs[m];

        ++j;
      }
    }

    wave /= j;

    *outL++ = wave;
    *outR++ = wave;
  }

  return noErr;
}

float phaseOffsetFromFrequency(float frequency) {
  return (float)(frequency * 2.0 * M_PI / gSampleRate);
}

-(void)resetFrequencies{
  memset(frequencies, 0.0f, sizeof(frequencies));
  memset(phase, 0.0f, sizeof(phase));
}

-(OSStatus)start{
  [self resetFrequencies];
  OSStatus status = AudioOutputUnitStart(audioUnit);

  // See http://www.phys.unsw.edu.au/jw/notes.html for note numbers
  frequencies[0] = notes[60]; // C
  frequencies[1] = notes[64]; // E
  frequencies[2] = notes[67]; // G

  sleep(1);

  frequencies[0] = notes[64];
  frequencies[1] = notes[67];
  frequencies[2] = notes[71];

  sleep(1);

  frequencies[0] = notes[74];
  frequencies[1] = notes[67];
  frequencies[2] = notes[70];


  return status;
}

-(OSStatus)stop{
  return AudioOutputUnitStop(audioUnit);
}

-(void)cleanUp{
  AudioUnitUninitialize(audioUnit);
}

-(AudioStreamBasicDescription)audioFormat{
  return audioFormat;
}

// Below code is a cut down version (for output only) of the code written by
// Micheal "Code Fighter" Tyson (punch on Mike)
// See http://michael.tyson.id.au/2008/11/04/using-remoteio-audio-unit/ for details
-(void)intialiseAudio{
  OSStatus status;

  // Describe audio component
  AudioComponentDescription desc;
  desc.componentType = kAudioUnitType_Output;
  desc.componentSubType = kAudioUnitSubType_RemoteIO;
  desc.componentFlags = 0;
  desc.componentFlagsMask = 0;
  desc.componentManufacturer = kAudioUnitManufacturer_Apple;

  // Get component
  AudioComponent inputComponent = AudioComponentFindNext(NULL, &desc);

  // Get audio units
  status = AudioComponentInstanceNew(inputComponent, &audioUnit);

  UInt32 flag = 1;
  // Enable IO for playback
  status = AudioUnitSetProperty(audioUnit,
                  kAudioOutputUnitProperty_EnableIO,
                  kAudioUnitScope_Output,
                  kOutputBus,
                  &flag,
                  sizeof(flag));

  // Describe format
        audioFormat.mSampleRate = gSampleRate;
  audioFormat.mFormatID = kAudioFormatLinearPCM;
  audioFormat.mFormatFlags = kAudioFormatFlagsNativeFloatPacked | kAudioFormatFlagIsNonInterleaved;
  audioFormat.mFramesPerPacket = 1;
  audioFormat.mBytesPerPacket = sizeof(Float32);
  audioFormat.mBytesPerFrame = sizeof(Float32);
  audioFormat.mChannelsPerFrame = kNumChannels;
  audioFormat.mBitsPerChannel = 32;

  //Apply format
  status = AudioUnitSetProperty(audioUnit,
                  kAudioUnitProperty_StreamFormat,
                  kAudioUnitScope_Input,
                  kOutputBus,
                  &audioFormat,
                  sizeof(audioFormat));

  // Set up the playback  callback
  AURenderCallbackStruct callbackStruct;
  callbackStruct.inputProc = RenderCallback;
  //set the reference to "self" this becomes *inRefCon in the playback callback
  callbackStruct.inputProcRefCon = self;

  status = AudioUnitSetProperty(audioUnit,
                  kAudioUnitProperty_SetRenderCallback,
                  kAudioUnitScope_Global,
                  kOutputBus,
                  &callbackStruct,
                  sizeof(callbackStruct));

  // Initialise
  status = AudioUnitInitialize(audioUnit);
}

@end
```

I really wish that I could remember how most of it worked, or, more
importantly, the multitude of projects whose code we had to look at to
get it working. If you recognise any of the code or can help add
references on how people can build up a nice list of references for
people to use if are following a similar pursuit.

If you are wondering what ever happened to the application we were
building, it seems we weren’t the only ones who recognised the potential
of ToneMatrix as an iPhone application. During the 2-3 weeks of us
developing the sound generation code, about 5-6 ToneMatrix clones were
added to the AppStore. While most were crap, there was one that we felt
left little to be improved upon and we therefore recommend everyone goes
and downloads
[Melodica](http://itunes.apple.com/WebObjects/MZStore.woa/wa/viewSoftware?id=313573137&mt=8).
It’s just as addictive as the Flash version, only portable!
