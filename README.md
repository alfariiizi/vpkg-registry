# Vandor Package Registry (vpkg-registry)

The official package registry for the Vandor ecosystem, providing reusable components for Go projects using hexagonal architecture.

## 🏗️ Architecture

The vpkg system provides two types of packages:

- **`fx-module`** - Library modules that integrate with Uber FX dependency injection
- **`cli-command`** - Executable CLI tools that can be run via `vandor vpkg exec`

## 📦 Available Packages

### Core Packages (vandor namespace)

#### vandor/redis-cache
- **Type**: fx-module
- **Description**: Redis caching adapter using go-redis/v9 with Fx integration
- **Features**: Configuration management, lifecycle handling, common cache operations
- **Tags**: full-backend, eda, minimal, cache

#### vandor/audit-logger
- **Type**: fx-module  
- **Description**: Structured audit logging with context tracking
- **Features**: Structured logging, context propagation, audit trails
- **Tags**: full-backend, eda, logging

### Community Packages (acme namespace)

#### acme/migrate-db
- **Type**: cli-command
- **Description**: Database migration tool with Atlas integration
- **Features**: Migration management, status checking, rollback support
- **Tags**: full-backend, eda, database

## 🚀 Quick Start

### Install Vandor CLI
```bash
curl -fsSL https://raw.githubusercontent.com/alfariiizi/vandor-cli/main/install-vandor.sh | bash
```

### Browse Available Packages
```bash
# List all packages
vandor vpkg list

# Filter by type
vandor vpkg list --type fx-module
vandor vpkg list --type cli-command

# Filter by tags
vandor vpkg list --tags cache,full-backend
```

### Install Packages

#### fx-module Package (Library)
```bash
# Install Redis cache
vandor vpkg add vandor/redis-cache

# This creates files in: internal/vpkg/vandor/redis-cache/
# - redis.go (with RedisCache struct and Module)
# - README.md (usage instructions)
```

#### cli-command Package (Tool)
```bash
# Install migration tool
vandor vpkg add acme/migrate-db

# Run the tool
vandor vpkg exec acme/migrate-db status
vandor vpkg exec acme/migrate-db up
```

## 📁 Registry Structure

```
vpkg-registry/
├── registry.yaml                     # Package index (single source of truth)
├── packages/
│   ├── vandor/                        # Official Vandor packages
│   │   ├── redis-cache/
│   │   │   ├── meta.yaml             # Package metadata
│   │   │   └── templates/            # Template files
│   │   │       ├── redis.go.tmpl     # Main implementation
│   │   │       └── README.md.tmpl    # Usage documentation
│   │   └── audit-logger/
│   │       ├── meta.yaml
│   │       └── templates/
│   └── acme/                          # Community packages
│       └── migrate-db/
│           ├── meta.yaml
│           └── templates/
│               ├── cmd/main.go.tmpl   # CLI entry point
│               ├── migration.go.tmpl  # Core functionality
│               └── README.md.tmpl     # Documentation
└── docs/                              # Future documentation site
```

## 🔧 Package Development

### Creating a New Package

1. **Choose Namespace**: Use your organization/username (e.g., `myorg/package-name`)

2. **Create Directory Structure**:
```bash
mkdir -p packages/myorg/my-package/templates
```

3. **Create `meta.yaml`**:
```yaml
name: myorg/my-package
title: My Package
description: Brief description of your package
type: fx-module  # or cli-command
entry: main.go   # entry point for cli-command packages
templates:
  - templates/main.go.tmpl
  - templates/README.md.tmpl
destination: internal/vpkg/myorg/my-package
version: v0.1.0
license: MIT
author: Your Name
tags:
  - full-backend
  - eda
dependencies:
  - github.com/example/dependency
```

4. **Create Templates**:
   - Use `.tmpl` extension for files that need template rendering
   - Non-template files are copied as-is
   - Templates receive rich context (Module, Package, Version, etc.)

5. **Update Registry**: Add your package to `registry.yaml`

### Template Context

Templates receive the following context variables:

```go
type TemplateContext struct {
    Module      string // "github.com/user/project" (auto-detected)
    VpkgName    string // "myorg/my-package"
    Namespace   string // "myorg" 
    Pkg         string // "my-package"
    Package     string // "mypackage" (sanitized Go identifier)
    PackagePath string // "internal/vpkg/myorg/my-package"
    Version     string // "v0.1.0"
    Author      string // "Your Name"
    Time        string // RFC3339 timestamp
    Title       string // "My Package"
    Description string // Package description
}
```

### Template Helper Functions

Available in all templates:

