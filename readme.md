# Moddable SDK support for M5Core Ink
Updated October 6, 2021

This is experimental. The display fundamentals are working. Feel free to help.

The Moddable SDK Piu that do not depend on touch generally seem to work as-is.

This porting effort depends on the new APIs standardized by [Ecma-419](https://419.ecma-international.org). Be sure to grab the latest Moddable SDK to get some fixes that this code depends on.

This work is similar to the M5Paper port, though the display driver is entirely different as the devices use dramatically different display controllers.

## Setup, build, run

Copy the directory at `targets/m5core_ink` to `$MODDABLE/build/devices/esp32/targets`

Then `cd` to `$MODDABLE/examples/piu/balls` directory and build as usual:

```
mcconfig -d -m -p esp32/m5core_ink
```

The display refresh rate is about 3 FPS. The balls bounce slowly.

## Porting Status

The following are implemented and working:

- EPD display driver
- RTC
- Up / Down / Middle / Power / External buttons 
- LED

To be done:

- Buzzer

Known issues:

- The setup code sets GPIO 12 high to try to keep the power on when running on battery power. But that doesn't seem to work. 

## Display Driver

The display driver is a [Poco `PixelsOut`](https://github.com/Moddable-OpenSource/moddable/blob/public/documentation/commodetto/commodetto.md#pixelsout-class) implementation. This allows it to use both the [Poco](https://github.com/Moddable-OpenSource/moddable/blob/public/documentation/commodetto/poco.md) graphics APIs and[ Piu](https://github.com/Moddable-OpenSource/moddable/blob/public/documentation/piu/piu.md) user interface framework from the Moddable SDK.

The display driver is written entirely in JavaScript. It uses [Ecma-419 IO](https://419.ecma-international.org/#-9-io-class-pattern) APIs for all hardware access. Performance is excellent, often faster than the EPD class built into the native M5Core Ink library. 

The display driver implements dithering, which allows many levels of gray to be displayed using only black and white pixels. The default dithering algorithm is the venerable [Atkinson dither](https://twitter.com/phoddie/status/1274054345969950720). To change to Burkes or to disable dithering:

```js
screen.dither = "burkes";
screen.dither = "none"
```

Dithering is performed using the new `commodetto/Dither` module.

To support dithering the driver requires that Poco render at 8-bit gray (256 Gray). The driver also only supports full screen updates as partial updates with dithering show seams. Partial updates could, and probably should, be supported as they could be useful when not using dithering.

The driver maintains a back buffer, which is more-or-less necessary because of the way the display controller works. Fortunately the buffer is just 5000 bytes, since it can use 1-bit pixels.

Hardware rotation is not supported. It could be, but given that the display is square it isn't obviously useful. Both Poco and Piu support software rotation at build time, so rotation is available if needed, just not at runtime.

## Buttons

All of the buttons are available with callbacks when changed. Here's a basic test that installs callbacks on each.

```js
for (let name in device.peripheral.button) {
	new device.peripheral.button[name]({
		onPush() {
			trace(`Button ${name}: ${this.pressed}\n`);
		}
	})
}
```

## LED

The green LED at the top of the unit is available:

```js
const led = new device.peripheral.led.Default;
led.on = 1;		// full strength
led.on = 0;		// off
led.on = 0.5;		// half strength
```
