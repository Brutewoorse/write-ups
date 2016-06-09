### MISC(250) sound.apk

We are presented with an apk-file and the hint that we have to find some morse code.

Installing it on and Android-Emulator gives us a simple Activity with an "Play" Button which plays a song which sounds a little off. Listening closely shows us that the audio-volume gets modulated over time. 

Opening the apk in jadx-gui shows us an play.mp3 in the Ressources/res/raw
Listening to that mp3 reveals that it is the song which plays in the app, but not volume modulated. 
So the modulation happens in the app.

The MainActivity points us to a make-threads which loads a whole lot of threads which all look very similar. 
They all change the volume of the AudioChannel 3 and wait for a certain amount of time.

```java
package org.teamsik.apps.hackingchallenge.sound.threads;

import android.app.Activity;
import android.media.AudioManager;

public class X00a6915f2bd395a55fb85aed647039e4136e35cb extends Thread {
    Activity f4a;
    long sleepTillTime = 17000;
    double style = 1.0d;
    int timetoSleep = 250;

    public X00a6915f2bd395a55fb85aed647039e4136e35cb(Activity a) {
        this.f4a = a;
    }

    public void run() {
        AudioManager audioManager = (AudioManager) this.f4a.getSystemService("audio");
        try {
            Thread.sleep(this.sleepTillTime);
            int sb2value = audioManager.getStreamMaxVolume(3);
            audioManager.setStreamVolume(3, sb2value / 2, 0);
            Thread.sleep((long) this.timetoSleep);
            audioManager.setStreamVolume(3, (int) (((double) sb2value) * this.style), 0);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

So i wrote a simple app which checks the volume of the audiostream 3 and every time it changes it prints in the log.

```java
import android.util.Log;
import android.media.AudioManager;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        AudioManager audioManager = (AudioManager) getSystemService(AUDIO_SERVICE);
        int now = 3, past = 3;
        while(true) {
            now = audioManager.getStreamVolume(3);
            if (now != past) {
                android.util.Log.d("vol", String.valueOf(now));
            }
            past = now;
        }
    }
}
```

The Output looks like this
```
06-02 14:03:27.208 1408-1408/com.[...].audiologger D/vol: 0
06-02 14:03:27.448 1408-1408/com.[...].audiologger D/vol: 7
06-02 14:03:27.700 1408-1408/com.[...].audiologger D/vol: 15
06-02 14:03:27.948 1408-1408/com.[...].audiologger D/vol: 7
06-02 14:03:28.700 1408-1408/com.[...].audiologger D/vol: 0
06-02 14:03:28.948 1408-1408/com.[...].audiologger D/vol: 7
06-02 14:03:29.200 1408-1408/com.[...].audiologger D/vol: 0
06-02 14:03:29.448 1408-1408/com.[...].audiologger D/vol: 7
06-02 14:03:29.700 1408-1408/com.[...].audiologger D/vol: 0
06-02 14:03:29.948 1408-1408/com.[...].audiologger D/vol: 7
06-02 14:03:30.200 1408-1408/com.[...].audiologger D/vol: 0
06-02 14:03:30.448 1408-1408/com.[...].audiologger D/vol: 7
06-02 14:03:31.200 1408-1408/com.[...].audiologger D/vol: 0
06-02 14:03:31.448 1408-1408/com.[...].audiologger D/vol: 7
06-02 14:03:32.200 1408-1408/com.[...].audiologger D/vol: 0
06-02 14:03:32.452 1408-1408/com.[...].audiologger D/vol: 7
06-02 14:03:32.700 1408-1408/com.[...].audiologger D/vol: 15
06-02 14:03:32.948 1408-1408/com.[...].audiologger D/vol: 7
06-02 14:03:33.200 1408-1408/com.[...].audiologger D/vol: 15
06-02 14:03:33.448 1408-1408/com.[...].audiologger D/vol: 7
06-02 14:03:33.700 1408-1408/com.[...].audiologger D/vol: 15
06-02 14:03:33.952 1408-1408/com.[...].audiologger D/vol: 7
06-02 14:03:34.200 1408-1408/com.[...].audiologger D/vol: 15
...
```

Analyzing the values shows us that we get 0, 7 and 15 as AudioVolumes. Every second value is 7 so we can assume that it is a padding-value.
Removing them leaves us with the 0 and 15 values as short and long.
Looking at the timestamps of the logs shows us 2 different time deltas. Since morse shows a End-Of-Character by waiting for a longer time we can then assume that 1s between 2 values means a new letter.

```
06-02 14:03:27.208 1408-1408/com.[...].audiologger D/vol: 0
06-02 14:03:27.700 1408-1408/com.[...].audiologger D/vol: 15	// A

06-02 14:03:28.700 1408-1408/com.[...].audiologger D/vol: 0
06-02 14:03:29.200 1408-1408/com.[...].audiologger D/vol: 0
06-02 14:03:29.700 1408-1408/com.[...].audiologger D/vol: 0
06-02 14:03:30.200 1408-1408/com.[...].audiologger D/vol: 0	// H

06-02 14:03:31.200 1408-1408/com.[...].audiologger D/vol: 0	// E

06-02 14:03:32.200 1408-1408/com.[...].audiologger D/vol: 0
06-02 14:03:32.700 1408-1408/com.[...].audiologger D/vol: 15
06-02 14:03:33.200 1408-1408/com.[...].audiologger D/vol: 15
06-02 14:03:33.700 1408-1408/com.[...].audiologger D/vol: 15
06-02 14:03:34.200 1408-1408/com.[...].audiologger D/vol: 15	// 1
...
```

Going through the values gives us the flag: AHE16-L157ENCL053LY2M3-