- `{{ .Name | Title }}` - Title case
- `{{ .Name | Camel }}` - camelCase  
- `{{ .Name | Pascal }}` - PascalCase
- `{{ .Name | Snake }}` - snake_case
- `{{ .Name | Kebab }}` - kebab-case
- `{{ .Name | Upper }}` - UPPERCASE
- `{{ .Name | Lower }}` - lowercase
- `{{ .Name | GoIdent }}` - Valid Go identifier

### Example fx-module Template

```go
package {{ .Package }}

import (
	"go.uber.org/fx"
)

// {{ .Title }} provides {{ .Description | lower }}
type {{ .Title | Pascal }} struct {
	// Implementation here
}

// New{{ .Title | Pascal }} creates a new {{ .Title | lower }}
func New{{ .Title | Pascal }}() *{{ .Title | Pascal }} {
	return &{{ .Title | Pascal }}{}
}

// Module provides Fx module for dependency injection
var Module = fx.Module("{{ .Package }}",
	fx.Provide(New{{ .Title | Pascal }}),
)
```

### Example cli-command Template

```go
package main

import (
	"fmt"
	"os"
	
	"{{ .Module }}/{{ .PackagePath }}"
	"github.com/spf13/cobra"
)

var rootCmd = &cobra.Command{
	Use:   "{{ .Pkg }}",
	Short: "{{ .Description }}",
	Run: func(cmd *cobra.Command, args []string) {
		fmt.Println("{{ .Title }} v{{ .Version }}")
	},
}

func main() {
	if err := rootCmd.Execute(); err != nil {
		fmt.Println(err)
		os.Exit(1)
	}
}
```

## 🏷️ Package Tags

Use tags to indicate compatibility:

- **Architecture Types**:
  - `full-backend` - Complete backend with all features
  - `eda` - Event-driven architecture
  - `minimal` - Minimal lightweight setup

- **Functionality**:
  - `cache` - Caching related
  - `database` - Database related  
  - `logging` - Logging and audit
  - `auth` - Authentication/authorization
  - `http` - HTTP and API related
  - `messaging` - Message queues and events

## 📋 Package Guidelines

### Best Practices

1. **Follow Go Conventions**: Use proper Go naming and project structure
2. **Include Documentation**: Provide clear README templates with usage examples
3. **Version Properly**: Use semantic versioning (v1.0.0, v1.1.0, etc.)
4. **Test Templates**: Ensure templates generate valid, compilable Go code
5. **Minimal Dependencies**: Keep external dependencies to a minimum
6. **Configuration**: Use environment variables or config structs for settings

### fx-module Guidelines

- Export a `Module` variable for Fx integration
- Provide constructor functions (e.g., `NewRedisCache()`)
- Include proper lifecycle management (start/stop hooks)
- Use configuration structs with tags for environment variables

### cli-command Guidelines

- Include `cmd/main.go` as entry point
- Use Cobra for CLI structure
- Support `--help` and `--version` flags
- Handle errors gracefully with proper exit codes
- Include subcommands for complex functionality

## 🤝 Contributing

1. **Fork the Repository**: Create your own fork of vpkg-registry
2. **Create Package**: Add your package following the structure above
3. **Update Registry**: Add entry to `registry.yaml`
4. **Test Installation**: Verify your package installs and works correctly
5. **Submit PR**: Create a pull request with your changes

### Contribution Requirements

- Package must follow naming conventions (`namespace/package-name`)
- Templates must generate valid, formatted Go code
- Include comprehensive README template
- Specify correct dependencies and tags
- Add appropriate license information

## 🔍 Package Discovery

Users can discover packages through:

```bash
# Browse all packages
vandor vpkg list

# Search by keyword
vandor vpkg list | grep redis

# Filter by type and tags
vandor vpkg list --type fx-module --tags cache

# Get package details
vandor vpkg add package-name --dry-run
```

## 📚 Documentation

- **CLI Documentation**: See [vandor-cli README](../vandor-cli/README.md)
- **Architecture Guide**: See [vandor-backend README](../vandor-backend/README.md)
- **Package Development**: This README
- **Template Reference**: See example packages in `packages/` directory

## 🆘 Support

- **Issues**: [Report bugs or request packages](https://github.com/alfariiizi/vpkg-registry/issues)
- **Discussions**: [Community discussions](https://github.com/alfariiizi/vpkg-registry/discussions)
- **Documentation**: [Full documentation](https://docs.vandor.dev)

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

---

**Registry URL**: `https://raw.githubusercontent.com/alfariiizi/vpkg-registry/main/registry.yaml`

The Vandor Package Registry makes it easy to share and reuse components across Go projects using hexagonal architecture. Start building and sharing your packages today!