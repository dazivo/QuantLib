# CLAUDE.md

This file provides guidance for AI assistants working with the QuantLib codebase.

## Project Overview

QuantLib is a free/open-source library for quantitative finance written in C++, providing tools for modeling, trading, and risk management. It requires C++11 or later and depends on Boost (1.58.0+).

## Build Instructions

### CMake (Recommended)

```bash
mkdir build && cd build
cmake ..
cmake --build . -j$(nproc)
```

Common CMake options:
```bash
cmake -DBOOST_ROOT=/path/to/boost \
      -DQL_BUILD_TEST_SUITE=ON \
      -DQL_BUILD_EXAMPLES=OFF \
      ..
```

Predefined presets are available via `cmake --list-presets`.

### Autoconf (Alternative)

```bash
./autogen.sh
./configure --disable-static
make -j$(nproc)
```

## Running Tests

```bash
# After CMake build:
./build/test-suite/quantlib-test-suite --log_level=message

# Run a specific test:
./build/test-suite/quantlib-test-suite --log_level=message --run_test=SomeTestSuite
```

The test suite uses Boost.Test and lives in `test-suite/`.

## Code Style

Format with clang-format using the provided `.clang-format` config:
```bash
clang-format -i path/to/file.cpp
```

Key style rules:
- **Indentation**: 4 spaces, no tabs
- **Line length**: 100 characters max
- **References/pointers**: Left-aligned (`T& x`, `T* p`)
- **Namespace indentation**: Indent contents of all namespaces
- **Include order**: QuantLib headers → Boost → standard library

### File Header Template

Every source file should start with:
```cpp
/* -*- mode: c++; tab-width: 4; indent-tabs-mode: nil; c-basic-offset: 4 -*- */

/*
 Copyright (C) <YEAR> <Author/Organization>

 This file is part of QuantLib, a free-software/open-source library
 for financial quantitative analysts and developers - http://quantlib.org/

 QuantLib is free software: you can redistribute it and/or modify it
 under the terms of the QuantLib license. You should have received a
 copy of the license along with this program; if not, please email
 <quantlib-dev@lists.sf.net>. The license is also available online at
 <http://quantlib.org/license.shtml>.

 This program is distributed in the hope that it will be useful, but WITHOUT
 ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS
 FOR A PARTICULAR PURPOSE. See the license for more details.
*/

/*! \file filename.hpp
    \brief One-line description.
*/

#ifndef quantlib_filename_hpp
#define quantlib_filename_hpp

// ... includes and code ...

#endif
```

### Include Guards

Use the pattern `quantlib_<filename>_hpp` (all lowercase, underscores for separators).

### Error Handling

Use QuantLib macros instead of throwing exceptions directly:
```cpp
QL_REQUIRE(condition, "Error message with " << variable);
QL_ENSURE(condition, "Post-condition failed");
QL_FAIL("Unreachable or unimplemented: " << details);
```

## Repository Structure

```
ql/                     # Library source and headers
  cashflows/            # Cash flow instruments
  currencies/           # Currency definitions
  experimental/         # Experimental (less stable) features
  indexes/              # Market indexes
  instruments/          # Financial instruments
  math/                 # Mathematical utilities
  methods/              # Numerical methods (finite diff, Monte Carlo, etc.)
  models/               # Financial models (HJM, LMM, short-rate, etc.)
  patterns/             # Design patterns (Observer, Singleton, etc.)
  pricingengines/       # Option and instrument pricing engines
  processes/            # Stochastic processes
  quotes/               # Market quote handles
  termstructures/       # Yield curves, vol surfaces, default curves
  time/                 # Date/calendar/day-count utilities
  utilities/            # Miscellaneous helpers
test-suite/             # Boost.Test test suite
Examples/               # Example programs
Docs/                   # Doxygen documentation config
cmake/                  # CMake find-modules and helpers
.github/workflows/      # CI pipelines
```

## Architecture Notes

- **Lazy evaluation**: Instruments and term structures compute values on demand and cache results until market data changes.
- **Observer/Observable pattern**: Market data objects (quotes, term structures) notify dependent instruments when they change. Use `registerWith()` / `notifyObservers()`.
- **Handle<T>**: A shared, relinkable pointer used extensively for market data. Prefer `Handle<Quote>` over raw pointers for inputs.
- **Smart pointers**: `ext::shared_ptr<T>` is the standard (maps to `std::shared_ptr` when `QL_USE_STD_SHARED_PTR` is set, otherwise `boost::shared_ptr`).
- **Settings singleton**: `Settings::instance()` holds the global evaluation date and other session-wide settings.

## Contributing

See `CONTRIBUTING.md` for the full contribution workflow. In summary:
1. Fork the repository on GitHub.
2. Create a feature branch.
3. Follow the coding conventions above.
4. Add tests in `test-suite/` for new functionality.
5. Open a pull request against the `master` branch.

Check CI results — the project tests against many compiler/Boost combinations (see `.github/workflows/`).
