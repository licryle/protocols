# [Beurer-TL100](https://www.beurer.com/web/fr/produits/wellbeing/therapie-par-la-lumiere/lampe-de-luminotherapie/tl-100.php)
Tech: [BLE](https://www.polidea.com/blog/bluetooth-low-energy-sniffing-guide-part-2/)

## Services list
* OTA (Over The Air updates) | *UUID: 00010203-0405-0607-0809-0a0b0c0d2b12*
* Lamp Control | *UUID: 14839ac4-7d7e-415c-9a42-1673-40cf2339*
  * Control Characteristic | *UUID: 8B00ACE7-EB0B-49B0-BBE9-9AEE0A26E1A3* => main focus today
  * Subscribable Characteristic | *UUID: 0734594A-A8E7-4B1A-A6B1-CD5243059A57* => sometimes spits a confirmation OR error code "0A"
  * Unknown Subscribable/Wrtable Characteristic | *UUID: BA04C4B2-892B-43BE-B69C-5D13F2195392*
  * Unknown Characteristic | *UUID: E06D5EFB-4F4A-45C0-9EB1-371AE5A14AD4* => always reads 0x01

## Lamp Control Characteristic - General Commands structure:
[Header: 0xFEEF0a09ABAA0][Command Type: [43,53,63,93]][Values: various][Footer: 0x550D0A]
Everything is Hexadecimal

* Header: 0xfeef0a09abaa0
* Command Type:
  * 0x43: ON/OFF commands
    * 0x4370132 --- White on (mid-intensity)
    * 0x4350130 --- White off
    * 0x4370231 --- Colors on (coupled with rainbow)
    * 0x4350233 --- Colors off
    * 0x438023E --- Timer ON (2h)
    * 0x4360230 --- Timer OFF
    * Color modes
      * Rainbow: 0x70231 (on as well)
      * Static: 0x40030
      * Mode x: 0x40131
      * Mode x: 0x40232
      * Mode x: 0x40333
      * Mode x: 0x40434
      * Mode x: 0x40535
      * Mode x: 0x40636
      * Mode x: 0x40737
      * Mode x: 0x40838
      * Mode x: 0x40939
      * Mode x: 0x40A3A
  * 0x53: Intensity
    * 0x53101 -- [0x0001 until 0x9999] white dimming
    * 0x53102 -- [0x0000 until 0x9999] color dimming
    * 0x53302 -- [0x0100 until 0x78FF] timer length - where each 0x100 (or 256d) = 1min. => 1 to 121min.
      * Note 2: hardware seem to truncate to the minute, so 0x0099 bing under 1min is an immediate shutdown.
  * 0x63: Changing Colors, followed by the color in strange format 0x2RRGGBB00
    * For example, red is 0x632FF000000
    * Changing colors stops the color modes

* Footer: 0x550D0A

## Lamp Control Characteristic - Protocol

### Warnings
1. White and Colors are 2 different modes, coded completely separately
2. Any "ON" command is coupled with a predefined value. This also means you cannot turn on with a dimmed value, it turns on straight to a predefined value, then you can change it.

3. Any command in the wrong state will "stuck" the device and you'll need to reset.
** Sending any "white" command (except "white on") while in "color" mode crashes the device.
** Sending any "color" command (except "color on") while in "white" mode crashes the device.
To start anything, ensure to send either "white on" or "color on" before any other command.

If it craches, just press the physical on/off and reconnect to it.

Because of this, and the non-existance of a read mechanism from the device, you'll notice:
- the Beuer App stores the state of the device, and loses it upon disconnect.
- If th eapp is connected, the Timer on the device is disabled.

## Lamp Control Characteristic examples:
### White
1. ON:   feef0a09abaa04370132550d0a
2. FULL: feef0a0aabaa0531019999550d0a
3. MID:  feef0a0aabaa0531015000550d0a
4. LOW:  feef0a0aabaa0531010001550d0a
5. OFF:  feef0a09abaa04350130550d0a

### Color
1. ON:   feef0a09abaa04370231550d0a (rainbow)
2. FULL: feef0a0aabaa0531029999550d0a
3. MID:  feef0a0aabaa0531025000550d0a
4. LOW:  feef0a0aabaa0531020001550d0a
5. OFF:  feef0a09abaa04350233550d0a

* RED:   feef0a09abaa0632FF000000550d0a
* BLUE:  feef0a09abaa063200FF0000550d0a
* GREEN: feef0a09abaa06320000FF00550d0a

### Timer
1. Timer   ON: feef0a09abaa0438023e550d0a (121min)
2. Timer 121m: feef0a0aabaa05330278FF550d0a (max)
3. Timer   1m: feef0a0aabaa0533020100550d0a (min)
4. Timer  OFF: feef0a09abaa04360230550d0a
