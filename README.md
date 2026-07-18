# Wickra Compile — Go

[![CI](https://github.com/wickra-lib/wickra-compile/actions/workflows/ci.yml/badge.svg)](https://github.com/wickra-lib/wickra-compile/actions/workflows/ci.yml)
[![codecov](https://codecov.io/gh/wickra-lib/wickra-compile/branch/main/graph/badge.svg)](https://codecov.io/gh/wickra-lib/wickra-compile)
[![Go module](https://raw.githubusercontent.com/wickra-lib/.github/main/profile/badges/go.svg)](https://pkg.go.dev/github.com/wickra-lib/wickra-compile-go)
[![License: MIT OR Apache-2.0](https://img.shields.io/badge/license-MIT_OR_Apache--2.0-blue)](https://github.com/wickra-lib/wickra-compile#license)

**Go bindings for the Wickra strategy compiler over its C ABI hub via cgo. A `Compiler` is driven over a JSON boundary, so the manifest it produces is byte-identical to every other Wickra Compile binding.**

## Install

Use the published **`wickra-compile-go`** module, which bundles the prebuilt C ABI
library for every platform, so `go get` + `go build` works with no extra steps
(a C compiler is still required, as the binding uses cgo):

```bash
go get github.com/wickra-lib/wickra-compile-go
```

```go
import wickra "github.com/wickra-lib/wickra-compile-go"
```

`wickra-compile-go` is generated from the [`bindings/go`](https://github.com/wickra-lib/wickra-compile/tree/main/bindings/go)
directory of [wickra-compile](https://github.com/wickra-lib/wickra-compile) by the release
pipeline: it mirrors the Go sources, the vendored C ABI header (`include/wickra_compile.h`)
and the prebuilt libraries under `lib/<goos>_<goarch>/`. On Linux/macOS the
library path is baked in via rpath; on Windows the DLL must be discoverable at
run time (next to the executable or on `PATH`).

### Building from this repository (contributors)

The `bindings/go` directory in the main repository is the development source. To
build against a locally compiled C ABI, build the hub and stage the library into
the per-platform directory cgo links against:

```bash
cargo build -p wickra-compile-c --release
mkdir -p lib/linux_amd64                          # match your GOOS_GOARCH
cp target/release/libwickra_compile.so    lib/linux_amd64/    # Linux
cp target/release/libwickra_compile.dylib lib/darwin_arm64/   # macOS (arm64)
cp target/release/wickra_compile.dll      lib/windows_amd64/  # Windows
```

## Quick start

```go
package main

import (
	"fmt"

	wickra "github.com/wickra-lib/wickra-compile-go"
)

func main() {
	c := wickra.New()
	defer c.Close()

	resp, _ := c.Command(`{"cmd":"compile","dry_run":true,"spec":{
		"strategy":{"symbol":"x","timeframe":"1h",
			"indicators":{"f":{"type":"Ema","params":[3]}},
			"entry":{"cross_above":["f","f"]},"exit":{"cross_below":["f","f"]},
			"sizing":{"type":"fixed_qty","qty":1}},
		"target":{"kind":"wasm"},"crate_name":"demo"}}`)
	fmt.Println(resp) // response JSON, including manifest.project_hash
}
```

## Documentation

The full guides, quickstarts and API reference live in the main repository and
documentation site:

- **Repository:** <https://github.com/wickra-lib/wickra-compile>
- **Docs:** <https://docs.wickra.org>

Wickra ships native bindings for Python, Node.js, WASM and Rust, plus a C ABI hub
that any C-capable language (C, C++, C#, Go, Java, R) links against — all exposing
the same core from the shared, `unsafe`-forbidden Rust core.

## Security

Found a security issue? **Please don't open a public issue.** Report it privately
via the affected repository's *Security* tab (*"Report a vulnerability"*) or email
**support@wickra.org** with a subject line starting `[wickra security]`. Full
policy: <https://github.com/wickra-lib/wickra-compile/blob/main/SECURITY.md>.

## Disclaimer

Wickra Compile is research and analytics software. Its outputs are
deterministic transforms of the input data — they are not financial advice and do
not predict the market. Any use in a live trading context is at your own risk. The
software is provided **as is**, without warranty of any kind.

## License

Licensed under either of [Apache-2.0](https://github.com/wickra-lib/wickra-compile/blob/main/LICENSE-APACHE)
or [MIT](https://github.com/wickra-lib/wickra-compile/blob/main/LICENSE-MIT) at your option.
