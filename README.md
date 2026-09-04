# StackPack Templates

A collection of templates for scaffolding StackPack projects for SUSE Observability. This repository provides ready-to-use templates that can be used with the `sts stackpack scaffold` command to quickly create new StackPack projects.

## What are StackPacks?

StackPacks are extensions for SUSE Observability that provide automated integrations with external systems. They enable you to:
- Add custom monitoring, alerting, and visualization capabilities

## Quick Start

### Prerequisites
- SUSE Observability CLI (`sts`) installed and configured
- Access to a SUSE Observability instance
- For the webshop example: the **OpenTelemetry StackPack** installed and receiving
  data. The generic template is a *specialization layer* on top of it — it extends
  the generic `otel service` / `otel service instance` components (merging with
  them and inheriting their span metrics) rather than observing the app from
  scratch.

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

The generic template is a **worked tutorial** covering one runnable example of
every concept needed to build a custom integration. Most of it is a single
connected **webshop** example (built as a specialization layer on top of the
OpenTelemetry StackPack); a standalone **Dashboard** example ships alongside it
and can be kept or deleted independently.

**🧩 Component Presentation (the main tutorial)**
- A domain-specific example modeled on the [OpenTelemetry Demo](https://github.com/open-telemetry/opentelemetry-demo) webshop, built as a **specialization layer on top of the OpenTelemetry StackPack**: it takes the generic `otel service` / `otel service instance` components produced from OTel telemetry and **extends** them with a tailored view. Two component types — **microservices** (every demo service **instance**, with richer presentations for cart and checkout) and a **database** (kafka, at the **service** level) — connected by real dependencies (checkout instance → cart instance from the service graph; consumer instance → kafka service from consumer spans). Microservices **merge with** the generic service instances and **inherit** their span metrics; only domain-specific metrics are added. The database lists its connecting instances via related resources.
- Covers the full StackPacks 2.0 presentation model: component mappings, relation mappings, `ComponentPresentation` (columns, highlight, summary/perspective metrics, projections, filters), `MainMenuGroup`, and icons
- Every `.sty` file is commented and cross-linked to the reference documentation

**NOTE**
The template has been validated to work with opentelemetry-demo Helm chart version `0.41.0`.  Other versions may not be compatible.

**🔍 Monitoring**
- **Add-to-cart latency monitor**: a threshold `Monitor` on the cart's own domain metric — goes Deviating when the 95th-percentile add-to-cart latency exceeds 50&nbsp;ms, bound to the cart service instance
- Threshold-based alerting with a `!include` remediation hint (domain metric → monitor → health)

**📊 Metrics**  
- Metrics are defined inside a `ComponentPresentation` (there is no separate `MetricBinding` resource in StackPacks 2.0)
- The webshop presentations show metrics in every UI location: summary, highlight sections, and the metric perspective

**📈 Dashboard**
- **Pod resources**: A basic dashboard showing how to include dashboards in a stackpack
- An easy way to include dashboards is to first create it in the UI, then copy the yaml into the `dashboard` field of the Dashboard (it also needs a name, identifier and optional description). 

**📝 Documentation**
- Complete project README with customization guide
- Overview and configuration documentation templates
- Logo placeholder for branding

**⚙️ Configuration**
- Ready-to-use `stackpack.yaml`
- Provisioning templates in `settings/` using `.sty` files
- Reusable template fragments in `includes/`
- Template variables with `<< .Name >>` placeholders

## Repository Structure

```
stackpack-templates/
├── README.md                           # This file
└── templates/                          # Template directory
    └── generic/                        # Generic StackPack template
        ├── README.md                   # Template documentation
        ├── stackpack.yaml              # StackPack configuration
        ├── icons/                      # SVG icons referenced via !icon
        │   ├── microservice.svg
        │   └── database.svg
        ├── settings/                   # Provisioning definitions (imported on install)
        │   ├── main-menu.sty           # MainMenuGroup (left-hand navigation)
        │   ├── component-mappings/     # OtelComponentMapping (telemetry -> components)
        │   │   ├── microservices.sty   # demo service instances (except kafka) -> microservice
        │   │   └── database.sty        # kafka service -> database
        │   ├── relation-mappings/      # OtelRelationMapping (edges between components)
        │   │   ├── checkout-to-cart.sty      # service-graph metric edge
        │   │   └── consumer-to-database.sty  # consumer-span messaging edge
        │   ├── presentations/          # ComponentPresentation (how components look)
        │   │   ├── microservice.sty    # Base for all microservice instances (composed by rank)
        │   │   ├── cart.sty            # Rich, cart-specific (the full tour)
        │   │   ├── checkout.sty        # Rich, checkout-specific
        │   │   └── database.sty        # Second-type presentation
        │   ├── monitors.sty            # Monitor on the cart's domain metric
        │   └── dashboard.sty           # Dashboard (standalone)
        ├── includes/                   # Reusable template fragments
        │   └── remediation-hints/      # Monitor remediation hints
        │       └── cart-add-item-latency.md.hbs
        └── resources/                  # Documentation and assets
            ├── overview.md             # StackPack overview
            ├── provisioning.md         # Shown while provisioning
            ├── deprovisioning.md       # Shown while deprovisioning
            ├── installed.md            # Shown when installed
            ├── notinstalled.md         # Shown when not installed
            ├── waitingfordata.md       # Shown while waiting for data
            ├── error.md                # Shown on error
            └── logo.png                # StackPack logo
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
- Map your telemetry to components in `settings/component-mappings/`
- Shape the UI (columns, highlight, metrics) in `settings/presentations/`
- Modify monitors in `settings/monitors.sty`
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
