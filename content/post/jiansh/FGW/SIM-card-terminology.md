+++
title = "SIM-card-terminology"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "FGW"
]
toc = true
+++



[link on JianShu](https://www.jianshu.com/p/1333629e5b35)

###SIM
[wiki SIM_card](https://en.wikipedia.org/wiki/SIM_card) 
>A subscriber identity module or subscriber identification module (**SIM**), widely known as a **SIM card**,  is an [integrated circuit](https://en.wikipedia.org/wiki/Integrated_circuit "Integrated circuit") that is intended to [securely](https://en.wikipedia.org/wiki/Encryption "Encryption") store the [international mobile subscriber identity](https://en.wikipedia.org/wiki/International_mobile_subscriber_identity "International mobile subscriber identity") (IMSI) number and its related [key](https://en.wikipedia.org/wiki/SIM_card#Authentication_key)

>The SIM circuit is part of the function of a [universal integrated circuit card](https://en.wikipedia.org/wiki/Universal_integrated_circuit_card "Universal integrated circuit card") (UICC) physical [smart card](https://en.wikipedia.org/wiki/Smart_card "Smart card"), which is usually made of [PVC](https://en.wikipedia.org/wiki/PVC "PVC") with embedded contacts and [semiconductors](https://en.wikipedia.org/wiki/Semiconductor "Semiconductor"). SIM cards are transferable between different mobile devices. The first UICC smart cards were the size of credit and bank cards; sizes were reduced several times over the years, usually keeping electrical contacts the same, so that a larger card could be cut down to a smaller size.

>A SIM card contains its unique serial number ([ICCID](https://en.wikipedia.org/wiki/ICCID "ICCID")), international mobile subscriber identity (IMSI) number, security authentication and ciphering information, temporary information related to the local network, a list of the services the user has access to, and two passwords: a [personal identification number](https://en.wikipedia.org/wiki/Personal_identification_number "Personal identification number") (PIN) for ordinary use, and a [personal unblocking code](https://en.wikipedia.org/wiki/Personal_unblocking_code "Personal unblocking code") (PUC/PUK) for PIN unlocking.

Your **PIN** (Personal Identification Number) code is an access code composed of 4 digits that you received with your SIM card. With this code, you have access to Carrier’s network. You must enter the PIN code each time you switch on your mobile.

The **PUK** (Personal Unblocking Key) code is a code consisting of 8 digits that you also received with your SIM card. It is used to unblock your SIM card when you entered 3 times a wrong PIN code.

---
###ICCID

[what is ICCID/](https://www.imei.info/faq-what-is-ICCID/)

ICCID (Integrated Circuit Card Identifier) - A SIM card contains its unique serial number (ICCID). ICCIDs are stored in the SIM cards and are also printed on the SIM card during a personalisation process. The ICCID is defined by the ITU-T recommendation E.118 as the Primary Account Number. A full ICCID is 19 or 20 characters.

The format of the ICCID is: MMCC IINN NNNN NNNN NN C x
MM = Constant (ISO 7812 Major Industry Identifier)
CC = Country Code
II= Issuer Identifier
N{12} = Account ID ("SIM number")
C = Checksum calculated from the other 19 digits using the Luhn algorithm.
x= An extra 20th digit is returned by the 'AT!ICCID?' command, but it is not officially part of the ICCID.

---
### IMSI 
[wiki IMSI](https://en.wikipedia.org/wiki/International_mobile_subscriber_identity)
>The **international mobile subscriber identity** or **IMSI** [/ˈɪmziː/](https://en.wikipedia.org/wiki/Help:IPA/English "Help:IPA/English") is a number that uniquely identifies every user of a [cellular network](https://en.wikipedia.org/wiki/Cellular_network "Cellular network").<sup>[[1]](https://en.wikipedia.org/wiki/International_mobile_subscriber_identity#cite_note-1)</sup> It is stored as a 64-bit field and is sent by the mobile device to the network.

IMSI is unique International Mobile subscriber Indetity. It is stored inside the SIM.  It consists of three part.
- Mobile Country Code (MCC) : First 3 digits of IMSI gives you MCC.
- Mobile Network Code (MNC) : Next 2 or 3 digits give you this info.
- Mobile Station ID (MSID) : Rest of the digits. Gives away the network you are using like IS-95, TDMA , GSM etc.

Example:
```
MCC  310  USA
MNC 150   AT&T Mobility
MSIN 123456789
```

---
###IMEI
[wiki IMEI](https://en.wikipedia.org/wiki/International_Mobile_Equipment_Identity)  
Just key in `*#06#` on your mobile phone and it will display its IMEI number in most phones.  

The **International Mobile Equipment Identity** or **IMEI**<sup>[[1]](https://en.wikipedia.org/wiki/International_Mobile_Equipment_Identity#cite_note-3gppspec-1)</sup> is a number, usually unique,<sup>[[2]](https://en.wikipedia.org/wiki/International_Mobile_Equipment_Identity#cite_note-bbcnews-2)</sup><sup>[[3]](https://en.wikipedia.org/wiki/International_Mobile_Equipment_Identity#cite_note-3)</sup> to identify [3GPP](https://en.wikipedia.org/wiki/3GPP "3GPP") and [iDEN](https://en.wikipedia.org/wiki/IDEN "IDEN") [mobile phones](https://en.wikipedia.org/wiki/Mobile_phone "Mobile phone"), as well as some [satellite phones](https://en.wikipedia.org/wiki/Satellite_phone "Satellite phone").

Devices without a SIM card slot usually don't have the IMEI code. However, the IMEI only identifies the device and has no particular relationship to the subscriber. The phone identifies the subscriber by transmitting the [International mobile subscriber identity](https://en.wikipedia.org/wiki/International_mobile_subscriber_identity "International mobile subscriber identity") (IMSI) number, which it stores on a SIM card that can, in theory, be transferred to any handset. However, the network's ability to know a subscriber's current, individual device enables many network and security features.




