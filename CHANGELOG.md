# Changelog

All notable changes to this project will be documented here.

## [1.0.0] — 2026-04-27

### Added
- Initial release
- Pre-built firmware binaries for ESP32-S3 N16R8 (16MB flash, 8MB embedded octal PSRAM)
- Custom board configuration `esp32_S3_N16R8` with correct flash size, PSRAM mode (OCT), and partition table
- IDF-compatible patch (`patches/esp-idf-idf6.0-compat.patch`) — trims the USB IAD hunk that fails on IDF 6.0, retains the LCD parallel panel fix
- `README.md` with step-by-step flash and configuration guide for non-technical users
- `BUILD_FROM_SOURCE.md` for developers who want to rebuild with ESP-IDF 5.5.4
- Apache-2.0 license (matching upstream esp-claw)

### Known working
- Claude Sonnet 4.6 via OpenRouter
- GPT-5.5 via OpenRouter
- Telegram bot integration
- Tavily web search

### Known limitations
- Tested on ESP32-S3 N16R8 only — other ESP32-S3 variants are not guaranteed to work with this binary
- Built with ESP-IDF 5.5.4 toolchain
