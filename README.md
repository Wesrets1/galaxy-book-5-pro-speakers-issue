# galaxy-book-5-pro-speakers-issue
I installed Ubuntu on my galaxy book 5 pro, but the speakers don't work. 

Title: Lunar Lake (Galaxy Book5) — ALC298: speakers exposed but amp/pin not enabled (SOF topology/UCM missing)

Description
The internal speakers on a Samsung Galaxy Book5 are present and exposed by the SOF stack, but remain silent because the codec amp/pin is not enabled. ALSA controls show speaker switches on and volumes at max, yet Pin Widget reads return 0x0 and hda-verb writes have no effect. This strongly suggests the DSP topology or vendor init sequence required to power/route the smart amp is missing from SOF for this platform.

Hardware / OS

Model: Samsung Galaxy Book5 Pro(platform)

Codec: Realtek ALC298 (VendorId 0x10ec0298)

Subsystem Id: 0x144d:c1dd

Kernel: 6.18.1-061801-generic

Distro: Ubuntu 25.10 (Questing Quokka)

SOF driver: sof-hda-dsp (snd_sof_pci_intel_lnl)

Platform module: samsung_galaxybook (loaded; handles platform quirks but not audio topology)

Observed behavior

aplay -l shows card 0: sofhdadsp [sof-hda-dsp] with HDA Analog and HDMI outputs.

PipeWire/ALSA expose a Speaker sink, but it stays SUSPENDED (no RUNNING stream).

ALSA mixer controls show speaker/headphone switches on and volumes at maximum.

cat /proc/asound/card0/codec#0 shows the ALC298 codec and a fixed internal speaker pin (node 0x17) whose Amp-Out vals are [0x00 0x00].

hda-verb reads for pin widgets return value = 0x0 and hda-verb writes (examples tried: 0x71 0x40, 0x71 0x41) are ignored (no change).

Vendor widget reads returned zeros as well.

Key excerpts (paste of relevant outputs):

Codec: Realtek ALC298
Vendor Id: 0x10ec0298
Subsystem Id: 0x144dc1dd

Node 0x17 [Pin Complex] ... Pin Default: [Fixed] Speaker at Int N/A
  Amp-Out vals: [0x00 0x00]

amixer -c 0 contents shows:
  Speaker Playback Switch : on
  Speaker Playback Volume : 127

hda-verb reads:
  sudo hda-verb /dev/snd/hwC0D0 0x17 0x70 0x00  => value = 0x0
  sudo hda-verb /dev/snd/hwC0D0 0x14 0x70 0x00  => value = 0x0
  sudo hda-verb /dev/snd/hwC0D0 0x03 0x70 0x00  => value = 0x0

hda-verb writes attempted:
  sudo hda-verb /dev/snd/hwC0D0 0x17 0x71 0x40  => no effect (readback still 0x0)
Why this looks like a SOF/topology issue

The codec and controls are present and unmuted, but the amp outputs are zero and HDA verbs are ignored. On Lunar Lake platforms the amp is often enabled by the DSP topology or by a vendor init sequence executed by the SOF driver. If the topology or UCM mapping is missing, the DSP will not route audio to the amp or power the smart amp.

Requested action

Please advise or add the correct SOF topology and/or vendor init quirk for Realtek ALC298 on Intel Lunar Lake (Samsung Galaxy Book5, subsystem 144d:c1dd). If maintainers need additional logs or tests, I can provide them or run targeted hda-verb reads/writes under guidance.

Attachments / files I can provide

Full codec dump: ~/codec-dump-card0.txt (cat /proc/asound/card0/codec#0)

amixer -c 0 contents output: ~/amixer-contents-1.txt

dmesg SOF logs: ~/sof-dmesg.txt (captured with sudo dmesg | grep -iE 'sof|tplg|hda|codec|galaxybook')

aplay -l and pactl list short sinks outputs

modinfo samsung_galaxybook and kernel version

If you want me to attach these files to the issue, tell me which ones and I will paste them into the issue body or attach them as text files.

alsa-ucm-conf issue (ready to paste)
Title: ALC298 (Galaxy Book5) — UCM/topology mapping missing; speakers present but amp not enabled

Description
On a Samsung Galaxy Book5 with Intel Lunar Lake and Realtek ALC298, ALSA/ASoC expose the sof-hda-dsp card and speaker controls, but the internal speaker amp remains off (Pin Widget amp values are 0x00) and hda-verb writes do not enable it. This indicates a missing UCM/topology mapping or vendor init sequence for ALC298 on this platform.

System summary

Distro: Ubuntu 25.10 (Questing Quokka)

Kernel: 6.18.1-061801-generic

Card: sof-hda-dsp (SOF + HDA bridge)

Codec: Realtek ALC298 (Vendor 0x10ec0298)

Subsystem: 144d:c1dd

Symptoms

amixer shows Speaker Playback Switch = on and Speaker Playback Volume = 127.

cat /proc/asound/card0/codec#0 shows node 0x17 (internal speaker) with Amp-Out vals: [0x00 0x00].

hda-verb reads/writes to pin widgets return zeros / have no effect.

Playback sinks exist but remain suspended or produce no audible sound from internal speakers.

Request

Please add or advise the correct UCM profile and/or topology mapping for ALC298 on Lunar Lake (Samsung Galaxy Book5). If a UCM profile or topology snippet is required for testing, I can run it and report results.

Files available

Full codec dump (/proc/asound/card0/codec#0)

amixer -c 0 contents

aplay -l and pactl outputs

SOF dmesg logs

How to reproduce on my machine (for maintainers)
Boot Ubuntu 25.10 with kernel 6.18.1-061801-generic.

Confirm aplay -l shows card 0: sofhdadsp [sof-hda-dsp].

cat /proc/asound/card0/codec#0 shows ALC298 and node 0x17 (internal speaker) with Amp-Out [0x00 0x00].

Attempting playback (PipeWire or speaker-test) produces no audible sound from internal speakers.

