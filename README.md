# ladspa_dsp setup for system wide dsp on linux
## connected as the tape monitor loop on an AV amp to provide room correction

## tested on debian bookworm

```

su -
apt update
apt upgrade
apt install git make pkg-config libfftw3-dev libzita-convolver-dev libsndfile1-dev libavcodec59  libavformat59 libavutil57 libao-dev libmad0-dev ladspa-sdk libtool libalsaplayer-dev

git clone https://github.com/bmc0/dsp
make
make install
```

### create /etc/asound.conf with contents

```
defaults.pcm.card 1
defaults.ctl.card 1
```

### create .asoundrc in home directory with contents

```
pcm.dsp {
        type plug
        slave {
                format FLOAT
                rate unchanged
                pcm {
                        type ladspa
                        channels 2
                        path "/usr/lib/ladspa"
                        playback_plugins [{
                                label "ladspa_dsp"
                        }]
                        slave.pcm {
                                type plug
                                slave {
                                        pcm "front:1"
                                        rate 48000
                                        channels unchanged
                                }
                        }
                }
        }
}

pcm.!default {
        type copy
        slave.pcm "dsp"
}
```

### create /etc/ladspa_dsp with the contents
```
input_channels=2
output_channels=2
LC_NUMERIC=C
effects_chain=:0 eq 22.80 11.626 -9.60 eq 26.35 9.523 -10.00 eq 35.15 7.546 -5.30 eq 65.20 23.411 -0.90 eq 82.00 10.475 -4.80 eq 152.5 4.784 -6.00 eq 241.0 5.000 -5.20 eq 342.0 5.000 -4.70 eq 615.0 5.000 -2.00 eq 828.0 3.805 -2.50 eq 1289 2.570 -4.10 eq 1669 3.085 -4.60 eq 6760 1.042 -2.30 eq 44.15 22.087 -1.20 eq 119.0 15.859 -3.10 eq 619.0 5.000 -0.60 eq 8739 4.012 -1.50 :1 eq 22.35 11.018 -6.40 eq 25.05 12.291 -6.90 eq 27.75 9.857 -7.80 eq 33.75 16.897 -4.20 eq 36.40 9.701 -3.30 eq 68.20 9.377 -5.30 eq 75.40 10.488 -3.90 eq 111.0 7.006 -6.20 eq 145.0 7.227 -5.60 eq 170.0 6.973 -2.60 eq 235.0 5.000 -5.80 eq 326.0 5.000 -9.60 eq 435.0 5.000 -2.20 eq 723.0 5.000 -3.80 eq 1036 3.078 -6.50 eq 4048 4.999 -2.50 eq 6568 1.539 -2.60 eq 10021 1.091 -3.50 eq 30.05 14.904 -1.80 eq 35.00 17.434 -1.30 eq 1273 5.000 -1.20 delay 1m

```

### create start_audio.sh with the contents
```
while true; do
  arecord -D hw:1 -f dat | aplay
done
```

In alsamixer select line in as the capture input to sounds card 1.  Make sure sound card 2 is the default
This will record from line input of usb card 1 and pipe to sounds card 2.  ladspa_dsp will filter the audio as it is played


