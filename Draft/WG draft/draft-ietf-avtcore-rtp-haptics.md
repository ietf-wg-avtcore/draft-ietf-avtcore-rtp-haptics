---
title: "RTP Payload for Haptics"
abbrev: RTP-Payload-Haptic
docname: draft-ietf-avtcore-rtp-haptics-01
date: {DATE}
stream: IETF
category: std
ipr: trust200902
area: Transport
workgroup: avtcore

stand_alone: yes
pi: [toc, sortrefs, symrefs, docmapping]

author:
  -
    ins: HS. Yang
    name: Hyunsik Yang
    org: InterDigital
    email: hyunsik.yang@interdigital.com
    country: USA
  -
    ins: X. de Foy
    name: Xavier de Foy
    org: InterDigital
    email: xavier.defoy@interdigital.com
    country: Canada
	
normative:
  ISO.IEC.23090-31:
    title: "Text of ISO/IEC FDIS 23090-31 MPEG Haptics Coding"
    author:
      org: "ISO/IEC"
    date: 2024
    seriesinfo:
      ISO/IEC: 23090-31
    target: https://isotc.iso.org/livelink/livelink/open/jtc1sc29wg7

informative:

  RFC2119:
  RFC3550:
  RFC2736:
  RFC8088:
  RFC8866:
  RFC3264:
  RFC8174:
  RFC7201:
  RFC7202:
  RFC5124:
  RFC3711:
  RFC3551:
  RFC4585:
  RFC8866:
  
  I-D.ietf-mediaman-haptics:

--- abstract

This memo describes an RTP payload format for the MPEG-I haptic data. A haptic media stream is composed of MIHS units including a MIHS(MPEG-I Haptic Stream)  unit header and zero or more MIHS packets. The RTP payload header format allows for packetization of a MIHS unit in an RTP packet payload as well as fragmentation of a MIHS unit into multiple RTP packets.

--- middle

# Introduction

Haptics provides users with tactile effects in addition to audio and video, allowing them to experience sensory immersion. Haptic data is mainly transmitted to devices that act as actuators and provides them with information to operate according to the values defined in haptic effects. The IETF is registering haptics as a primary media type akin to audio and video {{I-D.ietf-mediaman-haptics}}.

The MPEG Haptics Coding standard {{ISO.IEC.23090-31}} defines the data formats, metadata, and codec architecture to encode, decode, synthesize and transmit haptic signals. It defines the "MIHS unit" as a unit of packetization suitable for streaming, and similar in essence to the NAL unit defined in some video specifications. This document describes how haptic data (MIHS units) can be transmitted using the RTP protocol. This document followed recommendations in {{RFC8088}} and {{RFC2736}} for RTP payload format writers.

# Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, as shown here. 


# Definition

This document uses the definitions of the MPEG Haptics Coding standard {{ISO.IEC.23090-31}}. Some of these terms are provided here for convenience.

Actuator: component of a device for rendering haptic sensations.

Avatar: body (or part of body) representation.

Band: component in a channel for containing effects for a specific range of frequencies.

Channel: component in a perception containing one or more bands rendered on a device at a specific body location.

Device: physical system having one or more actuators configured to render a haptic sensation corresponding with a given signal.

Effect: component of a band for defining a signal, consisting of a haptic waveform or one or more haptic keyframes.

Experience: top level haptic component containing perceptions and metadata.

Haptics: tactile sensations.

Keyframe: component of an effect mapping a position in time or space to an effect parameter such as amplitude or frequency.

Metadata: global information about an experience, perception, channel, or band.

MIHS unit: unit of packetization of the MPEG-I Haptic Stream format, which is used as unit of payload in the format described in this memo. See {{haptic-format-dsecription}} for details.

Modality: type of haptics, such as vibration, force, pressure, position, velocity, or temperature.

Perception: haptic perception containing channels of a specific modality.

Signal: representation of the haptics associated with a specific modality to be rendered on a device.

