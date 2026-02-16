# << .Name >> StackPack

This StackPack was created from the generic template and includes examples for monitoring and metricbinding resources for SUSE Observability.

## What's Included

This StackPack template comes up with examples for:
- **Monitor Definition**: A copy of `Node Memory Pressure monitor` for Kubernetes nodes
- **Metric Binding**: A copy of MetricBinding for `Memory available for scheduling` for Kubernetes nodes
- **Basic Configuration**: Ready-to-customize stackpack.conf

## Project Structure

```
<< .Name >>/
├── README.md
├── settings
│   ├── metricbindings.sty
│   ├── monitor.sty
│   └── stackpack.sty
├── resources
│   ├── deprovisioning.md
│   ├── error.md
│   ├── installed.md
│   ├── logo.png
│   ├── notinstalled.md
│   ├── overview.md
│   ├── provisioning.md
│   └── waitingfordata.md
└── stackpack.yaml

```


## Understanding the Provisioning System

This StackPack uses SUSE Observability's provisioning system to deploy resources. The key components are:

### Settings folder

The `settings/` folder contains all settings that are automatically imported.

### Adding Your Own Resources

To add additional resources to your StackPack:

1. **Create a new .sty file** in the `settings/` directory (e.g., `components.sty`, `checks.sty`, `templates.sty`)

2. **Define your resources** using YAML format in the new file:
```yaml
nodes:
- _type: Component
  name: My Custom Component
  identifier: urn:stackpack:<< .Name >>:component:my-component
  # ... other component properties

- _type: CheckTemplate
  name: My Custom Check
  identifier: urn:stackpack:<< .Name >>:check:my-check
  # ... other check properties
```

### Supported Resource Types

You can include various SUSE Observability resource types:
- **Monitors**: Health checks and alerting rules
- **MetricBindings**: Custom charts and visualizations
- **And more**: more resources are coming soon!

### Extract large values 

Large values can be extracted to their own file instead of being embedded in a settings yaml.  These values live as files under the `includes/` folder.
They can be included by using the `!include` yaml tag:

```yaml
nodes:
- _type: Monitor
  remediationHint: !include "remediation-hints/node-memory-pressure.md.hbs"
  # ... other monitor properties
```

## Customization Guide

### 1. Update StackPack Configuration

Edit `stackpack.yaml` to customize:

```yaml
name: "<< .Name >>"                    # Update with your StackPack name
displayName: "<< .Name >>"             # Update display name
categories: [ "Test" ]                 # Update categories
version: "0.0.1"                       # Set appropriate version
```

### 2. Customize Monitors

Edit `settings/monitor.sty` or create additional monitor files:

```yaml
nodes:
- _type: Monitor
  name: My Custom Monitor
  identifier: urn:stackpack:<< .Name >>:shared:monitor:my-custom-monitor
  # ... monitor configuration
```

### 3. Customize Metric Bindings

Edit `settings/metricbindings.sty` or create additional metric binding files:

```yaml
nodes:
- _type: MetricBinding
  name: My Custom Chart
  identifier: urn:stackpack:<< .Name >>:shared:metric-binding:my-chart
  # ... metric binding configuration
```

### 4. Add Additional Resources

Create new `.sty` files for other resources in the `settings/` folder.

### 5. Update Documentation

- **`resources/*.md`**: Customize messages given to the user during different StackPack states

## Development Workflow

### 1. Modify Your StackPack

### 2. Validate Your StackPack

### 3. Package the StackPack

### 4. Upload to the test Instance of SUSE Observability
