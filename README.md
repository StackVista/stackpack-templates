# StackPack Templates

A collection of templates for scaffolding StackPack projects for SUSE Observability. This repository provides ready-to-use templates that can be used with the `sts stackpack scaffold` command to quickly create new StackPack projects.

## What are StackPacks?

StackPacks are extensions for SUSE Observability that provide automated integrations with external systems. They enable you to:
- Add custom monitoring, alerting, and visualization capabilities

## Quick Start

### Prerequisites
- SUSE Observability CLI (`sts`) installed and configured
- Access to a SUSE Observability instance

### Create a New StackPack

Use the `sts stackpack scaffold` command to create a new StackPack from these templates:

```bash
# Create a new StackPack using the generic template
sts stackpack scaffold --name my-awesome-stackpack

# Use this repository as the template source
sts stackpack scaffold --name my-stackpack --template-github-repo your-org/stackpack-templates

# Use a local copy of this repository
sts stackpack scaffold --name my-stackpack --template-local-dir ./stackpack-templates/templates
```

This will create a new directory with your StackPack project structure, ready for customization.

## Available Templates

### Generic Template (`templates/generic/`)

The generic template provides a foundational StackPack structure with:

**🔍 Monitoring**
- **Node Memory Pressure Monitor**: Detects memory pressure conditions on Kubernetes nodes
- Pre-configured with threshold-based alerting and remediation guidance

**📊 Metrics**  
- **Memory Available for Scheduling**: Chart showing allocatable memory per node
- Custom metric binding with line chart visualization

**📝 Documentation**
- Complete project README with customization guide
- Overview and configuration documentation templates
- Logo placeholder for branding

**⚙️ Configuration**
- Ready-to-use `stackpack.conf` with HOCON format
- Provisioning templates using `.sty` files
- Template variables with `<< .Name >>` placeholders

## Repository Structure

```
stackpack-templates/
├── README.md                           # This file
├── templates/                          # Template directory
│   └── generic/                        # Generic StackPack template
│       ├── README.md                   # Template documentation
│       ├── stackpack.conf              # StackPack configuration
│       ├── provisioning/               # Provisioning definitions
│       │   ├── stackpack.sty          # Main provisioning template
│       │   ├── monitor.sty            # Monitor definitions  
│       │   └── metricbindings.sty     # Metric binding definitions
│       └── resources/                  # Documentation and assets
│           ├── overview.md            # StackPack overview
│           ├── default.md             # Configuration instructions
│           ├── *.md                   # Other documentation files
│           └── logo.png               # StackPack logo
└── sts-scaffold.md                     # STS scaffold command documentation
```

## Template Features

### Template Variables
All templates support variable substitution during scaffolding:
- `<< .Name >>`: Your StackPack short name (from `--name` flag)
- `<< .DisplayName >>`: Your StackPack User Friendly name (from `--display-name` flag)


## Development Workflow

### 1. Scaffold Your StackPack
```bash
sts stackpack scaffold --name my-awesome-stackpack -display-name "My Awesome StackPack"
cd my-awesome-stackpack
```

### 2. Customize Your StackPack
- Edit `stackpack.conf` with your integration details
- Modify monitors in `provisioning/monitor.sty`
- Update metric bindings in `provisioning/metricbindings.sty`
- Replace documentation in `resources/`

### 3. Validate and Package
```bash
# Validate and Package your StackPack
...
```

### 4. Upload to SUSE Observability
```bash
# Upload to SUSE Observability
sts stackpack upload my-awesome-stackpack-1.0.0.sts
```

### 4. Install and Test
Install your StackPack on a SUSE Observability instance either in Stackpack UI or using the CLI:
