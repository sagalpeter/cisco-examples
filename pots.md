# POTS (Plain old telephone system) with VoIP and multiple router interconnect

My goal in this one was to interconnect rotary dial telephones for my kids, and as usual, it went over the initial idea, so I finished with connecting also VoIP devices (android phone with linphone app) and also trying to connect friend's kids in other locality with another POTS devices.

### Locality #1

If you want just to make calls betewwn devices cabled to one central point, this locality is enough for you.

- **Cisco 2901 ISR**
- **FXS card**, in this locality 4 port one [VIC3-4FXS/DID](https://www.cisco.com/c/en/us/td/docs/routers/access/interfaces/ic/hardware/installation/guide/4port_FXS_DID_VIC.html) Foreign exchange station - these ports serve as "wall socket" connected to PBX, they power connected phone. Beware, there are also FXO (Foreign Exchange Office) cards in the wild, but those are not any use for this scenario, as they serve as "phone" side.
- **PVDM DIMM module**, in this locality [PVDM3-32](https://www.cisco.com/c/en/us/products/collateral/interfaces-modules/voice-modules-interface-cards/data_sheet_c78-553971.html). This is voice processing DSP, necessary for POTS.
- **Configuration**
  - IP address: 10.48.0.101
  - Phone number prefix: 48
  - VoIP

### Locality #2

This is router at my friend's, will serve as other locality in this example. VPN interconnect configuration is not scope of this example, just assume that both devices can reach each other.

- **Cisco 2901 ISR**
- **FXS card**, in this locality VIC3-2FXS/DID
- **PVDM DIMM module**, in this locality PVDM3-16
- **Configuration**
  - IP address: 10.67.0.101
  - Phone number prefix: 67

## POTS calls in one router

First we are going to configure POTS on Locality #1, this is the most basic config to enable calling between phones connected to FXS ports on one router. I assume IP address and other basic config is already configured.

Check inventory of router, my output showing router type, PVDM card and FXS card:

    c2901-loc-1#show inventory
    NAME: "CISCO2901/K9", DESCR: "CISCO2901/K9 chassis, Hw Serial#: F**********, Hw Revision: 1.0"
    PID: CISCO2901/K9      , VID: V06 , SN: F**********

    NAME: "3rd generation four port FXS DID voice interface daughtercard on Slot 0 SubSlot 0", DESCR: "3rd generation four port FXS DID voice interface daughtercard"
    PID: VIC3-4FXS/DID     , VID: V03 , SN: F**********

    NAME: "PVDM3 DSP DIMM with 32 Channels on Slot 0 SubSlot 4", DESCR: "PVDM3 DSP DIMM with 32 Channels"
    PID: PVDM3-32          , VID: V01 , SN: F**********

Configure POTS in few steps
- Enter configuration mode

      configure terminal
- Enable voice services for POTS

      voice service pots
- Voice port 0/0/0 configuration explained line by line. This configuration is related to "outgoing" calls.

    |||
    | --- | --- |
    | ``voice-port 0/0/0`` | port 0/0 on FXS card in slot 0 |
    | ``ring cadence pattern03`` | ring cadence |
    | ``compand-type a-law`` | use ``u-law`` for north america, ``a-law`` for europe |
    | ``cptone SK`` | ringing tone, Slovakia in my case, you can try your county code |
    | ``description FXS port 0/0/0`` | port description |
    | ``station-id name Room 1`` | station identificator name |
    | ``station-id number 48000`` | station number |
    | ``caller-id enable`` | caller-id, this is mainly for VoIP, not needed for POTS only setup |

- All 4 ports copy-paste ready config lines:

      voice-port 0/0/0
       ring cadence pattern03
       compand-type a-law
       cptone SK
       description FXS 0/0/0
       station-id name 48-0/0/0
       station-id number 48000
       caller-id enable
      !
      voice-port 0/0/1
       ring cadence pattern03
       compand-type a-law
       cptone SK
       description FXS 0/0/1
       station-id name 48-0/0/1
       station-id number 48001
       caller-id enable
      !
      voice-port 0/0/2
       ring cadence pattern03
       compand-type a-law
       cptone SK
       description FXS 0/0/2
       station-id name 48-0/0/2
       station-id number 48002
       caller-id enable
      !
      voice-port 0/0/3
       ring cadence pattern03
       compand-type a-law
       cptone SK
       description FXS 0/0/3
       station-id name 48-0/0/3
       station-id number 48003
       caller-id enable

- Dial peer configuration - "incoming" calls, explained line by line

    |||
    | --- | --- |
    | ``dial-peer voice 48000 pots`` | configurin dial peer in POTS configuration, with peer id 48000 (I prefer having this id same as phone number, but it is not mandatory to be the same)
    | ``description 48-0/0/0`` | port description, for configuration visibility purposes only |
    | ``destination-pattern 48000`` | phone number |
    | ``port 0/0/0`` | destination to route the call (voice-port we configured above)|

- All 4  dial-peers copy-paste ready config lines:

      dial-peer voice 48000 pots
       description 48-0/0/0
       destination-pattern 48000
       port 0/0/0
      !
      dial-peer voice 48001 pots
       description 48-0/0/1
       destination-pattern 48001
       port 0/0/1
      !
      dial-peer voice 48002 pots
       description 48-0/0/2
       destination-pattern 48002
       port 0/0/2
      !
      dial-peer voice 48003 pots
       description 48-0/0/3
       destination-pattern 48003
       port 0/0/3

Router on Locality #2 is configured in similar way, just there is only 2 ports FXS card, inserted in slot 2, hence the 0/2/0 numbering:

    voice service pots
    !
    voice-port 0/2/0
     ring cadence pattern03
     compand-type a-law
     cptone SK
     description FXS 0/2/0
     station-id name 67-0/2/0
     station-id number 67020
    caller-id enable
    !
    voice-port 0/2/1
     ring cadence pattern03
     compand-type a-law
     cptone SK
     description FXS 0/2/1
     station-id name 67-0/2/1
     station-id number 67021
     caller-id enable
    !
    dial-peer voice 67020 pots
     description cab67 0/2/0
     destination-pattern 67020
     port 0/2/0
    !
    dial-peer voice 67021 pots
     description cab67 0/2/1
     destination-pattern 67021
     port 0/2/1

## Interconnection

Now, both routers serve as POTS exchange, but calls are possible only between devices directly connected to FXS ports on either router. To interconnect them, voip config is needed to route calls over IP interconnect.

### Locality #1

- Enter configuration mode

      configure terminal
- Enable voice services for VOIP

      voice service voip
      voice call send-alert

- Dial peer to route calls to another device, line by line

    |||
    | --- | --- |
    | ``dial-peer voice 48 voip`` | dial peer for routing to voip |
    | ``destination-pattern 48T`` | calls starting with 67 will match here |
    | ``session target ipv4:10.67.0.101`` | calls will be forwarded to IP |
    | ``codec g711ulaw`` | voice codec to use |
    | ``no vad`` | disable voice activity detection, will use more bandwidth, but increase quality |

- Dial peer, copy-paste:

      dial-peer voice 67 voip
       destination-pattern 67T
       session target ipv4:10.67.0.101
       codec g711ulaw
       no vad

### Locality #2

Router on Locality #2 is configured similarly, only changing prefix and destination of calls.
    
    voice service voip
    voice call send-alert
    !
    dial-peer voice 48 voip
     destination-pattern 48T
     session target ipv4:10.48.0.101
     codec g711ulaw
     no vad

Now, when phone on Locality #1 dials number starting with 67, call will be directed over VoIP to Locality #2, where router checks dial peers, and if there is configured peer with matching destination-pattern, it will route call accordingly. Same in other direction.

## VoIP phones

Now we want to connect also VoIP phone to Locality #1 router, so it can make calls to all other devices connected to all routers. Next configuration I am not sure with, it just works for me, but I have been trying many different scenarios before, so this may not be the most optimal configuration.

- Enter configuration mode

      configure terminal

- Enable voice services for VOIP and enable SIP server

      voice service voip
       allow-connections h323 to h323
       allow-connections h323 to sip
       allow-connections sip to h323
       allow-connections sip to sip
       sip
        registrar server

- Configure VoIP register

      voice register global
       mode cme
       source-address 10.48.0.101 port 5060
       max-dn 10
       max-pool 10
       auto-register
      !
      voice register dn  1
       number 48101
       name linphone 48101
       no-reg
      !
      voice register dn  2
       number 48102
       name linphone 48102
      !
      voice register pool  1
       id device-id-name voipphone1
       number 1 dn 1
       username user1 password password1
       codec g711alaw
      !
      voice register pool  2
      id device-id-name voipphone2
      number 1 dn 2
      username user2 password password2
      codec g711alaw

- Add dial peers

      dial-peer voice 48101 voip
       description linphone t530
      !
      dial-peer voice 48102 voip
       description linphone pixel7

If you have router accessible via for example wifi, you should be able to configure for linphone application on your smartphone and make calls to all other devices in your phone network. 
