# CRT-Control Dynamic

Originally on [CRT-C® – Crosspoint RESPONSIVE Touchscreen-Control](https://shmups.system11.org/viewtopic.php?f=6&t=69630) which allows you to quickly configure presets.

----------

This is a different appraoch and it a dynamic controller for Extron Crosspoint. Instead of pre-programming routes on the Extron unit itself, **you send ties (inputs & output combinations) on the fly — select an input, pick one or more outputs, and hit Submit**

## What it does

- Dynamically generates Extron tie commands and sends them via HTTP to your Extron unit
- Supports routing video and/or audio independently to multiple outputs at once
- Includes a **Clear All Previous** toggle that wipes all existing ties before applying new ones — useful when switching games/consoles entirely

---

## How the ties work

Extron Crosspoint units accept routing commands over HTTP using a specific string syntax. This page constructs those commands using three symbols to specify what type of signal is being tied:

| Symbol | Meaning |
|--------|---------|
| `%`    | Video only |
| `$`    | Audio only |
| `!`    | Both video and audio |

A full command looks like this:

```
W+Q{input}*{output}{symbol}
```

For example, routing input 3 to output 2 (video only):

```
W+Q3*2%
```

Multiple ties can be chained in a single command. The page URL-encodes this string and sends it to the Extron via a GET request:

```
control-dynamic.html?cmd=W%2BQ3*2%25
```

When **Clear All Previous** is enabled, the command first sends `0*1!` through `0*{MAX_OUTPUTS}!` to clear every output before applying your new routing. This was a hack that I found, but if anyone knows of a better way feel free to send a PR!

---

## Setup

The initial setup process is the same as described in the original forum post — but importantly, **you do not need to buy a serial-to-USB cable**. I won't go into too much detail but you can most likely do the following instead:

1. Temporarily change your PC's subnet to match the Extron's default IP range
2. Access the Extron's web interface and change its IP address to one on your normal home network range
3. From that point on, you can reach the Extron directly from your browser on your regular network

Once the Extron is on your network, drop `control-dynamic.html` into any location your browser can open it from (local file, NAS, simple HTTP server, etc.) and you're good to go.

---

## Configuring for your system

There are three things to update in the `<script>` block at the top of `control-dynamic.html`:

### 1. `inputTypes`

Define each input on your matrix. The key is the Extron input number (1-based), `name` is a label, and `image` is a filename inside the `images-png/` folder.

```javascript
const inputTypes = {
  1: { name: "NES",       image: "nes.png" },
  2: { name: "SNES",      image: "snes.png" },
  3: { name: "N64",       image: "n64.png" },
  4: { name: "Gamecube",  image: "gamecube.png" },
  5: { name: "Wii",       image: "wii.png" },
  6: { name: "PS1",       image: "ps1.png" },
  7: { name: "PS2",       image: "ps2.png" },
  8: { name: "Xbox360",   image: "xbox360.png" },
};
```

### 2. `outputTypes`

Define each output destination. `outputButton` maps to the physical Extron output number. `type` must be `"video"`, `"audio"`, or `"both"`. Two entries can share the same `outputButton` as long as their `type` differs (e.g., pairing a video output and its embedded audio channel).

```javascript
const outputTypes = {
  1: { outputButton: 1, name: "CRT YPbPr",     image: "crt_component.png",  type: "video" },
  2: { outputButton: 2, name: "CRT Composite",  image: "crt_composite.png",  type: "video" },
  3: { outputButton: 3, name: "PVM YPbPr",      image: "pvm_component.png",  type: "video" },
  4: { outputButton: 4, name: "PVM Composite",  image: "pvm_composite.png",  type: "video" },
  5: { outputButton: 5, name: "Speakers",       image: "speakers.jpg",       type: "audio" },
  6: { outputButton: 1, name: "CRT Audio",      image: "crt_audio.png",      type: "audio" },
};
```

Note: the validation function will alert you on load if you accidentally configure two outputs with the same `outputButton` + `type` combination.

### 3. `MAX_OUTPUTS`

Set this to the total number of physical outputs on your Extron unit. This is only used when **Clear All Previous** is active — it determines how many outputs get zeroed out before the new tie is applied.

```javascript
const MAX_OUTPUTS = 8;
```

---

## Adding custom images

Place any images you want to use for inputs/outputs into the `images-png/` folder alongside `control-dynamic.html`.

Limitations / Not yet supported
The current implementation assumes all inputs carry both a video and audio signal (e.g. game consoles). Inputs that only use one signal type — like a CD player (audio only) — are not explicitly supported and would require some tweaks to handle correctly. If you have a need for this and want to contribute, feel free to raise a PR.