Hmpg format: hmpg is a binary compressed format for haptics data. Information is stored in a binary form and data compression is applied on data at the band level. The haptics/hmpg media subtype is registered in {{I-D.ietf-mediaman-haptics}} and updated by this memo. 

Independent unit: a MIHS unit is independent if it can be decoded independently from earlier units. Independent units contain timing information and are also called "sync units" in {{ISO.IEC.23090-31}}. 

Dependent unit: a MIHS unit is dependent if it requires earlier units for decoding. Dependent units do not contain timing information and are also called "non-sync units" in {{ISO.IEC.23090-31}}. 

Time-independent effect: a haptic effect that occurs regardless of time. The tactile feedback of a texture is a representative example. Time-independent effects are encoded in spatial MIHS units, defined in {{MIHS-format}}.

Time-dependent effect: a haptic effect that varies over time. For example, tactile feedback for vibration and force  are time-dependent effects, and are encoded in temporal MIHS units, defined in {{MIHS-format}}.

# Haptic Format Description {#haptic-format-dsecription}

## Overview of Haptic Coding

The MPEG Haptics Coding standard specifies methods for efficient transmission and rendering of haptic signals, to enable immersive experiences. It supports multiple types of perceptions, including the most common vibrotactile (sense of touch that perceives vibrations) and kinesthetic perceptions (tactile resistance or force), but also other, less common perceptions, including for example the sense of temperature or texture. It also supports two approaches for encoding haptic signals: a "quantized" approach based on samples of measured data, and a "descriptive" approach where the signal is synthesized using a combination of functions. Both quantized and descriptive data can be encoded in a human-readable exchange format based on JSON (.hjif), or in a binary packetized format for distribution and streaming (.hmpg). This last format is referred to as the MPEG-I Haptic Stream (MIHS) format and is a base for the RTP payload format described in this document.

## MPEG-I Haptic Stream (MIHS) format {#MIHS-format}

MIHS is a stream format used to transport haptic data. Haptic data including haptic effects is packetized according to the MIHS format, and delivered to actuators, which operate according to the provided effects. The MIHS format has two level packetization, MIHS units and MIHS packets.

MIHS units are composed of a MIHS unit header and zero or more MIHS packets. Four types of MIHS units are defined. An initialization MIHS unit contains MIHS packets carrying metadata necessary to reset and initialize a haptic decoder, including a timestamp. A temporal MIHS unit contains one or more MIHS packets defining time-dependent effects and providing modalities such as pressure, velocity, and acceleration. The duration of a temporal unit is a positive number. A spatial MIHS unit contains one or more MIHS packets providing time-independent effects, such as vibrotactile texture, stiffness, and friction. The duration of a spatial unit is always zero.
A silent MIHS unit indicates that there is no effect during a time interval and its duration is a positive number.

A MIHS unit can be marked as independent or dependent. When a decoder processes an independent unit, it resets the previous effects and therefore provides a haptic experience independent from any previous MIHS unit.  A dependent unit is the continuation of previous MIHS units and cannot be independently decoded and rendered without having decoded previous MIHS unit(s). Initialization and spatial MIHS units are always independent units. Temporal and silent MIHS units can be dependent or independent units.

{{figure-stream}} illustrates a succession of MIHS units in a MIHS stream.

