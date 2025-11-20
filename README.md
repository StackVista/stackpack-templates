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

**📈 Dashboard**
- **Pod resources**: A basic dashboard showing how to include dashboards in a stackpack
- An easy way to include dashboards is to first create it in the UI, then copy the yaml into the `dashboard` field of the Dashboard (it also needs a name, identifier and optional description). 

**📝 Documentation**
- Complete project README with customization guide
- Overview and configuration documentation templates
- Logo placeholder for branding

**⚙️ Configuration**
- Ready-to-use `stackpack.yaml`
- Provisioning templates using `.sty` files
- Template variables with `<< .Name >>` placeholders

## Repository Structure

```
stackpack-templates/
├── README.md                           # This file
├── templates/                          # Template directory
│   └── generic/                        # Generic StackPack template
│       ├── README.md                   # Template documentation
│       ├── stackpack.yaml              # StackPack configuration
│       ├── provisioning/               # Provisioning definitions
│       │   ├── stackpack.sty           # Main provisioning template
│       │   ├── monitor.sty             # Monitor definitions  
│       │   ├── dashboard.sty           # Dashboard definitions
│       │   └── metricbindings.sty      # Metric binding definitions
│       └── resources/                  # Documentation and assets
│           ├── overview.md             # StackPack overview
│           ├── default.md              # Configuration instructions
│           ├── *.md                    # Other documentation files
│           └── logo.png                # StackPack logo
└── sts-scaffold.md                     # STS scaffold command documentation
```

## Template Features

### Template Variables
All templates support variable substitution during scaffolding:
- `<< .Name >>`: Your StackPack short name (from `--name` flag)
- `<< .DisplayName >>`: Your StackPack User Friendly name (from `--display-name` flag)


## Development Workflow
Use the `--help` option on the CLI commands to discover all available options.

### 1. Scaffold your Stackpack
```bash
sts stackpack scaffold --name my-awesome-stackpack -display-name "My Awesome StackPack"
cd my-awesome-stackpack
```

### 2. Customize your Stackpack
- Edit `stackpack.yaml` with your integration details
- Modify monitors in `provisioning/monitor.sty`
- Update metric bindings in `provisioning/metricbindings.sty`
- Replace documentation in `resources/`

### 3. Test your Stackpack
Test the stackpack against on SUSE Observability:

```bash
sts stackpack test
```

## Releasing 

### Package your Stackpack
Update the version in the stackpack.yaml file to the desired version. Then run:

```bash
sts stackpack package
...
```

Now you can share the generated `.sts` file of your stackpack and make it available in 
SUSE Observability by uploading it:

```bash
# Upload to SUSE Observability
sts stackpack upload my-awesome-stackpack-1.0.0.sts
```
