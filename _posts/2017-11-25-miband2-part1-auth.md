---
layout: post
title: "Mi Band 2, Part 1: Authentication"
---

I found out that playing with the [**Mi Band 2**](http://www.mi.com/en/miband2/) withouth the official app was not so forward like I thought.

After pairing, the device would only stay connected for a few seconds before disconnecting and there was no response from the usage of **GATT** services like for example writing `0x01` to the [Immediate Alert](https://www.bluetooth.com/specifications/gatt/viewer?attributeXmlFile=org.bluetooth.service.immediate_alert.xml) service.

By taking a look of how [others implement communications](https://github.com/Freeyourgadget/Gadgetbridge/blob/1ddea9268d08edaf4b71c587e6db37017ea55c8b/app/src/main/java/nodomain/freeyourgadget/gadgetbridge/service/devices/miband2/operations/InitOperation.java) with the band I found out that there is an [authentication procedure](#authentication-procedure) that takes place before the band giving you access to the **GATT** services.

# Authentication procedure

The procedure is as follows:

1. A **16 Byte** key is sent to the Band
2. A random number is requested from the Band
3. The number is then encrypted with the shared **16 Byte** key using **`AES/ECB/NoPadding`** and sent to the Band.

I guess the band internally encrypts the random number with the previously received key and then compares to the received encrypted number.

This authentication procedure takes place on a custom **GATT** service, `UUID 0x0009`.

### Enable notifications

First we need to enable for notifications at [`UUID 0x0009` (handle: 0x0051)](#reference).

### Sending a "secret" key to the band

Write [`{ 0x01, 0x08, SECRET_KEY}` (Send Key command, Auth Byte, Secret Key)](#reference) to [`UUID 0x0009` (handle: 0x0050)](#reference).

### Requesting a random authentication key from the band

We check that the first three bytes we get as a notification from the last write command are [`{ 0x10, 0x01, 0x01 }` (Response, Send Key command, Success)](#reference), then we make another request by writing [`{ 0x02, 0x08}` (Request random number command, Auth Byte)](#reference) to [`UUID 0x0009` (handle: 0x0050)](#reference).

### Sending the encrypted random authentication key to the band

We check that the first three bytes we get as a notification from the last write command are [`{ 0x10, 0x02, 0x01 }` (Response, Request random number command, Success)](#reference) and if so, we first start by encrypting the last **16 bytes** from the response using `AES/ECB/NoPadding` and the [SECRET_KEY](#reference), then we use that value to make another request by writing [`{ 0x03, 0x08, ENCRYPTED_RESPONSE }` (Send encrypted random number command, Encrypted random number)](#reference) to [`UUID 0x0009` (handle: 0x0050)](#reference).

### Check the final response

If the first three bytes we get as a notification from the last write command are [`{ 0x10, 0x03, 0x01 }` (Response, Send encrypted random number command, Success)](#reference) all went well.

# Putting into code

I used [Python 2](https://www.python.org) and [bluepy](https://github.com/IanHarvey/bluepy) to create the automation and testing of this procedure.

The code authenticates and sends 2 notification (0x01 and 0x02) with 3s of duration each, the code can be found at [miband2_auth.py](https://github.com/leojrfs/miband2/blob/master/miband2_auth.py).

# Reference

```java
UUID_CHARACTERISTIC_AUTH = "00000009-0000-3512-2118-0009af100700"
AUTH_SEND_KEY = 0x01
AUTH_REQUEST_RANDOM_AUTH_NUMBER = 0x02
AUTH_SEND_ENCRYPTED_AUTH_NUMBER = 0x03
AUTH_RESPONSE = 0x10
AUTH_SUCCESS = 0x01
AUTH_FAIL = 0x04
AUTH_BYTE = 0x8
SECRET_KEY = {0x30, 0x31, 0x32, 0x33, 0x34, 0x35, 0x36, 0x37, 0x38, 0x39, 0x40, 0x41, 0x42, 0x43, 0x44, 0x45}
```

# Credits

- [Gadgetbridge](https://github.com/Freeyourgadget/Gadgetbridge/blob/master/app/src/main/java/nodomain/freeyourgadget/gadgetbridge/devices/miband/MiBand2Service.java)
- [DontPanic @ stackoverflow](https://stackoverflow.com/questions/41417747/connection-to-mi-band-2?answertab=votes#tab-top)
- [Mifit 2 app analysis](https://medium.com/@cirku17/more-mifit-2-app-analysis-e2bbb4674093)