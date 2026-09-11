# Recovering Final Fantasy XI Screenshots from a PS2 HDD

How I recovered ~20-year-old FFXI screenshots off a PlayStation 2 hard drive in 2026.
The method is dead simple: image the drive with `ddrescue`, then run `foremost` directly on the raw image. 
No filesystem parsing, no partition mapping, no proprietary tools required.

## TL;DR

```bash
# 1. Image the drive (USB-to-IDE adapter)
ddrescue /dev/sdX ps2-hdd.img ps2-hdd.map

# 2. Carve images straight out of the raw image
foremost -t jpg,bmp -i ps2-hdd.img -o recovered/
```

That's it. Details below.

## Hardware & Imaging

- PS2 HDD pulled and connected via a USB-to-IDE adapter.
- Imaged with **`ddrescue`** on Linux:

  ```bash
  ddrescue /dev/sdX ps2-hdd.img ps2-hdd.map
  ```

  `ddrescue` is preferred over `dd` because it retries bad sectors and is resumable via the mapfile.

- The resulting image is a raw byte-for-byte copy of the APA-formatted drive.

## Carving the Screenshots

The key insight: **you don't need to understand the filesystem to recover image files.** 
Screenshots are JPEGs (and possibly BMPs), and those have well-known headers. 
Point a carver at the raw image and let it find them.

```bash
sudo apt install foremost
foremost -t jpg,bmp -i /path/to/ps2-hdd.img -o test/
```

`foremost` scans the entire image and dumps recovered files into `test/jpg/` and `test/bmp/`. 
This worked — it found the old FFXI screenshots.

No need to compute partition offsets or extract specific regions. 
Running it on the whole image is fine.

## Notes on PS2 FFXI Screenshots

- PS2 FFXI screenshots were written by the game client to the HDD.
- Format is typically **JPEG**, sometimes reported as **BMP** depending on client version and region — hence carving for both.
- PS2 screenshots are often **512×448** or **640×448**.
- Some recovered JPEGs may have odd or missing EXIF data. They still open fine; thumbnailers may show them as black.
- PS2 FFXI screenshot support was far more limited than the PC version, so you may find fewer files than expected. They may also be mixed in with unrelated JPEGs recovered from elsewhere on the disk — sort by resolution and date to find the game ones.
