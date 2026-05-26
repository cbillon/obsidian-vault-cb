---
link: https://cylab.be/blog/337/sending-sms-with-a-usb-gsm-modem-and-python
excerpt: Despite its occasional unreliability as seen here and there, SMS
  remains a common communication method. Before diving into Python,
  understanding AT commands is a good start for interfacing with a USB GSM
  modem. This concise guide lays out the steps to integrate SMS functionality
  into your projects, enabling you to utilize the power of SMS communication
  with ease.
slurped: 2026-05-19T08:50
title: Sending SMS with a USB GSM modem (and Python)
---

Despite its occasional unreliability as seen [here](https://cylab.be/blog/171/sms-based-2-factor-authentication-is-insecure) and [there](https://cylab.be/blog/326/are-sms-messages-vulnerable-in-5g), SMS remains a common communication method. Before diving into Python, understanding AT commands is a good start for interfacing with a USB GSM modem. This concise guide lays out the steps to integrate SMS functionality into your projects, enabling you to utilize the power of SMS communication with ease.

## Supported devices[](https://cylab.be/blog/337/sending-sms-with-a-usb-gsm-modem-and-python#supported-devices)

The first thing you need is a device that can act upon a SIM card. Most USB GSM modems should be able to send SMS, here are a few examples:

- MF190 GSM Modem by ZTE
- OSTENT Modem GSM with Wavecom Q2303A module
- …

You may even use a SIM pool if you need multiple SIM cards. Of course, one SIM card is needed per GSM modem. Assuming you use a Linux-based OS, after being plugged in, your modem(s) will be found like any other USB device via:

```
ls /dev/ttyUSB*
```

## Communicating with the modem[](https://cylab.be/blog/337/sending-sms-with-a-usb-gsm-modem-and-python#communicating-with-the-modem)

In order to communicate with the modem and control it, you will need to use AT commands. AT commands, short for “Attention Commands”, are a standardized set of instructions used to communicate with and control modems, phones, and other devices over a serial interface. These commands typically start with the characters `AT` followed by specific instructions or parameters. They serve as a universal language for configuring settings, initiating actions (such as making phone calls or sending SMS), and retrieving information from these devices. Understanding AT commands is essential for effectively interfacing with GSM modems and leveraging their functionalities in various applications. The subset of commands you can use from this standardized set of commands will depend on your USB modem model. Documentation manuals can be easily found online.

### Checking the device

The first command to check if you indeed communicate with a modem that understands AT commands is the following:

```
AT
```

If everything goes well, you’ll receive the answer:

```
OK
```

For future commands, I’ll precede the input command with the `>` symbol, so let’s write the previous communication again:

```
>AT
OK
```

### Unlocking the SIM card

OK, let’s go further by unlocking the SIM card. In fact, if you did not disable this feature, you’ll need to send the correct PIN code as you would have to when booting on a mobile phone. You can check if the SIM card is locked or not using:

```
>AT+CPIN?
+CPIN: SIM PIN
```

In this case, the received message indicates that the SIM card is locked. That code is composed of four digits `XXXX` to replace by your own PIN code, here is how to unlock the SIM card:

```
>AT+CPIN="XXXX"
OK
```

After unlocking the SIM card, we ask again in which state it is thanks to the `AT+CPIN?` command and we see that the SIM card is ready for further use.

```
> AT+CPIN?
+CPIN: READY
```

### Sending an SMS

Let’s send an SMS to the mobile phone number `32XXXXXXXXX` (beginning with the country code but without a `+` sign). This must be done using the `AT+CMGS` command with the phone number, followed by the return carriage `\r`, then the message ended with the `CTRL-Z` symbol (the byte `\x1A`):

```
>AT+CMGS="32XXXXXXXXX"\r
>This is a test message.\x1A
```

### Reading received SMS

You can also read the messages you received via the command `AT+CMGR=X` where `X` is the Xth message you received, for example:

```
>AT+CMGR=1
+CMGR: "REC READ","+32XXXXXXXXX",,"24/03/13,19:15:11+04"
The test message worked!
```

Note that the text message you receive might have been sent in PDU mode, meaning it is encoded with a specific encoding such as [GSM-7](https://en.wikipedia.org/wiki/GSM_03.38).

## Sending AT commands with Python[](https://cylab.be/blog/337/sending-sms-with-a-usb-gsm-modem-and-python#sending-at-commands-with-python)

Here comes the practical Python implementation! We’ll first use the long and more detailed method using the `serial` module that implements AT commands as seen previously, then we’ll use the `gsmmodem` module that uses `serial` under the hood.

### Using the serial module

The Python module is installed by default. It is used to communicate with any USB device. Here is how it can be used to communicate with an USB GSM modem, assuming the USB device is located at the port `/dev/ttyUSB0`:

```
import serial, time

# Initiating
ser = serial.Serial()
ser.port = "/dev/ttyUSB0"
ser.baudrate = 115200 # transmission speed depending on the device
ser.writeTimeout = 20 # max time to wait for write operations to complete
ser.open()

# Entering PIN code
pin_code = ... # string
ser.write('AT+CPIN="{}"'.format(pin_code).encode())
time.sleep(2)
print(ser.readline().decode())

# Sending SMS
recipient_number = ... # string
message = 'This is a test message.'
ser.write('AT+CMGS="{}"\r'.format(recipient_number).encode())
time.sleep(2)
print(ser.readline().decode())
ser.write(message.encode() + b'\x1A') # CTRL-Z ending
time.sleep(2)
print(ser.readline().decode())

# Closing
ser.close()
```

### Using the gsmmodem module

This module can be installed via:

```
python3 -m pip install python-gsmmodem-new
```

It has built-in methods that encapsulate AT commands without the need to know them:

```
from gsmmodem.modem import GsmModem

pin_code = ... # string
recipient_number = ... # string
message = 'This is a test message.'

modem = GsmModem('/dev/ttyUSB0', 115200)
modem.smsTextMode = False # use PDU mode
modem.connect(pin_code)

modem.sendSms(recipient_number, message)

modem.close()
```

## Conclusion[](https://cylab.be/blog/337/sending-sms-with-a-usb-gsm-modem-and-python#conclusion)

Using an USB GSM modem is quite easy. Depending on your modem, you may access several additional SIM card options and features, such as what data it must store, making phone calls and so on. However, this ease of access also means that scammers may easily automate SMS phishing, also known as smishing. Being aware of the potential risks and implementing robust security measures is essential to safeguarding your personal information in an increasingly interconnected world.

This blog post is licensed under [CC BY-SA 4.0 ![creative commons](https://cylab.be/images/cc.svg) ![attribution](https://cylab.be/images/by.svg) ![share-alike](https://cylab.be/images/sa.svg)](http://creativecommons.org/licenses/by-sa/4.0/?ref=chooser-v1)