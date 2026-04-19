# Node.js Runtime — Agent Guide

Node.js is an open-source, cross-platform JavaScript runtime built on V8 and libuv. This repo contains the **full C/C++ and JavaScript source** for the runtime itself — not an application built on Node.js.

---

## Prerequisites

### macOS
- **Xcode Command Line Tools >= 13** (installs `clang`, `clang++`, `make`):
  ```bash
  xcode-select --install
  ```
- **Python >= 3** (required for `./configure` and test tooling):
  ```bash
  brew install python3
  ```

### Linux
- `gcc` and `g++` >= 10.1, GNU Make >= 3.81, Python >= 3
  ```bash
  # Ubuntu / Debian
  sudo apt-get install python3 g++ make python3-pip

  # Fedora
  sudo dnf install python3 gcc-c++ make python3-pip

  # CentOS / RHEL
  sudo yum install python3 gcc-c++ make python3-pip

  # Arch Linux
  sudo pacman -S python gcc make python-pip
  ```

### Windows
- Visual Studio 2022 with the **Desktop development with C++** workload and Windows 10 SDK
- Python >= 3 (from python.org)
- Use `vcbuild.bat` instead of `make` (see below)

---

## Configuring the Build

```bash
./configure
```

Common options:
| Flag | Purpose |
|------|---------|
| `--debug` | Enable debug symbols (produces both `out/Release/node` and `out/Debug/node`) |
| `--coverage` | Instrument for code coverage |
| `--openssl-no-asm` | Skip OpenSSL asm optimisations (required if assembler is too old) |
| `--ninja` | Generate Ninja build files instead of Makefiles |
| `--prefix=<path>` | Installation prefix (default: `/usr/local`) |
| `--with-intl=full-icu` | Build with full ICU (all locales) |
| `--with-intl=small-icu` | Build with English-only ICU (default) |
| `--without-intl` | Disable Intl / ICU entirely |

> **Gotcha:** If the path to your build directory contains a space the build will likely fail. Keep the repo in a path with no spaces.

---

## Compiling Node.js from Source

```bash
# Standard build — adjust -j to the number of CPU cores you want to use
./configure
make -j4
```

After a successful build the binary lives at `./node` (symlink) and `out/Release/node`.

### Verify the build
```bash
./node -e "console.log('Node.js ' + process.version + ' built OK')"
```

### Faster iteration with ccache
```bash
CC="ccache gcc" CXX="ccache g++" ./configure
make -j4
```

### Ninja (faster incremental builds)
```bash
./configure --ninja
ninja -C out/Release -j4
```

### Windows
```bat
vcbuild.bat release
```

### Installing
```bash
[sudo] make install
```

---

## Running the Test Suite

```bash
# Quick: run only the tests (no linting)
make test-only

# Full CI-style check (tests + linting + doc tests) — use before submitting a PR
make -j4 test
```

> `make -j4 test` also runs linters. Failures here will block PRs.

### Lint only
```bash
make lint           # JS, C++, and Markdown
```

---

## Running Specific Tests

### Single test file
```bash
# Using the test runner
tools/test.py test/parallel/test-stream2-transform.js

# Or directly with your local build
./node test/parallel/test-stream2-transform.js
```

### Entire subsystem
```bash
tools/test.py child-process
tools/test.py crypto
```

### Entire suite in a directory
```bash
tools/test.py test/message
```

### Options
```bash
tools/test.py --help
```

> On Windows use `python3 tools/test.py ...`.

---

## Coverage

```bash
./configure --coverage
make coverage
# Reports: coverage/index.html (JS), coverage/cxxcoverage.html (C++)
```

JavaScript-only coverage without recompiling:
```bash
make coverage-run-js
```

---

## Key Directories

| Directory | Contents |
|-----------|---------|
| `src/` | Core runtime in C/C++ — event loop glue, V8 bindings, built-in modules |
| `lib/` | Standard library implemented in JavaScript (`fs`, `http`, `stream`, etc.) |
| `test/` | Test suite — `test/parallel/` is the main bucket; also `test/sequential/`, `test/message/`, `test/async-hooks/`, etc. |
| `deps/` | Vendored dependencies: V8, libuv, OpenSSL, npm, llhttp, nghttp2, ICU, and more |
| `tools/` | Build scripts, linters, `tools/test.py` test runner, CI helpers |
| `doc/` | API documentation source (Markdown) |
| `benchmark/` | Performance benchmarks |
| `out/` | Build artefacts (created by `make`): `out/Release/node`, `out/Debug/node` |

---

## Common Gotchas

1. **Space in path** — The build system breaks if the repo path contains a space. Move the repo to a space-free path.

2. **Python version** — `./configure` requires Python **3**. Make sure `python3` (or `python`) resolves to Python 3, not Python 2.

3. **Old assembler (OpenSSL)** — If you get OpenSSL asm errors, pass `--openssl-no-asm` to `./configure`.

4. **Recompile after changing `lib/` or `src/`** — The `./node` symlink only updates after `make -j4`. Always rebuild before re-running tests when you change C++ or built-in JS files.

5. **IPv6 loopback on Ubuntu** — Some tests require `::1` to be available on the loopback interface:
   ```bash
   sudo sysctl -w net.ipv6.conf.lo.disable_ipv6=0
   ```

6. **macOS firewall popups** — Suppress network-permission popups during tests:
   ```bash
   sudo ./tools/macos-firewall.sh
   ```

7. **`out/` is not cleaned by default** — Run `make clean` to wipe the build output before a fresh `./configure` if you see strange linker errors.

8. **Debug vs Release** — `./configure --debug` produces **both** `out/Release/node` and `out/Debug/node`. The `./node` symlink always points to the Release binary.

---

## Useful Make Targets

| Target | Description |
|--------|-------------|
| `make -j4` | Compile (release) |
| `make test-only` | Run tests without linting |
| `make -j4 test` | Full test + lint (CI mode) |
| `make lint` | Run linters only |
| `make doc` | Build HTML documentation |
| `make docserve` | Serve docs in a browser |
| `make coverage` | Build with coverage and run tests |
| `make clean` | Remove build artefacts |
| `make install` | Install to system (or `--prefix`) |

---

## Further Reading

- [`BUILDING.md`](BUILDING.md) — comprehensive build guide including Windows, Android, ICU, FIPS
- [`CONTRIBUTING.md`](CONTRIBUTING.md) — contribution workflow and coding standards
- [`doc/contributing/`](doc/contributing/) — deeper guides (debugging, V8 upgrades, etc.)
