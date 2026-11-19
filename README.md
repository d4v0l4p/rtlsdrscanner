# RTL-SDR Scanner

A simple command-line spectrum scanner for RTL-SDR dongles. It sweeps a frequency range step by step, measures the signal power at each step, highlights frequencies that stand out above the noise floor, and plots the result.

![Python](https://img.shields.io/badge/python-3.8%2B-blue)
![Platform](https://img.shields.io/badge/platform-Linux%20%7C%20macOS%20%7C%20Windows-lightgrey)

## Features

- Interactive prompts for start frequency, stop frequency and step size, with sensible defaults
- Sweeps the full default RTL-SDR tuning range (24 MHz to 1766 MHz)
- Progress bar while scanning
- Automatic detection of high-power frequencies (anything above mean + 1 standard deviation)
- Power-vs-frequency plot with matplotlib

## Requirements

### Hardware

- An RTL-SDR USB dongle (RTL2832U-based, e.g. with an R820T/R820T2 tuner)
- An antenna suitable for the range you want to scan

### Software

- Python 3.8 or newer
- The `librtlsdr` driver library
- These Python packages:
  - `numpy`
  - `matplotlib`
  - `pyrtlsdr`
  - `tqdm`

## Installation

### 1. Install the RTL-SDR driver

**Debian / Ubuntu**

```bash
sudo apt install rtl-sdr librtlsdr-dev
```

**macOS (Homebrew)**

```bash
brew install librtlsdr
```

**Windows**

Use [Zadig](https://zadig.akeo.ie/) to install the WinUSB driver for the dongle, and make sure `librtlsdr.dll` is on your `PATH`.

### 2. Clone the repository

```bash
git clone https://github.com/d4v0l4p/rtlsdrscanner.git
cd rtlsdrscanner
```

### 3. Install the Python dependencies

```bash
pip install numpy matplotlib pyrtlsdr tqdm
```

Or, if you add a `requirements.txt`:

```bash
pip install -r requirements.txt
```

## Usage

Plug in your RTL-SDR dongle and run:

```bash
python scanner.py
```

You will be prompted for the scan parameters. Press **Enter** to accept a default:

```
Enter start frequency in MHz (default 24):
Enter stop frequency in MHz (default 1766):
Enter step size in kHz (default 200):
```

### Example: scan the FM broadcast band

```
Enter start frequency in MHz (default 24): 88
Enter stop frequency in MHz (default 1766): 108
Enter step size in kHz (default 200): 100
```

When the sweep finishes, the scanner prints the frequencies it flagged and opens a plot:

```
High-power frequencies detected at (MHz): [ 89.1  94.5  98.8 101.3 ]
```

## How it works

1. The dongle is set to a 2.4 MHz sample rate with automatic gain.
2. For each frequency step, the tuner is moved to that centre frequency and 131,072 IQ samples are read.
3. An FFT of the samples gives the power spectrum, and its mean is converted to decibels.
4. After the sweep, any step whose power is more than one standard deviation above the mean is reported as a high-power frequency.
5. The power levels are plotted against frequency.

## Notes and limitations

- **Power values are relative**, not calibrated dBm. They are good for spotting where activity is, not for absolute measurements.
- **Automatic gain** can shift between steps, which may make some readings inconsistent. Setting a fixed gain gives more comparable results.
- **Full-range scans take time.** The default sweep (24–1766 MHz at 200 kHz) is about 8,700 steps.
- The frequency range your dongle actually supports depends on its tuner chip.

## Troubleshooting

**`usb_claim_interface error -6` (Linux)**
The kernel's DVB-T driver has claimed the device. Unload it and blacklist it:

```bash
sudo rmmod dvb_usb_rtl28xxu
echo 'blacklist dvb_usb_rtl28xxu' | sudo tee /etc/modprobe.d/blacklist-rtl.conf
```

**`LibUSBError` or "No devices found"**
Check that the dongle is plugged in and that `rtl_test` can see it. On Linux you may need udev rules (included with the `rtl-sdr` package) or to run as a user in the `plugdev` group.

**`OSError: Error loading librtlsdr`**
The driver library is not installed or not on your library path. See the installation steps above.

## Author

**Tim Digga**
GitHub: [@timdigga](https://github.com/timdigga)

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.
