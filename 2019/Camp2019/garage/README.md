## Comment by the author (explo1t)
This challenge was the only Hardware On-Site Challenge and probably a last one for me on the Camp, because it was a mess to bring it to a stable level. Not only, was the dust a problem for the whole setup, but also the heat in the tent, constantly influenced the stability of the LimeSDR.

Some funny fuckups:
* Garages were 3D printed and when i set up the tent, the challenge was standing outside for like 15 minutes. During this time, the printed objects began to melt and 1 gate completely broke off...
* I printed the QR code, which were inside the garages a few days before. They were at the maximum size of the garage, but after they melted, they did not fit any more. Luckily QRs have some error corection, so i just cut some stuff away and most of the time it was still readable.
* As the challenge had a livestream to see the garages open up, i did not think about the problem of darkness in the night, as we did not have any light in the tent... Luckyli out neighbours got a spare one and so we could at least light up the garages at night.

# Garage
A pretty standard, but definitely not easy SDR challenge. The Goal was to decode the Garage Challenge/Response Signal and open the second Garage with the Flag.

## Description
In order to safeguard their flags, the ALLES team has brought their secure
storage garages to the camp. They can be opened remotely with transmitters,
using military-grade encryption™. We heard about security issues with similar
products, but according to the manufacturer, their garages are secure! Phew.

Only a small number of highly trusted team members carry the transmitters.
Unfortunately, one of them got drunk on Tschunk and a transmitter for
one of the garages ended up in enemy hands.

You got hold of the remote control and can press the button:
http://hax.allesctf.net:8080

Can you raid the second flag vault?

The following parameters might help you:

- Symbol Duration: 512u
- Sample Rate: 200k
- Frequency: 433.920 MHz

The signal is transmitted near Dragon Sleep Pwn Sector (each signal 10 times, bc transmission errors). You can either
connect to our remote receiver, or receive the signal locally using
the SDR of your choice. For transmitting your solution, please use our
submission queue which will allocate a time slot, transmit your signal
and provide you with a video feed of the garage.

You cannot physically access the garages.

Remote transceiver: [garage-e9cfbc3da45f4dd32c3ce3e98e141422830a8460d10a44e8388be53e27e13e41.zip](https://raw.githubusercontent.com/gcm-explo1t/Writeups/2019/Camp2019/garage/challenge/master/garage-e9cfbc3da45f4dd32c3ce3e98e141422830a8460d10a44e8388be53e27e13e41.zip)

## Solution
First of all, it was key to start the delivered client and capture some signals with it. Next it was key, to find a tool which could represent the captured data. One of these tools are [Universal Radio Hacker](https://github.com/jopohl/urh) or [GNUradio](https://www.gnuradio.org/) for example. When the client listened more than 30 seconds, there should at least be 2 signals visible:

BILD-SIGNALS

Additionally, there was a Website, with a camera livestream of one of the Garages:

BILD-GARAGE

With a click on the button it was possible to open it and see a QR code:

BILD-OffeneGarage

If the QR code is decoded it said: 1n 0rd3r 2 5ee 4 flag 7ry the other 6arage
Which firstly hinted the existence of another garage.

The mechanism behind the button, was a click on the remote, which send the correct solution for one of the garages.

So now if somebody listened via the limeSDR and the button was pressed, there were now 3 signals visible:

BILD-SignalsSend

The logical solution to this is, that there have to be 2 garages, where each of them sends a challenge and one remote, where it is possible for the user to send a correct response for garage 1.

Now it was key to understand the protocol and the Challenge-Response these garages use. Extracting a single challenge-signal for each garage from the recorded data it looked like this:

BILD-ChallengeSend1

BILD-ChallengeSend2

So with some more messages of these types it was possible to identify following message structure:

2Bit Garage ID (00 or 01)
5Bit Rolling Code (minutes%30)
101010 Static
11Bit Random (static until challenge solved)

Now the main task was, to analyze the succesfull response from garage 1. An extracted signal looked like this:

BILD-RESP1

In direct comparison some data seemed the same, but there are also some differences. BUT, when we look back at the QR in garage one, it was some odd leet speak text...
When you now look at only the digits of the text, you may recognize sth. at least, when you google "10325476" the first links directly reference to some MD5 implementations, as this number is one of the used constants.

Thus, with this hint, it was known, that MD5 is used somewhere. With a bit try and error and some additional recordings of the full challenge-response messages, it is possible to identify, that the response protocol looked like this:

2Bit Garage ID
5Bit Rolling Code (minutes%30)
0    Static (Identifier for a Response)
16Bit `md5(challenge_as_interger)->last 16 Bit`

Now the full protocol is know and the last step was to create a signal with the correct response for the garage 0. I used GNUradio for this:

BILD-GNUradio

When sending the created signal to the correct time, the second garage opened and the flag was visible.

Flag: `ALLES{SDR_h4xx_1s_b3st_h4xx}`
