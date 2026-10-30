# Binary Signal Lamp

A single-page teaching tool that turns binary into a blinking lamp. Type any string of 0s and 1s and watch it sent as light, using one of three encodings:

- **Level**: lamp on = 1, lamp off = 0.
- **Manchester**: every bit has a change in the middle (IEEE 802.3 and G.E. Thomas conventions).
- **8B/10B**: each byte becomes a 10-bit code group with balanced on/off time (optionally starting with a K28.5 sync character).

It is one static HTML file with no dependencies, no build step and no network requests, so it works on GitHub Pages, any static host, or straight from your disk.

## Using it

1. Type binary into the input field (only `0` and `1`; spaces are ignored), or click a preset.
2. Pick a **Byte coding** (None or 8B/10B) and a **Line coding** (Level or one of the two Manchester conventions). They combine freely.
3. Press **Play** (or Enter). Use **Bit time** to slow things down and **Repeat** to loop.

While it plays you will see the lamp, a waveform with a moving cursor, the bits being sent (grouped into code groups for 8B/10B), and, for 8B/10B, a table showing how each byte was encoded. The "How does this work?" panel on the page explains the ideas in plain language.

## The encodings

### Level
A high level (lamp on) is 1 and a low level (lamp off) is 0. Long runs of identical bits produce no changes, so a receiver cannot easily count them.

### Manchester
Each bit is split into two halves and the signal changes in the middle of every bit, which gives the receiver a built-in clock ("self-clocking"). The cost is that the signal changes twice as often for the same data.

| Convention | 0 | 1 |
|---|---|---|
| IEEE 802.3 | on, then off | off, then on |
| G.E. Thomas | off, then on | on, then off |

### 8B/10B
Each byte is split as `HGF EDCBA` (3 bits and 5 bits). The 5 bits map to a 6-bit code and the 3 bits to a 4-bit code, giving a 10-bit group `abcdei fghj` that is sent left to right. The code is named `Dx.y`, where `x` is the value of the 5 bits and `y` the value of the 3 bits. For example `01001000` is `010` and `01000`, so `x = 8`, `y = 2`, giving `D8.2`.

The codes are chosen so that:

- there are never more than 5 identical bits in a row, and
- over time the lamp is on about as long as it is off (DC balance).

**Running disparity (RD)** is a tally of whether the stream so far has had one extra 0 (RD−) or one extra 1 (RD+). Many bytes have two possible codes; the encoder picks the one that steers RD back toward balance. The stream starts at RD−. With **Repeat** on, each round starts at the RD the previous round ended on, as a real continuous link would.

The optional **K28.5** character is the "comma" used for synchronisation in links such as Gigabit Ethernet and Fibre Channel. Only K28.5 is implemented, not the other control characters.

Input that is not a multiple of 8 bits is padded with zeros in the last byte.

### Checked
The encoder tables were checked for correct disparity in both RD columns, for known codes (D0.0, D3.1, D17.7, D21.5, D28.5), and for a maximum run length of 5 over random data.

## Publish on GitHub Pages

**In the browser**

1. Create a new repository on GitHub (public, unless you have a plan that allows private Pages).
2. Upload the contents of this folder to the repository root: `index.html`, `README.md`, `LICENSE` and `.nojekyll`. Use **Add file → Upload files**. Hidden files such as `.nojekyll` may not show up in your file manager, but the page works without it.
3. Go to **Settings → Pages**. Under **Build and deployment**, set **Source** to **Deploy from a branch**, choose the `main` branch and the `/ (root)` folder, then save.
4. After a minute or two the site is live at `https://<your-username>.github.io/<repository-name>/`.

**From the command line**

```bash
cd binary-signal-lamp
git init -b main
git add .
git commit -m "Add binary signal lamp"
git remote add origin https://github.com/<your-username>/<repository-name>.git
git push -u origin main
```

Then enable Pages as in step 3 above.

## Run locally

Open `index.html` in any modern browser. No server is needed. If you prefer one:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

## Files

| File | Purpose |
|---|---|
| `index.html` | The whole app: HTML, CSS and JavaScript |
| `README.md` | This file |
| `LICENSE` | MIT license |
| `.nojekyll` | Tells GitHub Pages to serve the files as they are |

## Limitations

- It encodes only; there is no decoder or "receiver" view.
- Only the K28.5 control character is available.
- Very fast playback can be limited by how quickly your browser updates the page. For classroom projection, keep Bit time at 300 ms or more.
- The lamp flashes on and off, which can be uncomfortable for people with photosensitivity. Keep the speed low if that is a concern.

## License

MIT. See `LICENSE`.
