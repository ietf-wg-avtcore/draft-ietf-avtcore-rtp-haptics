#  RTP Payload for Haptics

This repository is used for discussion of a personal draft for RTP Payload for Haptics (draft-hsyang-avtcore-rtp-haptics). 

* Current draft in [IETF Datatracker](https://datatracker.ietf.org/doc/draft-hsyang-avtcore-rtp-haptics/).
* [Current version draft in Github] (https://github.com/7439henry/draft-hsyang-avtcore-rtp-haptics/blob/main/Draft/draft-hsyang-avtcore-rtp-haptics-00.txt)

## Copy of abstract
This memo describes an RTP payload format for the MPEG-I haptic data. A haptic media stream is composed of MIHS units including a MIHS unit header and zero or more MIHS packets.  The RTP payload header format allows for packetization of a MIHS unit in an RTP packet payload as well as fragmentation of a MIHS unit into multiple RTP packets.

## Building the draft

The draft can be built into .txt or .xml files using [kramdown-rfc](https://github.com/cabo/kramdown-rfc). Please see details about kramdown-rfc usage from the related github project. 

Alternatively, [IETF conversion tool](https://author-tools.ietf.org/) can be used without installing additional software packages.

## Contributing

All material in this repository is considered as contributions to the IETF Standards Process and the applicaple policies shall apply. See guidelines for [contributions](https://github.com/7439henry/draft-hsyang-avtcore-rtp-haptics/blob/main/CONTRIBUTING.md)