~~~~~~~~~~
+--------+ +-------+ +------------+ +-------------+ +-----------+
|Initial*| |Spatial| |  Temporal  | |Temporal Unit| |Silent Unit|
| Unit   |-| Unit  |-|Unit(indep.)|-| (dependent) |-| (indep.)  |
+--------+ +-------+ +------------+ +-------------+ +-----------+
*Initialization 
~~~~~~~~~~
{: #figure-stream title="Example of MIHS stream"}

# Payload format for haptics

## RTP header Usage

The RTP header is defined in {{RFC3550}} and represented in {{figure-rtpheader}}. Some of the header field values are interpreted as follows.

~~~~~~~~~~
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|V=2|P|X|  CC   |M|     PT      |       sequence number         |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                           timestamp                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           synchronization source (SSRC) identifier            |
+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+=+
|            contributing source (CSRC) identifiers             |
|                             ....                              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~~~~~~~
{: #figure-rtpheader title="RTP header for Haptic."}

Payload type (PT): 7 bits. The assignment of a payload type MUST be performed either through the profile used or in a dynamic way.

Time Stamp (TS): 32 bits. A timestamp representing the sampling time of the first sample of the MIHS unit in the RTP payload. The clock frequency MUST be set to the sample rate of the encoded haptic data and is conveyed out-of-band (e.g., as an SDP parameter).

Marker bit (M): 1 bit. The marker bit SHOULD be set to one in the first non-silent RTP packet after a period of haptic silence. This enables jitter buffer adaptation and haptics device washout (i.e., reset to a neutral position) prior to the beginning of the burst with minimal impact on the quality of experience for the end user. The marker bit in all other packets MUST be set to zero.



## Payload Header {#payload-header}

The RTP Payload Header follows the RTP header. {{figure-payloadheader}} describes RTP Payload Header.

~~~~~~~~~~
+---------------+
|0|1|2|3|4|5|6|7|
+-+-+-+-+-+-+-+-+
|D| UT  |   L   |
+-+-----+-------+
~~~~~~~~~~
{: #figure-payloadheader title="RTP payload header for Haptic."}

D (Dependency, 1 bit): this field is used to indicate whether the MIHS unit included in the RTP payload is, when its value is one, dependent or, when its value is zero, independent. 

UT (Unit Type, 3 bits): this field indicates the type of the MIHS unit included in the RTP payload. UT field values are listed in {{figure-transmission-type}}.

L (MIHS Layer, 4 bits): this field is an integer value which indicates the priority order of the MIHS unit included in the RTP payload, as determined by the haptic sender (e.g., by the haptic codec), based on application-specific needs. For example, the sender may use the MIHS layer to prioritize perceptions with the largest impact on the end-user experience. Zero corresponds to the highest priority. The semantic of individual MIHS layers is not specified and left for the application to assign. 


## Payload Structures

Three different types of RTP packet payload structures are specified. A single unit packet contains a single MIHS unit in the payload.  A fragmentation unit contains a subset of a MIHS unit. An aggregation packet contains multiple MIHS units in the payload. The unit type (UT) field of the RTP payload header {{figure-transmission-type}} identifies both the payload structure and, in the case of a single unit structure, also identifies the type of MIHS unit present in the payload.

~~~~~~~~~~
Unit     Payload   Name
Type     Structure
----------------------------------------
0        N/A       Reserved
1        Single    Initialization MIHS Unit
2        Single    Temporal MIHS Unit
3        Single    Spatial MIHS Unit
4        Single    Silent MIHS Unit
5        Aggr     Aggregation Packet(STAP)
6        Aggr     Aggregation Packet(MTAP)
7        Frag     Fragmentation Unit
~~~~~~~~~~
{: #figure-transmission-type title="Payload structure type for haptic"}

The payload structures are represented in {{figure-transmission-type}}.  The single unit payload structure is specified in {{single}}. The fragmented unit payload structure is specified in {{fragmented}}. The aggregation unit payload structure is specified in {{aggregated}}. 

~~~~~~~~~~
                                            +-------------------+
                                            |     RTP Header    |
                                            +-------------------+
                                            | RTP payload Header|
                      +-------------------+ |   (UT = Aggr)     |
                      |     RTP Header    | +-------------------+
+-------------------+ +-------------------+ |  MIHS unit 1 Size |
|     RTP Header    | | RTP payload Header| +-------------------+
+-------------------+ |   (UT = Frag)     | |    MIHS Unit 1    |
| RTP payload Header| +-------------------+ +-------------------+
+-------------------+ |     FU Header     | |  MIHS unit 2 Size |
|    RTP payload    | +-------------------+ +-------------------+
| (Single MIHS unit)| |    RTP Payload    | |    ...            |
+-------------------+ +-------------------+ +-------------------+
(a) single unit      (b)fragmentation unit (c) aggregation packet
~~~~~~~~~~
{: #figure-transmission-style  title="RTP Transmission mode"}

### Single Unit Payload Structure {#single}

In a single unit payload structure, as described in {{figure-transmission-style}}, the RTP packet contains the RTP header, followed by the payload header and one single MIHS unit. The payload header follows the structure described in {{payload-header}}. The  payload contains a MIHS unit as defined in {{ISO.IEC.23090-31}}.

~~~~~~~~~~
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                          RTP Header                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|payload Header |                                               |
+---------------+                                               |
|                        MIHS unit data                         |
|                               +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                               :...OPTIONAL RTP padding        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~~~~~~~
{: #figure-transmission-single  title="Single unit payload structure"}


### Fragmented Unit Payload Structure {#fragmented}

In a fragmented unit payload structure, as described in {{figure-fragment-structure}}, the RTP packet contains the RTP header, followed by the payload header, a Fragmented Unit (FU) header, and a MIHS unit fragment. The payload header follows the structure described in {{payload-header}}. The value of the UT field of the payload header is 7. The FU header follows the structure described in {{figure-fragment-header}}.

~~~~~~~~~~
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                          RTP Header                           |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Payload Header | FU Header     |                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+                               |
|                     MIHS Unit Fragment                        |
|                               +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                               :...OPTIONAL RTP padding        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~~~~~~~
{: #figure-fragment-structure title="Fragmentation unit header"}

FU headers are used to enable fragmenting a single MIHS unit into multiple RTP packets. Fragments of the same MIHS unit MUST be sent in consecutive order with ascending RTP sequence numbers (with no other RTP packets within the same RTP stream being sent between the first and last fragment). FUs MUST NOT be nested, i.e., an FU MUST NOT contain a subset of another FU.

{{figure-fragment-header}} describes a FU header, including the following fields:

~~~~~~~~~~
+-------------------------------+
| 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
+---+---+---+---+---+---+---+---+
|FUS|FUE|   RSV     |     UT    |
+---+---+-----------+-----------+
~~~~~~~~~~
{: #figure-fragment-header title="Fragmentation unit header"}

FUS (Fragmented Unit Start, 1 bit): this field MUST be set to 1 for the first fragment, and 0 for the other fragments.

FUE (Fragmented Unit End, 1 bit): this field MUST be set to 1 for the last fragment, and 0 for the other fragments.

RSV (Reserved, 3 bits): these bits MUST be set to 0 by the sender and ignored by the receiver.

UT (Unit Type, 3 bits): this field indicates the type of the MIHS unit this fragment belongs to, using values defined in {{figure-transmission-type}}.

The use of MIHS unit fragmentation in RTP means that a media receiver can receive some fragments, but not other fragments. The missing fragments will typically not be retransmitted by RTP. This results in partially received MIHS units, which can be either dropped or used by the decoding application, based on implementation.

### Aggregation Packet Payload Structure {#aggregated}

In an aggregation packet, as described in {{figure-aggre-structure}}, the RTP packet contains an RTP header, followed by a payload header, and, for each aggregated MIHS Unit, a MIHS unit size followed by the MIHS unit. The payload header follows the structure described in {{payload-header}}. 

~~~~~~~~~~
     0                   1                   2                   3
     0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                          RTP Header                           |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |        RTP Payload Header     |       MIHS Unit 1 Size        |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                           MIHS Unit 1                         |
    |                                                               |
    :                                                               :
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |        MIHS Unit 2 Size     |                                 |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+                                 |
    |                           MIHS Unit 2                         |
    |                                                               |
    |                               +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                               :...OPTIONAL RTP padding        |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
~~~~~~~~~~
{: #figure-aggre-structure title="Single-Time Aggregation Packet"}

{{figure-aggre-structure}} shows a Single-Time Aggregation Packet (STAP), which can be used to transmit multiple MIHS units that correspond to the same timestamp. For example, if two frequencies are used for the same content, they can be transmitted at once in a STAP. Multiple spatial units can also be sent together in a STAP, since this type of haptics data is time independent. The value of the UT field of the payload header is 5. 



~~~~~~~~~~
    0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                          RTP Header                           |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |        RTP Payload Header     |       MIHS Unit 1 Size        |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                           TS offset           |               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+               |
    |                           MIHS Unit 1                         |
    |                                                               |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |       MIHS Unit 2 Size        |            TS offset          |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |   TS offset   |                                               |
    |-+-+-+-+-+-+-+-+                                               |
    |                          MIHS Unit 2                          |
    |                               +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
    |                               :...OPTIONAL RTP padding        |
    +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+	
~~~~~~~~~~
{: #figure-aggremtap-structure title="Multiple-time aggregation packet"}

{{figure-aggremtap-structure}} shows a multi-time aggregation packet. It is used to transmit multiple MIHS units with different timestamps, in one RTP packet. Multi-time aggregation can help reduce the number of packets, in environments where some delay is acceptable. The value of the UT field of the payload header is 6. 

## MIHS Units Transmission Considerations {#mihs-trans}

The following considerations apply for the streaming of MIHS units over RTP:

The MIHS format enables variable duration units and uses initialization MIHS units to declare the duration of subsequent non-zero duration MIHS units, as well as the variation of this duration. A sender SHOULD set constant or low-variability (e.g., lower than the playout buffer) durations in initialization MIHS units, for RTP streaming. This enables the receiver to determine early (e.g., using a timer) when a unit has been lost and make the decoder more robust to RTP packet loss. If a sender sends MIHS units with high duration variations, the receiver may need to wait for a long period of time (e.g., the upper bound of the duration variation), to determine if a MIHS unit was lost in transmission. Whether this behavior is acceptable or not is application dependent.

The MIHS format uses silent MIHS units to signal haptic silence. A sender MAY decide not to send silent units, to save network resources. Since, from a receiver standpoint, a missed MIHS unit may originate from a not-sent silent unit, or a lost packet, a sender MAY send one, or a few, MIHS silent units at the beginning of a haptic silence. If a media receiver receives a MIHS silent unit, the receiver SHOULD assume that silence is intended until the reception of a non-silent MIHS unit. This can reduce the number of false detection of lost RTP packets by the decoder.

# Payload Format Parameters  

This section describes payload format parameters. {{format-param}} updates the 'haptics' Media Type Registration and {{optional-param}} specifies new optional parameters. {{sdp-registration}} further registers a new token in the media sub-registry of the Session Description Protocols (SDP) Parameters registry.

## Media Type Registration Update {#format-param}

This memo updates the 'hmpg' haptic subtype defined in section 4.3.3 of {{I-D.ietf-mediaman-haptics}} for use with the MPEG-I haptics streamable binary coding format described in ISO/IEC DIS 23090-31: Haptics coding {{ISO.IEC.23090-31}}. This memo especially defines optional parameters for this type in {{optional-param}}.  A mapping of the parameters into the Session Description Protocol (SDP) {{RFC8866}} is also provided for applications that use SDP. Equivalent parameters could be defined elsewhere for use with control protocols that do not use SDP.
The receiver MUST ignore any parameter unspecified in this memo.

The following entries identify the media type being updated:

Type name: haptics

Subtype name: hmpg

The following entries are replaced by this memo:

Optional parameters: see section 6.2 of RFC XXX (note to RFC editor: replace with this RFC's number).

Person & email address to contact for further information: Yeshwant Muthusamy (yeshwant@yeshvik.com) and Hyunsik Yang (hyunsik.yang@interdigital.com)

## Optional Parameters Definition  {#optional-param}

*hmpg-ver*
provides the year of the edition and amendment of ISO/IEC 23090-31 that this file conforms to, as defined in {{ISO.IEC.23090-31}}. MPEG_haptics object.version is a string which may hold values such as XXXX or XXXX-Y where XXXX is the year of publication and Y is the amendment number, if any. For the initial release of the specifications, the value is "2023".

*hmpg-profile*
indicates the profile used to generate the encoded stream as defined in {{ISO.IEC.23090-31}}: MPEG_haptics object.profile is a string which may in the initial release of the specifications hold the values "simple-parametric" or "main".

*hmpg-lvl*
indicates the level used to generate the encoded stream as defined in {{ISO.IEC.23090-31}}: MPEG_haptics object.level is an integer which may in the initial release of the specifications hold the value 1 or 2.

*hmpg-maxlod*
indicates the maximum level of details to use for the avatar(s). The avatar level of detail (LOD) is defined in {{ISO.IEC.23090-31}}: MPEG_haptics.avatar object.lod is an integer which may in the initial release of the specifications hold 0 or a positive integer.

*hmpg-avtypes* 
indicates, using a coma-separated list, types of haptic perception represented by the avatar(s). The avatar type is defined in {{ISO.IEC.23090-31}}: MPEG_haptics.avatar object.type is an integer which may in the initial release of the specifications hold values among "Vibration", "Pressure", "Temperature", "Custom".

*hmpg-modalities*
indicates, using a coma-separated list, haptic perception modalities (e.g., pressure, acceleration, velocity, position, temperature, etc.). The perception modality is defined in {{ISO.IEC.23090-31}}: MPEG_haptics.perception object.perception_modality is a string which may in the initial release of the specifications hold values among "Pressure", "Acceleration", "Velocity", "Position", "Temperature", "Vibrotactile", "Water", "Wind", "Force", "Electrotactile", "Vibrotactile Texture", "Stiffness", "Friction", "Humidity", "User-defined Temporal", "User-defined Spatial", "Other".

*hmpg-bodypartmask*
indicates, using a bitmask, the location of the devices or actuators on the body. The body part mask is defined in {{ISO.IEC.23090-31}}: MPEG_haptics.reference_device object.body_part_mask is a 32-bit integer which may in the initial release of the specifications hold a bit mask using bit positions defined in table 7 of [ISO.IEC.23090-31].

*hmpg-maxfreq*
indicates the maximum frequency of haptic data for vibrotactile perceptions (Hz). Maximum frequency is defined in {{ISO.IEC.23090-31}}: MPEG_haptics.reference_device object.maximum_frequency is defined as an integer or floating-point number in the initial release of the specifications.

*hmpg-minfreq*
indicates the minimum frequency of haptic data for vibrotactile perceptions (Hz). Minimum frequency is defined in {{ISO.IEC.23090-31}}: MPEG_haptics.reference_device object.minimum_frequency is defined as an integer or floating-point number in the initial release of the specifications.

*hmpg-dvctypes*
indicates, using a coma-separated list, the types of actuators. The device type is defined in {{ISO.IEC.23090-31}}: MPEG_haptics.reference_device object.type is a string which may in the initial release of the specifications hold values among "LRA", "VCA", "ERM", "Piezo" or "Unknown".

*hmpg-silencesupp*
indicates whether silence suppression should be used (1) or not (0). The default value shall be 1.

## SDP Parameter Registration {#sdp-registration}
 
This memo registers an 'haptics' token in the media sub-registry of the Session Description Protocols (SDP) Parameters registry. This registration contains the required information elements outlined in the SDP registration procedure defined in section 8.2 of {{RFC8866}}.
 
   (1)  Contact Information:
 
           Name: Hyunsik Yang
           Email: hyunsik.yang@interdigital.com
 
   (2)  Name being registered (as it will appear in SDP): haptics
 
   (3)  Long-form name in English: haptics
 
   (4)  Type of name ('media', 'proto', 'fmt', 'bwtype', 'nettype', or
        'addrtype'): media
 
   (5)  Purpose of the registered name:
 
           The 'haptics' media type for the Session Description Protocol
           is used to describe a media stream whose content can be 
           rendered as touch-related sensations. 
           The media subtype further describes the specific
           format of the haptics stream.  The 'haptics' media type for
           SDP is used to establish haptics media streams.
 
   (6)  Specification for the registered name: RFC XXXX
   
   RFC Editor Note: Replace RFC XXXX with the published RFC number.


# SDP Considerations

The mapping of above defined payload format media type to the corresponding fields in the Session Description Protocol (SDP) is done according to {{RFC8866}}.

The media name in the "m=" line of SDP MUST be haptics.

The encoding name in the "a=rtpmap" line of SDP MUST be hmpg

The clock rate in the "a=rtpmap" line may be any sampling rate, typically 8000.

The OPTIONAL parameters (defined in {{optional-param}}), when present, MUST be included in the "a=fmtp" line of SDP. This is expressed as a media type string, in the form of a semicolon-separated list of parameter=value pairs.

An example of media representation corresponding to the hmpg RTP payload in SDP is as follows:

    m=haptics 43291 UDP/TLS/RTP/SAVPF 115
    a=rtpmap:115 hmpg/8000
    a=fmtp:115 hmpg-profile=1;hmpg-lvl=1;hmpg-ver=2023


## SDP Offer/Answer Considerations

When using the offer/answer procedure described in {{RFC3264}} to negotiate the use of haptic, the following considerations apply:

The haptic signal can be sampled at different rates. The MPEG Haptics Coding standard does not mandate a specific frequency. A typical sample rate is 8000Hz.

The parameter 'hmpg-ver' indicates the version of the haptic standard specification. If it is not specified, the initial version of the MPEG Haptic Coding specification SHOULD be assumed, although the sender and receiver MAY use a specific value based on an out-of-band agreement. The parameter 'hmpg-profile' is used to restrict the number of tools used (e.g., the simple-parametric profile fits enable simpler implementations than the main profile). If it is not specified, the most general profile "main" SHOULD be assumed, although the sender and receiver MAY use a specific value based on an out-of-band agreement. The parameter 'hmpg-lvl' is used to further characterize implementations within a given profile, e.g., according to the maximum supported number of channels, bands, and perceptions. If it is not specified, the most general level "2" SHOULD be assumed, although the sender and receiver MAY use a specific version based on an out-of-band agreement.

Other parameters can be used to indicate bitstream properties as well as receiver capabilities. The parameters 'hmpg-maxlod', 'hmpg-avtypes', 'hmpg-bodypartmask', 'hmpg-maxfreq', 'hmpg-minfreq', 'hmpg-dvctypes', and 'hmpg-modalities' can be sent by a sender to reflect the characteristics of bitstreams and can be set by a receiver to reflect the nature and capabilities of local actuator devices, or a preferred set of bitstream properties. For example, different receivers may have different sets of local actuators, in which case these parameters can be used to select a stream adapted to the receiver. In some other cases, some receivers may indicate a preference for a set of bitstream properties such as perceptions, min/max frequency, or body-part-mask, which contribute the most to the user experience for a given application, in which case these parameters can be used to select a stream which include and possibly prioritizes those properties.

The parameter 'hmpg-silencesupp' can be used to indicate sender and receiver capabilities or preferences. This parameter indicates whether silence suppression should be used, as described in {{mihs-trans}}.



## Declarative SDP considerations

When haptic content over RTP is offered with SDP in a declarative style, the parameters capable of indicating both bitstream properties as well as receiver capabilities are used to indicate only bitstream properties.  For example, in this case, the parameters hmpg-maxlod, hmpg-bodypartmask, hmpg-maxfreq, hmpg-minfreq, hmpg-dvctypes, and hmpg-modalities declare the values used by the bitstream, not the capabilities for receiving bitstreams. A receiver of the SDP is required to support all parameters and values of the parameters provided; otherwise, the receiver MUST reject or not participate in the session.  It falls on the creator of the session to use values that are expected to be supported by the receiving application.

# Congestion control consideration

The general congestion control considerations for transporting RTP data apply to HMPG haptics over RTP as well {{RFC3550}}. 

It is possible to adapt network bandwidth by adjusting either the encoder bit rate or by adjusting the stream content (e.g., level of detail, body parts, actuator frequency range, target device types, modalities).
 
In case of congestion, a receiver or intermediate node MAY prioritize independent packets over dependent ones, since the non reception of an independent MIHS unit can prevent the decoding of multiple subsequent dependent MIHS units. In case of congestion, a receiver or intermediate node MAY prioritize initialization MIHS units over other units, since initialization MIHS units contain metadata used to re-initialize the decoder, and MAY drop silent MIHS units before other types of MIHS units, since a receiver may interpret a missing MIHS unit as a silence. It is also possible, using the layer field of the RTP payload header, to allocate MIHS units to different layers based on their content, to prioritize haptic data contributing the most to the user experience. In case of congestion, intermediate nodes and receivers SHOULD use the MIHS layer value to determine the relative importance of haptic RTP packets.



# Security Considerations

This RTP payload format is subject to security threats commonly associated with RTP payload formats, as well as threats specific to the interaction of haptic devices with the physical world, and threats associated with the use of compression by the codec. 
Security consideration for threats commonly associated with RTP payload formats are outlined in {{RFC3550}}, as well as in RTP profiles such as RTP/AVP {{RFC3551}}), RTP/AVPF {{RFC4585}}, RTP/SAVP {{RFC3711}}, or RTP/SAVPF {{RFC5124}}.

Haptic sensors and actuators operate within the physical environment. This introduces the potential for information leakage through sensors, or damage to actuators due to data tampering. Additionally, misusing the functionalities of actuators (such as force, position, temperature, vibration, electro-tactile, etc.) may pose a risk of harm to the user, for example by setting keyframe parameters (e.g., amplitude, position, frequency) or channel gain to a value that surpasses a permissible range. While individual devices can implement security measures to reduce or eliminate those risks on a per-device basis, in some cases harm can be inflicted by setting values which are permissible for the individual device. For example, causing contact with the physical environment or triggering unexpected force feedback can potentially harm the user. Each haptic system should therefore implement system-dependent security measures, which is more error prone. To limit the risk that attackers exploit weaknesses in haptic systems, it is important that haptic transmission should be protected against malicious traffic injection or tempering.

However, as "Securing the RTP Framework: Why RTP Does Not Mandate a Single Media Security Solution" {{RFC7202}} discusses, it is not an RTP payload format's responsibility to discuss or mandate what solutions are used to meet the basic security goals like confidentiality, integrity, and source authenticity for RTP in general. This responsibility lays on anyone using RTP in an application. They can find guidance on available security mechanisms and important considerations in "Options for Securing RTP Sessions" {{RFC7201}}. Applications SHOULD use one or more appropriate strong security mechanisms.

The haptic codec used with this payload format uses a compression algorithm (see sections 8.2.8.5 and 8.3.3.2 in {{ISO.IEC.23090-31}}). An attacker may inject pathological datagrams into the stream which are complex to decode and cause the receiver to be overloaded, similarly to {{RFC3551}}. 

End-to-end security with authentication, integrity, or confidentiality protection will prevent a Media-Aware Network Element (MANE) from performing media-aware operations other than discarding complete packets. In the case of confidentiality protection, it will even be prevented from discarding packets in a media-aware way. To be allowed to perform such operations, a MANE is required to be a trusted entity that is included in the security context establishment.


# IANA Considerations

This memo updates the media type registration of haptics/hmpg with IANA, in {{format-param}}. 
