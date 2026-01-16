# DDEX Proto

[![Go Reference](https://pkg.go.dev/badge/github.com/alecsavvy/ddex-proto.svg)](https://pkg.go.dev/github.com/alecsavvy/ddex-proto)
[![Go Report Card](https://goreportcard.com/badge/github.com/alecsavvy/ddex-proto?style=flat&v=1)](https://goreportcard.com/report/github.com/alecsavvy/ddex-proto?style=flat&v=1)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![CI](https://github.com/alecsavvy/ddex-proto/workflows/CI/badge.svg)](https://github.com/alecsavvy/ddex-proto/actions)

Go library for working with DDEX (Digital Data Exchange) standards. Provides XML, JSON, and Protocol Buffer serialization for DDEX message types.

## What is DDEX?

DDEX is a consortium of media companies, music licensing organizations, digital service providers, and technical intermediaries that develop standards for exchanging information and rights data along the digital supply chain.

## Features

Go structs with Protocol Buffer, JSON, and XML serialization for:

- **ERN** (Electronic Release Notification) - v3.8.1, v3.8.3, v4.2, v4.3, v4.3.2
- **MEAD v1.1** (Media Enrichment and Description)
- **PIE v1.0** (Party Identification and Enrichment)

All message types support:
- XML marshal/unmarshal with DDEX XSD compliance
- Protocol Buffer binary serialization
- JSON serialization
- Bidirectional conversion between formats

## Installation

```bash
go get github.com/alecsavvy/ddex-proto@latest
```

## Quick Start

### Parsing DDEX XML

```go
package main

import (
    "encoding/xml"
    "fmt"
    "os"

    ernv432 "github.com/alecsavvy/ddex-proto/gen/ddex/ern/v432"
)

func main() {
    xmlData, _ := os.ReadFile("release.xml")

    var release ernv432.NewReleaseMessage
    xml.Unmarshal(xmlData, &release)

    fmt.Printf("Message ID: %s\n", release.MessageHeader.MessageId)
}
```

### Format Conversion

```go
package main

import (
    "encoding/json"
    "encoding/xml"
    ernv432 "github.com/alecsavvy/ddex-proto/gen/ddex/ern/v432"
    "google.golang.org/protobuf/proto"
)

func main() {
    release := &ernv432.NewReleaseMessage{
        MessageHeader: &ernv432.MessageHeader{
            MessageId: "MSG-12345",
        },
    }

    // Serialize to different formats
    protoData, _ := proto.Marshal(release)     // Protocol Buffer
    jsonData, _ := json.Marshal(release)       // JSON
    xmlData, _ := xml.MarshalIndent(release, "", "  ")  // XML
}
```

## Command Line Tools

Four CLI tools are included for code generation and DDEX file processing:

### xsd2proto

Converts DDEX XSD schemas to Protocol Buffer definitions.

```bash
go run ./cmd/xsd2proto
```

Reads XSD files from the `xsd/` directory and generates `.proto` files in `proto/`.

### protoc-go-inject-tag

Injects XML struct tags into generated Go code to enable XML serialization.

```bash
go run ./cmd/protoc-go-inject-tag -input="gen/**/*.pb.go"
```

### ddex-gen

Generates DDEX-specific Go extensions for protobuf-generated files:
- `enum_strings.go` - String conversion methods for enums
- `*.xml.go` - XML marshaling with namespace support
- `registry.go` - Message type registry

```bash
go run ./cmd/ddex-gen ./gen
```

### protoc-gen-ddex

All-in-one tool that runs both tag injection and extension generation.

```bash
# Generate .pb.go files from buf registry
buf generate

# Post-process to add XML support
go run ./cmd/protoc-gen-ddex ./gen
```

Options:
- `--dir <path>` - Target directory (default: `./gen`)
- `--verbose` - Enable verbose logging

### Installing Tools

Install globally with:

```bash
make install-tools
```

Or individually:

```bash
go install github.com/alecsavvy/ddex-proto/cmd/xsd2proto@latest
go install github.com/alecsavvy/ddex-proto/cmd/protoc-gen-ddex@latest
go install github.com/alecsavvy/ddex-proto/cmd/ddex-gen@latest
go install github.com/alecsavvy/ddex-proto/cmd/protoc-go-inject-tag@latest
```

## Go Packages

### Main Package

The main `ddex` package provides type aliases and helper functions:

```go
import "github.com/alecsavvy/ddex-proto"

// Type aliases for all message types
type NewReleaseMessageV432 = ernv432.NewReleaseMessage
type MeadMessageV11 = meadv11.MeadMessage
// ... etc

// Helper functions for parsing
message, version, err := ddex.ParseERN(xmlData)  // Auto-detect version
version, err := ddex.DetectERNVersion(xmlData)   // Get version only
```

### pkg/ddexgen

Library for generating DDEX-specific code extensions. Used by the `ddex-gen` CLI tool but can also be imported directly:

```go
import "github.com/alecsavvy/ddex-proto/pkg/ddexgen"

err := ddexgen.Generate("./gen", true) // directory, verbose
```

See [pkg/ddexgen/README.md](pkg/ddexgen/README.md) for details.

### pkg/injecttag

Library for injecting struct tags into Go source files. Used by `protoc-go-inject-tag` but can also be imported:

```go
import "github.com/alecsavvy/ddex-proto/pkg/injecttag"

areas, err := injecttag.ParseFile("generated.pb.go", src, nil)
err = injecttag.WriteFile("generated.pb.go", areas, false)
```

See [pkg/injecttag/README.md](pkg/injecttag/README.md) for details.

## Examples

### Testing DDEX Files

The `examples/proto` directory contains a parser for testing DDEX files:

```bash
# Parse any DDEX XML file
go run examples/proto/main.go -file path/to/ddex-file.xml

# With output to verify roundtrip
go run examples/proto/main.go -file input.xml -output output.xml
```

The example automatically detects the message type (ERN, MEAD, PIE) and version, then parses and displays the contents using `spew.Dump()`.

Test files are provided in `testdata/ddex/` covering all supported versions.

## Supported Message Types

### ERN (Electronic Release Notification)

All ERN versions support `NewReleaseMessage` and `PurgeReleaseMessage`.

ERN v3.8.1 and v3.8.3 also support `CatalogListMessage`.

| Version | Import Path |
|---------|-------------|
| v4.3.2  | `github.com/alecsavvy/ddex-proto/gen/ddex/ern/v432` |
| v4.3    | `github.com/alecsavvy/ddex-proto/gen/ddex/ern/v43` |
| v4.2    | `github.com/alecsavvy/ddex-proto/gen/ddex/ern/v42` |
| v3.8.3  | `github.com/alecsavvy/ddex-proto/gen/ddex/ern/v383` |
| v3.8.1  | `github.com/alecsavvy/ddex-proto/gen/ddex/ern/v381` |

### MEAD (Media Enrichment and Description) v1.1

```go
import meadv11 "github.com/alecsavvy/ddex-proto/gen/ddex/mead/v11"
// Types: MeadMessage
```

### PIE (Party Identification and Enrichment) v1.0

```go
import piev10 "github.com/alecsavvy/ddex-proto/gen/ddex/pie/v10"
// Types: PieMessage, PieRequestMessage
```

## Development

### Running Tests

```bash
make test                # All tests
make test-comprehensive  # Conformance, roundtrip, completeness tests
make benchmark          # Performance benchmarks
```

Test coverage includes:
- Conformance tests against official DDEX sample files
- Roundtrip tests (XML ↔ protobuf conversion)
- Field completeness validation
- Performance benchmarks

### Code Generation

Regenerate code from XSD schemas:

```bash
make generate           # Complete pipeline: XSD → proto → Go
make generate-proto     # XSD → proto only
make generate-proto-go  # proto → Go only
```

Or use individual commands:

```bash
make generate-proto     # Run xsd2proto
make buf-generate       # Run buf + protoc-gen-ddex
make generate-ddex      # Run protoc-gen-ddex only
```

See `make help` for all available commands.

### Generation Pipeline

1. **XSD → Proto**: `xsd2proto` converts DDEX XSD schemas to `.proto` files
2. **Proto → Go**: `buf generate` creates Go structs with protobuf support
3. **XML Tag Injection**: `protoc-go-inject-tag` adds XML tags for DDEX compliance
4. **Go Extensions**: `ddex-gen` generates enum strings, XML methods, and registry

Or use `protoc-gen-ddex` to combine steps 3 and 4.

## Buf Schema Registry

The Protocol Buffer schemas are published on the Buf Schema Registry:

```
buf.build/alecsavvy/ddex
```

### Using in Your Project

Add to your `buf.yaml`:

```yaml
version: v2
deps:
  - buf.build/alecsavvy/ddex
```

Import in `.proto` files:

```protobuf
syntax = "proto3";
package myservice;

import "ddex/ern/v432/new_release_message.proto";
import "ddex/avs/avs.proto";

message MyReleaseWrapper {
  ddex.ern.v432.NewReleaseMessage release = 1;
  ddex.avs.TerritoryCode territory = 2;
}
```

### Note on Generated Code

Code generated directly from the Buf registry will have Protocol Buffer support but won't include the XML struct tags needed for DDEX XML serialization. For full XML support, use the Go module (`go get github.com/alecsavvy/ddex-proto`) or clone the repository and run the generation pipeline with `protoc-gen-ddex`.

## Repository Structure

```
ddex-proto/
├── cmd/                     # Command-line tools
│   ├── xsd2proto/          # XSD to proto converter
│   ├── protoc-go-inject-tag/  # XML tag injector
│   ├── ddex-gen/           # DDEX extensions generator
│   └── protoc-gen-ddex/    # All-in-one generation tool
│
├── pkg/                     # Go libraries
│   ├── ddexgen/            # DDEX code generation library
│   └── injecttag/          # Struct tag injection library
│
├── gen/                     # Generated Go code
│   └── ddex/
│       ├── avs/            # Shared enum types
│       ├── ern/            # ERN Go code (all versions)
│       ├── mead/v11/       # MEAD Go code
│       └── pie/v10/        # PIE Go code
│
├── proto/                   # Protocol Buffer definitions
│   └── ddex/
│       ├── avs/            # Shared enums (proto)
│       ├── ern/            # ERN proto files
│       ├── mead/v11/       # MEAD proto files
│       └── pie/v10/        # PIE proto files
│
├── xsd/                     # Original DDEX XSD schemas
│   ├── avs*.xsd            # Allowed Value Sets
│   ├── ernv*/              # ERN XSD files by version
│   ├── mead/               # MEAD XSD files
│   └── pie/                # PIE XSD files
│
├── examples/                # Usage examples
│   └── proto/              # DDEX file parser example
│
├── testdata/                # Test files
│   └── ddex/               # DDEX samples organized by type/version
│
├── ddex.go                 # Main package with type aliases and helpers
├── buf.yaml                # Buf configuration
├── buf.gen.yaml            # Code generation config
└── Makefile                # Build automation
```

## Use Cases

- **XML**: DDEX standard compliance, external integrations
- **JSON**: REST APIs, web services, JavaScript interoperability
- **Protocol Buffers**: Internal APIs, microservices, gRPC/ConnectRPC

Convert seamlessly between all three formats as needed.

## License

MIT License. This library is for working with DDEX standards developed by the DDEX consortium.
