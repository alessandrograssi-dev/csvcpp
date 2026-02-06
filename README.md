# csvcpp

**Modern C++ wrapper around a legacy C CSV parsing library**

csvcpp is a modern C++17 library that wraps and modernizes a legacy C CSV parser (libcsv),
providing a **safe, exception-based, type-aware C++ interface** while preserving the original
parser’s behavior and correctness.

This project is designed both as a usable CSV parsing/writing library and as a demonstration
of **incremental modernization of legacy C code**, with strong attention to correctness,
API design, and test parity.

---

## Key Features

- **Modern C++17 API over a legacy C core**
  - The original C implementation is vendored and left untouched.
  - A clean C++ wrapper exposes RAII semantics and a safer interface.

- **Exception-based error handling**
  - Errors are reported via `CsvError` exceptions instead of raw error codes.
  - Parsing errors include semantic error types and byte-position information.

- **Incremental / streaming parsing**
  - Supports chunked input and streaming via callbacks.
  - Preserves the original libcsv parsing model.

- **Flexible option configuration**
  - Parser options can be provided using either:
    - `std::initializer_list<Option>` for static configuration
    - `std::vector<Option>` for dynamic/runtime configuration

- **CSV writing support**
  - Functions for escaping and writing CSV fields, with optional custom quote characters.

- **Legacy test suite preserved**
  - The original C test suite has been adapted to exercise the C++ wrapper.
  - Guarantees behavioral equivalence with the legacy parser.

- **CMake-based build**
  - Clear project structure and modern CMake usage for library, tests, and examples.

---

## Project Structure

```
csvcpp/
├── legacy/             # Original C CSV implementation (vendored)
├── csvcpp/             # Modern C++ wrapper
├── examples/           # Example usage
├── tests/              # Adapted legacy test suite
├── CMakeLists.txt      # Top-level CMake configuration
└── README.md
```

---

## Design Rationale

This project deliberately avoids rewriting the CSV parser from scratch.
Instead, it demonstrates how to:

- safely encapsulate legacy C code using modern C++ idioms,
- preserve correctness through strict test compatibility,
- improve ergonomics and error handling without altering core behavior.

This mirrors common real-world engineering scenarios, where stability and
backward compatibility are more important than replacing proven components.

---

## API Highlights

### RAII and Safety

Parser resources are fully owned and released automatically:

```cpp
csv::CsvParser parser;
// no manual cleanup required
```

### Flexible Options

```cpp
parser.set_options({
    csv::CsvParser::Option::Strict,
    csv::CsvParser::Option::EmptyIsNull
});
```

or dynamically:

```cpp
std::vector<csv::CsvParser::Option> opts = {...};
parser.set_options(opts);
```

### Rich Errors

Parsing failures throw `CsvError`, exposing both semantic error type and
how many bytes were successfully parsed before failure.

---

## Example

```cpp
#include "CsvParser.hpp"

int main() {
    csv::CsvParser parser;
    parser.set_options({csv::CsvParser::Option::Strict});

    parser.parse(data, size, field_cb, row_cb, context);
    parser.finish(field_cb, row_cb, context);
}
```

More examples are available in the `examples/` directory.

---

## Building and Testing

The project uses CMake (≥ 3.16):

```bash
mkdir build && cd build
cmake ..
cmake --build .
ctest --output-on-failure
```

This builds the library, runs the adapted legacy tests, and compiles examples.

---

## What This Project Demonstrates

From a reviewer or recruiter perspective, this project highlights:

- Safe modernization of legacy C code
- Careful API design and usability trade-offs
- Strong testing discipline and backward compatibility
- Clean project structure and build configuration
- Attention to real-world constraints over toy implementations

---

## Future Improvements

Planned enhancements include:

- Installation targets (`install(TARGETS ...)`)
- Packaging support (e.g., Conan / vcpkg)
- Additional C++-friendly overloads
- Documentation and API reference generation

These can be added incrementally without breaking existing behavior.