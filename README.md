# Firmware Installer PRMZNG

**Codename: SKIBIDI**

A standalone payload for Nintendo Switch that installs system firmware directly from NCA files on your SD card - no PC tools required.

## Why does this exist?

Daybreak (the standard firmware installer) requires booting into Horizon OS first. But if your firmware is corrupted, missing, or you're in a soft-brick state, you can't boot HOS to run Daybreak.

This tool solves the chicken-and-egg problem: **install firmware without having working firmware**.

## Why should this NOT exist?

Writing raw BIS and SYSTEM partition data is **extremely dangerous**. Nintendo made firmware installation complex for good reason - one wrong byte can permanently brick your console.

This tool exists because sometimes you have no other choice. **Use at your own risk.** 
Actually you have plenty of choices, but the systemic flow can get too heavy

## Features

- **Standalone Operation** - Runs entirely on the Switch, no PC needed after initial setup
- **Key Derivation** - Automatically derives encryption keys from device fuses or loads from `prod.keys`
- **emuMMC & sysMMC Support** - Install to emuMMC (recommended) or sysMMC
- **Safety First** - Multiple confirmation prompts, button combos, battery checks, and backup options
- **Generate-Only Mode** - Create BIS files without installing for verification against EmmcHaccGen
- **Progress Tracking** - Visual progress bar showing per-file copy status
- **Automatic Backups** - Backs up existing BOOT0/BOOT1 and SYSTEM contents before writing
- **EmmcHaccGen Compatible** - Generates `boot.bis` files in the same format as EmmcHaccGen

## Requirements

- Nintendo Switch with custom firmware capability (Erista or Mariko)
- SD card with firmware NCA files in `sd:/firmware/` folder
- For Mariko: `prod.keys` 

## Installation

1. Download `FirmwareInstaller.bin` from releases
2. Place it on your SD card (e.g., `sd:/bootloader/payloads/`)
3. Copy your firmware NCA files to `sd:/firmware/`
4. Boot the payload using your preferred method (Hekate, fusee, TegraRcmGUI, etc.)

## Usage

### Main Menu

| Option | Description |
|--------|-------------|
| **Scan Firmware Folder** | Scans `sd:/firmware/` for NCA files and detects firmware version |
| **View Detected Firmware** | Shows details about scanned firmware |
| **Generate Files Only** | Creates BOOT0, BOOT1, boot.bis, and NCAs to SD without installing |
| **Install to emuMMC** | Installs firmware to emuMMC partition (safer for testing) |
| **Install to sysMMC** | Installs firmware to sysMMC (DANGEROUS - can brick!) |
| **View Derived Keys** | Shows derived encryption keys |
| **About** | Project information and credits |
| **Reboot to payload.bin** | Chainloads `sd:/payload.bin` |
| **Power Off** | Shuts down the console |

### Installation Options

All safety options are **ON by default**:

| Option | Default | Description |
|--------|---------|-------------|
| exFAT + FAT32 support | ON | Include exFAT driver NCAs (disable only if SD is FAT32) |
| Save generated BIS to SD | ON | Saves BOOT0/BOOT1/boot.bis before writing |
| Save NCAs to SD | ON | Copies NCAs to backup folder before writing |
| Backup current BIS | ON | Backs up existing BOOT0/BOOT1 from eMMC |
| Backup current SYSTEM | ON | Backs up existing NCAs from SYSTEM partition (~2GB, slow) |

### Safety Features

1. **Button Combo Challenge** - Must enter a button sequence before installation
   - emuMMC: VOL+ VOL- POWER
   - sysMMC: VOL+ VOL+ VOL- VOL- VOL+ VOL- POWER
2. **Battery Check** - Requires 50% battery or charger connected
3. **5-Second Countdown** - Final chance to cancel before writing
4. **Warning Dialogs** - Extra confirmations when disabling safety options

### Output Paths

All generated and backup files are organized by eMMC serial number:

```
sd:/skibidi/
  XXXXXXXX/                    # eMMC serial number (8 hex chars)
    generated_XX.X.X/          # Generated files for firmware version
      BOOT0.bin
      BOOT1.bin
      BCPKG2_1.bin ... BCPKG2_6.bin
      boot.bis                 # Combined file (EmmcHaccGen format)
      ncas/                    # Copied NCA files
        *.nca
    backup/                    # Backups of existing data
      bis/
        BOOT0_emummc.bin
        BOOT1_emummc.bin
      registered/              # Backed up NCAs from SYSTEM
        *.nca
      save/
        8000000000000120       # NCM save file backup
```

## Generate-Only Mode

The "Generate Files Only" option creates all firmware files on the SD card without writing to eMMC. This allows you to:

1. Verify the generated files match EmmcHaccGen's output
2. Use the files with other tools (NxNandManager, etc.)
3. Test the generation process safely

Generated files are placed in `sd:/skibidi/XXXXXXXX/generated_XX.X.X/`


| Partition | Source | Description |
|-----------|--------|-------------|
| BOOT0 | Generated | BCT + PK1 + SECMON + WARMBOOT |
| BOOT1 | Generated | Mirror of BOOT0 data |
| SYSTEM | Copied | NCAs to `/Contents/registered/` |

### NCA Processing

- NCAs are decrypted using AES-XTS (header) and AES-CTR (content)
- Header key derived from device or loaded from prod.keys
- Title keys not required - firmware NCAs use standard crypto

### boot.bis Format

The `boot.bis` file follows EmmcHaccGen's format:
- 16 bytes: Version string (null-padded)
- 1 byte: Flags (bit 0 = exFAT, bit 1 = AutoRCM, bit 2 = Mariko)
- 4x4 bytes: Sizes (BOOT0, BOOT1, BCPKG2_1-3, BCPKG2_4-6)
- Data: BOOT0 + BOOT1 + BCPKG2 partitions

## Troubleshooting

### "NCA headers could not be decrypted"
- Mariko users: Ensure `sd:/switch/prod.keys` contains valid keys
- Erista users: Key derivation may have failed, try providing prod.keys

### "Missing: Boot FAT NCA"
- Firmware folder is incomplete
- Ensure all NCA files from the firmware package are in `sd:/firmware/`

### "Failed to mount SD card"
- SD card may be corrupted or not inserted properly
- Try reinserting the SD card

## Credits

everyone