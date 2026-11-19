# rtlsdrscanner

Small command line scanner for RTL-SDR sticks. It steps through a frequency range, measures the power at each step and shows you where something is going on. At the end you get a list of the loudest frequencies and a plot.

## What you need

- An RTL-SDR stick (RTL2832U, e.g. with R820T/R820T2 tuner) and an antenna
- Python 3.8 or newer
- librtlsdr

Installing librtlsdr:

```bash
# Debian / Ubuntu
sudo apt install rtl-sdr librtlsdr-dev

# macOS
brew install librtlsdr
```

On Windows, install the WinUSB driver for the stick with [Zadig](https://zadig.akeo.ie/) and put `librtlsdr.dll` somewhere on your PATH.

## Setup

```bash
git clone https://github.com/d4v0l4p/rtlsdrscanner.git
cd rtlsdrscanner
pip install -r requirements.txt
```

## Usage

```bash
python rtl_sdr_freq_scanner.py
```

It asks for start frequency, stop frequency and step size. Just press Enter to use the defaults (24 MHz to 1766 MHz, 200 kHz steps).

Example for the FM band:

```
Enter start frequency in MHz (default 24): 88
Enter stop frequency in MHz (default 1766): 108
Enter step size in kHz (default 200): 100
```

When it's done it prints something like

```
High-power frequencies detected at (MHz): [ 89.1  94.5  98.8 101.3 ]
```

and opens a power over frequency plot.

## How it works

The stick runs at 2.4 MHz sample rate with auto gain. At every step it reads 131072 samples, does an FFT and takes the mean power in dB. Everything more than one standard deviation above the average of the whole sweep counts as "high power".

## Good to know

- The dB values are relative, not calibrated. Fine for finding activity, not for real measurements.
- Auto gain can change between steps, so readings are not always comparable.
- A full sweep with the defaults is around 8700 steps and takes a while. Narrow the range if you know what you're looking for.
- What range actually works depends on the tuner in your stick.

## Problems

`usb_claim_interface error -6` on Linux means the DVB-T kernel driver grabbed the stick:

```bash
sudo rmmod dvb_usb_rtl28xxu
echo 'blacklist dvb_usb_rtl28xxu' | sudo tee /etc/modprobe.d/blacklist-rtl.conf
```

"No devices found": check if `rtl_test` sees the stick. On Linux your user might need to be in the `plugdev` group.

`Error loading librtlsdr`: the library isn't installed or can't be found, see above.

## License

MIT, see [LICENSE](LICENSE).
