# Build from Source

This guide is for people who want to compile the firmware themselves — for example, to pick up a newer version of esp-claw, or to make changes to the board config.

If you just want to flash the pre-built binary, see the main [README](README.md).

---

## Requirements

- macOS or Linux (not tested on Windows)
- Git
- Python 3.10 or newer
- ESP-IDF **v5.5.4** — this exact version is required. The official esp-claw patch does not apply to IDF 6.0 without modification (the patch in `patches/` has already been trimmed to fix this, but the build system still requires 5.5.4 toolchain versions)

---

## Step 1 — Install ESP-IDF 5.5.4

```bash
mkdir -p ~/esp
git clone --depth=1 --branch v5.5.4 https://github.com/espressif/esp-idf.git ~/esp/esp-idf-v5.5.4
```

Install the tools for ESP32-S3 only (faster than installing all targets):

```bash
IDF_PATH="$HOME/esp/esp-idf-v5.5.4" \
IDF_PYTHON_ENV_PATH="$HOME/.espressif/python_env/idf5.5_py3.13_env" \
~/esp/esp-idf-v5.5.4/install.sh esp32s3
```

Install the board manager helper:

```bash
~/.espressif/python_env/idf5.5_py3.13_env/bin/pip install esp-bmgr-assist
```

---

## Step 2 — Clone esp-claw

```bash
git clone https://github.com/espressif/esp-claw.git ~/Documents/Development/espclaw
cd ~/Documents/Development/espclaw/application/basic_demo
```

---

## Step 3 — Copy in the N16R8 board config

```bash
cp -r /path/to/this/repo/boards/esp32_S3_N16R8 ./boards/
```

---

## Step 4 — Apply the IDF patch

The IDF patch in this repo strips the USB header hunk (which was removed in IDF 6.0) and keeps only the LCD parallel panel fix. Apply it to your IDF 5.5.4 installation:

```bash
git -C ~/esp/esp-idf-v5.5.4 apply -C1 /path/to/this/repo/patches/esp-idf-idf6.0-compat.patch
```

---

## Step 5 — Generate the board config

```bash
export IDF_PATH="$HOME/esp/esp-idf-v5.5.4"
export IDF_PYTHON_ENV_PATH="$HOME/.espressif/python_env/idf5.5_py3.13_env"
source ~/esp/esp-idf-v5.5.4/export.sh

idf.py gen-bmgr-config -c ./boards -b esp32_S3_N16R8
```

---

## Step 6 — Build

```bash
idf.py build
```

The first build downloads several git submodules (WiFi, lwip, mbedtls, etc.) and takes 15–30 minutes depending on your internet connection. Subsequent builds take 3–5 minutes.

If the build fails with a submodule error, pre-fetch the slow ones manually with `--depth=1`:

```bash
cd ~/esp/esp-idf-v5.5.4
git submodule update --init --depth=1 -- components/esp_wifi/lib
git submodule update --init --depth=1 -- components/lwip/lwip components/mbedtls/mbedtls components/json/cJSON
```

Then re-run `idf.py build`.

---

## Step 7 — Flash

```bash
idf.py -p /dev/tty.usbmodem* flash
```

Replace `/dev/tty.usbmodem*` with your board's actual serial port.

---

## Board config details

The key settings in `boards/esp32_S3_N16R8/sdkconfig.defaults.board` that make this board work correctly:

```
CONFIG_ESP_DEFAULT_CPU_FREQ_MHZ_240=y
CONFIG_ESPTOOLPY_FLASHMODE_QIO=y
CONFIG_ESPTOOLPY_FLASHFREQ_80M=y
CONFIG_ESPTOOLPY_FLASHSIZE_16MB=y
CONFIG_SPIRAM=y
CONFIG_SPIRAM_MODE_OCT=y
CONFIG_SPIRAM_SPEED_80M=y
CONFIG_PARTITION_TABLE_CUSTOM_FILENAME="partitions_16MB.csv"
```

The timing issue that causes the board to get stuck after flashing a generic build is caused by `SPIRAM_MODE_OCT` not being set, or by the flash frequency mismatch. This config fixes both.
