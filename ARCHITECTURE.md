# Project Architecture

## Directory Responsibilities

### `legacy/`
**Purpose**: Original libcsv C implementation (vendored, read-only)

- **DO NOT MODIFY** - preserved for correctness verification
- Compiled as `libcsv` static library
- Original test suite remains for reference
- Acts as ground truth for behavioral verification

**Contents**:
- `libcsv.c` - CSV parser implementation
- `csv.h` - C API header
- Original documentation and license

### `csvcpp/`
**Purpose**: Modern C++ wrapper implementation

#### `csvcpp/include/`
**Public C++ API headers**
- `CsvParser.hpp` - main parser interface
- Zero dependencies on legacy headers in public API (encapsulated via pimpl)
- Exception-based error handling with `CsvError`
- C++17 features: RAII, smart pointers, initializer lists

#### `csvcpp/src/`
**Implementation details**
- `CsvParser.cpp` - bridges C++ API to C implementation
- Uses pimpl idiom to hide C structures from public interface
- Wraps `libcsv` C functions with exception translation
- Maintains thin wrapper philosophy (zero overhead abstraction)
- No additional allocations are introduced in the wrapper layer

### `tests/`
**Purpose**: Behavioral parity verification

- C tests adapted to exercise C++ wrapper
- **Must maintain output parity with legacy/tests/**
- Verification method: `diff <(legacy/test) <(tests/test)`
- Ensures wrapper correctness through comprehensive legacy test coverage

**Key principle**: If a test passed in C, it must pass identically in C++

### `examples/`
**Purpose**: Demonstrate modern usage patterns

Modern C++ implementations showcasing best practices:
- `csvtest.cpp` - basic streaming parse with callbacks
- `csvinfo.cpp` - file statistics and field counting
- `csvfix.cpp` - malformed CSV repair with RAII file handling
- `csvvalid.cpp` - strict validation with error position reporting

**Design characteristics**:
- RAII for resource management
- Exception-based error handling
- Professional code structure suitable for production use

### `csv_examples/`
**Purpose**: Test data corpus

Sample CSV files for testing and validation:
- Well-formed CSVs (simple, quoted, multiline)
- Malformed CSVs (unescaped quotes, invalid structure)
- Edge cases (empty files, single values, irregular rows)

---

## Build Targets

### Libraries

| Target | Purpose | Links Against | Output |
|--------|---------|---------------|--------|
| `libcsv` | Legacy C library | - | `libcsv.a` |
| `libcsvcpp` | C++ wrapper | `libcsv` | `libcsvcpp.a` |

### Executables

| Target | Purpose | Links Against | Location |
|--------|---------|---------------|----------|
| `test_csv` | Test suite | `libcsvcpp` | `build/tests/` |
| `csvtest` | Streaming parser | `libcsvcpp` | `build/examples/` |
| `csvinfo` | File statistics | `libcsvcpp` | `build/examples/` |
| `csvfix` | CSV repair tool | `libcsvcpp` | `build/examples/` |
| `csvvalid` | Validation tool | `libcsvcpp` | `build/examples/` |
---

## Design Principles

### 1. **Legacy Preservation**
The original C code is **never modified**. This ensures:
- Behavioral correctness is preserved
- We can always reference the original implementation
- Test comparison remains valid

### 2. **Test Parity**
C++ tests must produce **byte-for-byte identical output** to C tests:
```bash
diff <(./legacy/test_csv) <(./build/tests/test_csv)
# No output = perfect parity
```

This is more valuable than writing new tests because it proves the wrapper is transparent.

### 3. **Safety Through Modern C++**
The wrapper adds safety without changing behavior:
- **RAII**: Automatic resource cleanup
- **Exceptions**: Clear error propagation vs error codes
- **Type safety**: Strong typing, no void* in public API
- **Smart pointers**: No manual memory management

### 4. **Zero Overhead Abstraction**
The wrapper is intentionally thin:
- Direct delegation to C functions
- No buffering or transformation in wrapper
- Inlining opportunities for release builds
- Performance matches C implementation

### 5. **Incremental Modernization**
This project demonstrates real-world refactoring:
- Don't rewrite working code
- Wrap first, refactor later (if needed)
- Maintain backward compatibility
- Preserve battle-tested logic

---

## API Layers

### Layer 1: Legacy C (libcsv)
```c
#include <csv.h>
struct csv_parser p;
csv_init(&p, 0);
csv_parse(&p, data, len, cb1, cb2, NULL);
csv_free(&p);
```

**Characteristics**: Error codes, manual memory management, void* for context

### Layer 2: C++ Wrapper (libcsvcpp)
```cpp
#include "CsvParser.hpp"
csv::CsvParser p;
p.parse(data, len, cb1, cb2, nullptr);
// RAII cleanup automatic
```

**Characteristics**: Exceptions, RAII, type safety, modern C++ idioms

### Layer 3: Application Code
Uses the C++ wrapper with high-level patterns:
- File RAII with `std::unique_ptr<FILE, deleter>`
- Exception handling with `try-catch`
- Standard library containers
- Modern control flow

---

## Error Handling Strategy

### C Layer (legacy/)
```c
int result = csv_parse(&p, data, len, cb1, cb2, NULL);
if (result != len) {
    int error_code = csv_error(&p);
    const char* msg = csv_strerror(error_code);
    // Handle error manually
}
```

### C++ Layer (csvcpp/)
```cpp
try {
    parser.parse(data, len, cb1, cb2, context);
} catch (const csv::CsvError& e) {
    // e.type - error type enum
    // e.bytes_parsed - position of error
    // e.what() - human-readable message
}
```

**Translation**: C error codes → C++ exceptions with rich context

---

## Testing Strategy

### Unit Tests (tests/)
**Adapted from original C test suite**
- Ensures behavioral equivalence
- Validates all parser options
- Tests edge cases (empty input, large buffers, etc.)
- Verification: output must match C version exactly

### Integration Tests (examples/)
**Real-world usage patterns**
- File I/O with error handling
- Multi-file processing
- Streaming parse with callbacks
- Error recovery and reporting

### Test Parity Verification
```bash
# Run both test suites
make run-tests           # C++ wrapper tests
cd legacy && make test   # Original C tests

# Compare outputs programmatically
diff <(./legacy/test_csv) <(./build/tests/test_csv)

# Exit code 0 = perfect parity
```

---

## Build System

### CMake Structure
```
CMakeLists.txt              # Root configuration
├── legacy/CMakeLists.txt   # Build libcsv.a
├── csvcpp/CMakeLists.txt   # Build libcsvcpp.a (links libcsv)
├── examples/CMakeLists.txt # Build example programs
└── tests/CMakeLists.txt    # Build test suite
```


---

## Future Enhancements

### Planned
1. **Installation targets** - install support
2. **Package management** - Conan/vcpkg integration
3. **Documentation** - Doxygen API reference
4. **CI/CD** - GitHub Actions for test parity verification
5. **Benchmarks** - Performance comparison C vs C++ wrapper

### Under Consideration
1. **Additional overloads** - std::string_view support
2. **Range-based API** - C++20 ranges for parsing

All enhancements must maintain backward compatibility and test parity.

---

## Key Metrics

**What makes this project notable:**

✅ **100% test parity** - All legacy tests pass with identical output  
✅ **Zero modifications** to legacy code - Original implementation untouched  
✅ **Modern C++17** - RAII, exceptions, smart pointers, strong typing  
✅ **Documented rationale** - Clear architectural decisions and trade-offs  
