# Changelog

## [0.1.0] - 2026-02-06

### Added
- Modern C++17 wrapper around libcsv
- Exception-based error handling with CsvError
- RAII patterns with std::unique_ptr
- Test parity with original C implementation
- Four example programs (csvtest, csvinfo, csvfix, csvvalid)
- CMake build system

### Features
- Streaming CSV parsing with callbacks
- Flexible option configuration (initializer_list and vector)
- CSV writing with custom quote characters
- Tests for validation
