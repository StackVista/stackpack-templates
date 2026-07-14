# << .Name >> StackPack

This StackPack was created from the generic template. It is a **worked tutorial**:
it contains one runnable example of every concept you need to build a custom
integration for SUSE Observability — from turning raw telemetry into components,
to connecting them, to controlling exactly how they look in the UI.

> **This example is a *specialization layer* on top of the OpenTelemetry
> StackPack.** It does not re-observe the app from scratch; instead it takes the
> generic `otel service` / `otel service instance` components that the
> OpenTelemetry StackPack already produces from OTel telemetry and **extends**
> them with a domain-specific view — its own component types, menu, columns,
> relations, a domain metric and a monitor — while **merging with** and
> **inheriting from** the generic components (so the built-in span metrics come
> for free). This is the intended pattern for giving an app already observed via
> OpenTelemetry its own tailored representation.
>
> **Prerequisite:** the **OpenTelemetry StackPack must be installed** and
> receiving data. The mappings merge with its components (via
> `additionalIdentifiers`), the presentations inherit its metrics (their bindings
> are a subset of its bindings), and the relations read the OTel service graph and
> spans. Without it, the components would still be created but the merge, metric
> inheritance and relations would not resolve.

The examples are built around the [OpenTelemetry Demo](https://github.com/open-telemetry/opentelemetry-demo)
webshop: imagine a team running that shop who wants their own domain view. We model
two component **types** — `microservice` (the demo's services) and `database`
(the Kafka broker they use, treated here as a generic backing datastore) —
connected by two real dependencies: checkout → cart (a synchronous call, from the
service graph) and consumer → kafka (a messaging edge, read from consumer spans).
Change the service names and queries to match your own integration and you have a
head start.

Each component is a **service instance** (one running replica, keyed by
`service.instance.id`) rather than a whole service. Instances are the finest unit
OpenTelemetry reports, so every component drills into one replica's metrics —
and, crucially, relations are drawn instance-to-instance: Kafka's "Used by" panel
shows the specific consuming **instance**. Our instance components share their
identity with the OpenTelemetry StackPack's `otel service instance`, so they
**merge** into one component (rather than creating a duplicate) — this is the
specialization-on-top-of-OTel pattern the callout above describes.

> **Demo version:** this example was developed against **OpenTelemetry Demo
> v2.2.0**, where the services are named `cart`, `checkout`, `kafka`, etc. Older
> demo versions used `cartservice` / `checkoutservice`; if you run one of those,
> update the `service.name` values in `settings/presentations/cart.sty`,
> `checkout.sty` and the relation mappings accordingly.

## What's Included

Almost everything is one connected **webshop** example (component mappings,
relations, presentations, a domain metric and a monitor — all built on the
OpenTelemetry StackPack). A single standalone **Dashboard** ships alongside it and
can be kept or deleted independently.

**The webshop example — the main tutorial**
| File | Concept |
|------|---------|
| `settings/main-menu.sty` | `MainMenuGroup` — adds a section to the left-hand navigation |
| `settings/component-mappings/microservices.sty` | `OtelComponentMapping` — every demo service **instance** → `microservice` type |
| `settings/component-mappings/database.sty` | `OtelComponentMapping` — kafka **service** → `database` type |
| `settings/relation-mappings/checkout-to-cart.sty` | `OtelRelationMapping` — checkout → cart edge (instance-to-instance) |
| `settings/relation-mappings/consumer-to-database.sty` | `OtelRelationMapping` — consumer instance → kafka edge, read from consumer **spans** (messaging.system=kafka) |
| `settings/presentations/microservice.sty` | Base presentation for all microservice instances — shared list, columns & filters (`rank.specificity` composition) |
| `settings/presentations/cart.sty` | Rich `ComponentPresentation` for cart instances — the full tour, incl. a domain metric |
| `settings/presentations/checkout.sty` | Rich presentation for checkout instances — focuses on what it depends on |
| `settings/presentations/database.sty` | Second-type presentation for the database (service-level; lists connecting instances via related resources) |
| `settings/monitors.sty` | `Monitor` on the cart's **own domain metric** — Deviating when p95 add-to-cart latency > 50 ms, bound to the cart instance (domain metric → monitor → health), with an `!include` remediation hint |
| `icons/microservice.svg`, `database.svg` | Icons referenced via `!icon` |

The microservices mapping onboards **every** service instance in the demo (any
`service.namespace` containing `telemetry-demo` that carries a
`service.instance.id`). They all share the `microservice` type, so they appear
together in one **Microservices** list. The base presentation
(`microservice.sty`) covers every instance; `cart.sty` and `checkout.sty` bind to
their specific service (by the per-instance `service.name` tag) at a higher
specificity to add richer detail — while keeping the same list, so no extra menu
items appear. Kafka is modelled as a separate `database` type with its own
**Databases** list. Both lists live under the one menu group defined in
`settings/main-menu.sty`.

The database is deliberately modelled at the **service** level (one Kafka box,
not one per broker replica) — a shared backing store reads better as a single
component. Its callers, however, are **instances**. The two relations use two
different signals, which is itself instructive:

- **checkout → cart** comes from the **service-graph metric**
  (`traces_service_graph_request_total`) — the right source for synchronous
  request/response calls. It is instance-to-instance via the graph's
  `client_service.instance.id` / `server_service.instance.id`.
- **consumer → kafka** comes from **consumer spans**. Kafka is a messaging
  system, so it does *not* appear in the service graph; instead each consumer
  emits a span with `messaging.system = kafka`. The mapping reads that span to
  draw the consumer *instance* → kafka *service* edge. (It matches on
  `messaging.system` alone — the demo's consumer spans are not reliably tagged
  with `span.kind = CONSUMER`, so gating on that would match nothing.)

So Kafka's "Used by" panel surfaces the specific consuming **instances** — the
service-level database reaches instance-level detail one hop away through its
related resources rather than being an instance itself.

### Inheriting metrics by composition (no duplication)

Our microservice components deliberately rank **below** the generic OpenTelemetry
mapping, so their primary type stays `otel service instance` (our own type name
survives as the `source-type:<< .Name >> microservice` label). Each of our
presentations then **binds with the generic query as a prefix** — e.g.
`(label = "stackpack:open-telemetry" and type = "otel service instance" and
label = "service.name:cart")`. Because that is a strict subset of the generic
service-instance presentation's binding, presentation composition applies the
generic **span metrics** (rate / errors / duration) to our components for free.
We therefore define **only** our own domain metrics (e.g. the cart's
`add-item-latency`) and inherit the rest — no copy-pasted PromQL to maintain.

**Standalone example (not part of the webshop)**
- `settings/dashboard.sty` — a `Dashboard` (Pod resources), included only to show
  how to ship a dashboard in a StackPack. Delete it if you don't need it.

> Metrics no longer use a separate `MetricBinding` resource. In StackPacks 2.0
> metrics are defined inside a `ComponentPresentation` (see the `metrics` block
> and the `summary` / `highlight.metrics` / `metricPerspective` sections in
> `settings/presentations/cart.sty`).

Every `.sty` file starts with a one-line header comment explaining what it is.
Start with `settings/presentations/cart.sty` — it is the richest example. The
matching reference documentation is linked at the bottom of this README.

## Project Structure

```
<< .Name >>/
├── README.md
├── stackpack.yaml                     # StackPack manifest
├── icons/                             # SVG icons used by !icon
│   ├── microservice.svg
│   └── database.svg
├── settings/                          # Everything here is imported on install
│   ├── main-menu.sty                  # MainMenuGroup
│   ├── component-mappings/            # OtelComponentMapping (telemetry -> components)
│   │   ├── microservices.sty          # demo service instances (except kafka) -> microservice
│   │   └── database.sty               # kafka service -> database
│   ├── relation-mappings/             # OtelRelationMapping (edges between components)
│   │   ├── checkout-to-cart.sty       # service-graph metric edge
│   │   └── consumer-to-database.sty   # consumer-span messaging edge
│   ├── presentations/                 # ComponentPresentation (how components look)
│   │   ├── microservice.sty           # base for all microservice instances
│   │   ├── cart.sty                   # rich, cart-specific (the full tour)
│   │   ├── checkout.sty               # rich, checkout-specific
│   │   └── database.sty
│   ├── monitors.sty                   # Monitor on the cart's domain metric
│   └── dashboard.sty                  # Dashboard (standalone)
├── includes/                          # Large values pulled in via !include
│   └── remediation-hints/
│       └── cart-add-item-latency.md.hbs
└── resources/                         # Markdown shown for each lifecycle state + logo
    ├── overview.md
    ├── installed.md
    ├── notinstalled.md
    ├── provisioning.md
    ├── deprovisioning.md
    ├── waitingfordata.md
    ├── error.md
    └── logo.png
```

## How the presentation example fits together

The three concepts build on each other — this is the mental model for any
custom integration:

1. **Mapping** (`component-mappings/`) — a component only exists if a mapping
   creates it from incoming telemetry. Each mapping selects some telemetry
   (here, by `service.namespace` + `service.instance.id`) and emits a component
   with a stable URN and a type. Every demo service instance is a `microservice`,
   so many instances share one type. `output.required.additionalIdentifiers`
   claims the generic OpenTelemetry service-instance URN so the components merge.
2. **Relation** (`relation-mappings/`) — connects two components into a topology.
   The two examples use two signals: `checkout-to-cart.sty` reads the
   service-graph **metric** (for the synchronous checkout instance → cart instance
   call), while `consumer-to-database.sty` reads consumer **spans**
   (`messaging.system = kafka`) to link each consumer instance → the kafka
   service. Kafka is not in the service graph, so the span is the only honest
   signal — a good illustration that the right source depends on the interaction.
3. **Presentation** (`presentations/`) — binds to a set of components and
   describes their columns, detail page, metrics, filters, icon and menu
   placement. Presentations **compose two ways**: (a) our own base
   `microservice.sty` is refined by `cart.sty` / `checkout.sty` at higher
   `rank.specificity`; and (b) because our bindings are a subset of the generic
   OpenTelemetry service-instance binding, we **inherit** the generic span metrics
   and only add our domain-specific ones. Metric queries are scoped to one
   instance with `service_instance_id`.

Concepts demonstrated across the presentation files: bindings,
`rank` composition, **metric inheritance via binding subset**, icon, filters
(definition & reference), overview name/mainMenu/columns/sort, projection types
(Health, ComponentLink, Text, Tag, Metric),
highlight fields/provisioning/relatedResources/**links**/events, summary metrics,
highlight metrics, metric perspective (tabs → sections), metric definitions with
charts, and a **span-level** relation mapping (the Kafka consumer edge). The
checkout highlight adds a **link** to that instance's traces.

A few projection types are intentionally **not** used because the webshop's OTel
data has nothing honest to drive them — see the reference docs for
`RatioProjection`, `NumericProjection`, `ContainerImageProjection`,
`MapProjection`, and `DurationProjection`. (`DurationProjection` needs an ISO8601
start time; an OpenTelemetry service instance carries no portable start timestamp
— `k8s.pod.start_time` only exists on Kubernetes — so we leave it out rather than
ship a column that is blank off-Kubernetes.)

## Reference documentation

The StackPacks 2.0 reference docs are the companion to these examples:

- Presentation concepts: <https://documentation.suse.com/cloudnative/suse-observability/latest/en/setup/custom-integrations/presentation/concepts.html>
- Full presentation schema: <https://documentation.suse.com/cloudnative/suse-observability/latest/en/setup/custom-integrations/presentation/schemas-ref.html>
- Projections: <https://documentation.suse.com/cloudnative/suse-observability/latest/en/setup/custom-integrations/presentation/projections.html>
- Metrics: <https://documentation.suse.com/cloudnative/suse-observability/latest/en/setup/custom-integrations/presentation/metrics/README.html>
- OpenTelemetry mappings: <https://documentation.suse.com/cloudnative/suse-observability/latest/en/setup/custom-integrations/otelmappings/concepts.html>

> These reference pages are part of the StackPacks 2.0 documentation and may be
> gated until the 2.0 release ships.

## Customization Guide

1. **Manifest** — edit `stackpack.yaml`: `displayName`, `categories`, `version`.
2. **Point at your own services** — in `component-mappings/`, change the
   `condition` (`service.name == '...'`), the `output.identifier` template, and
   the `output.typeName`. Keep the relation mapping's URN templates in sync.
3. **Shape the UI** — edit `presentations/`. Reuse shared columns/filters from
   `microservice.sty`; add type-specific columns, metrics and highlight fields per type.
4. **Extract large values** — long strings (like remediation hints) can live in
   `includes/` and be pulled in with the `!include` tag, e.g.
   `remediationHint: !include "remediation-hints/cart-add-item-latency.md.hbs"`.
5. **Lifecycle messages** — edit `resources/*.md`.

## Development Workflow

```bash
# Validate the StackPack settings
sts stackpack validate

# Package it into a shareable .sts file
sts stackpack package

# Upload to a test instance of SUSE Observability
sts stackpack upload << .Name >>-0.0.1.sts
```
