# SPI TFT panels on composed boards

A composed ESP32-S3 board can drive an ST7789-class TFT over a plain 4-wire
SPI bus (MOSI, SCLK, CS, DC) by selecting the `st7789` display binding with
`"interface": "spi"`:

```json
{
  "driver": "st7789",
  "interface": "spi",
  "width": 240,
  "height": 320,
  "pclkHz": 40000000,
  "spiMode": 0,
  "colorOrder": "bgr",
  "invertColors": false,
  "pins": { "mosi": 16, "sclk": 21, "cs": 47, "dc": 17 }
}
```

`width` and `height` are the panel's native geometry in its portrait
addressing. The pipeline keeps the controller in portrait addressing and
rotates landscape frames at flush time, so an app that wants a landscape
logical surface sets `GEA_EMBEDDED_DISPLAY_WIDTH` and
`GEA_EMBEDDED_DISPLAY_HEIGHT` in its board `defines`; the native pair stays
with the board.

`pins.reset` and `pins.backlight` are optional. Leave them out when the
module's reset follows its enable line or the backlight is wired on: no LEDC
channel is claimed and brightness is a no-op. With a backlight pin, brightness
is PWM on that line.

`pclkHz` selects the SPI clock from 1–80 MHz (default 40 MHz; ST7789 glass is
commonly run at 80 through direct IO_MUX pins, so raise it after first light).
`spiMode` is the SPI mode, 0–3 (default 0). `colorOrder` is `rgb` or `bgr`
(default `rgb`), and `invertColors` (default `false`) says whether the glass
runs inverted, which most ST7789 IPS modules do and ILI9341-class modules do
not. `gapX`, `gapY`, `mirrorGapX` and `mirrorGapY` (default 0) place a glass
smaller than the controller's 240x320 GRAM in the two portrait orientations.

The binding is the M5Stack StickS3 pipeline from `@geastack/targets`, shared
with the Waveshare Touch LCD 1.54; a composed board gets the same staged DMA
flush, rotate-on-flush landscape and panel-endian framebuffer. An FT6336 touch
controller on the shared I2C bus composes alongside it; its reset and
interrupt pins are optional, and with no interrupt line the binding polls.
