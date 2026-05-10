+++
title = "evo framework"
description = "A brand-neutral steward for appliance-class devices. Everything is a plugin."
template = "index.html"
sort_by = "weight"
+++

<div class="home-hero-layout">
<div class="home-hero-text">

<h1 class="home-hero">Everything is a plugin. The framework just stewards them.</h1>

<p class="home-subhead">
Evo is a Rust framework for appliance-class devices where features
are independent, signed plugins composed by a central steward that
knows nothing about the domain. Adding a capability is a new plugin.
Replacing one is replacing one signed file. The system stays
coherent as the plugin set grows.
</p>

</div>

<div class="home-hero-visual" aria-hidden="true">
<div class="hero-mark">
<div class="hero-mark-glyph"></div>
<span class="hero-mark-dot hero-mark-dot-1"></span>
<span class="hero-mark-dot hero-mark-dot-2"></span>
<span class="hero-mark-dot hero-mark-dot-3"></span>
</div>
</div>
</div>

<hr class="section-divider">

## The problem

Appliance-class devices keep being built on stacks that were designed
for something else. A web runtime running as a backend daemon. An
event loop holding global state that two plugins both want to mutate.
A monolithic application that admits "extensions" but still owns the
truth. The result is a class of device that starts clean, accumulates
race conditions, develops a slow response surface, and ships a new
feature only by editing the centre. The user feels it as the device
becoming worse over time. The maintainer feels it as fear of the next
change. Both are symptoms of the same architectural choice, made
early, which the system can no longer escape.

## The move

Evo inverts the arrangement. The framework has no domain knowledge.
The device is described by a catalogue file, a small data document
that declares which concerns the device has, plus a set of plugins
that satisfy those concerns. There is no central application that
plugins are guests of. The catalogue is the device's specification;
the framework, which evo calls the steward, administers it. Plugins
contribute, and the steward composes their contributions into the
coherent thing the user sees.

## Why this is different

If you have used a device whose backend is a Node.js or Python
application server: the application owns the truth. Plugins are
callbacks the application invokes. Two plugins interested in the
same state coordinate through shared variables, and the application
is responsible for serialising them, which it usually does by
accident. Race conditions are the default; absence of races is a
feature you pay for plugin by plugin. In evo, the steward is the
only place state changes; plugins never see each other; the race
condition is not mitigated, it is structurally absent.

If you have built a plugin for an audio appliance: the plugin model
is usually a fixed set of extension points (decoder, source, output)
hung off the side of a core application. The application owns
playback, metadata, networking, the UI; plugins narrow the gaps.
Replacing the playback engine is forking the project. In evo, the
playback engine itself is a plugin. So is metadata, artwork, the
network manager. Nothing is privileged. Replacing the playback
engine is replacing one signed artefact.

If you have shipped firmware with a release cycle measured in
months: every change ships together because the build is monolithic.
Fixing a typo in an ALSA parameter means cutting a release. In evo,
a typo is a one-line config edit on the device; a plugin bug fix is
replacing one binary; a framework update is a deliberate act. Three
independent release cadences, three independent signing keys, three
planes (source, artefact, operational) that never blur into each
other.

## What this discipline buys

- Adding a feature is a new plugin. The framework is unchanged.
- Replacing a component is replacing one signed file. Nothing else
  moves.
- Two vendors building similar devices share the same brand-neutral
  plugins; only the vendor layer differs.
- Fixing a typo in a hardware parameter is a one-line config edit,
  not a rebuild and a re-flash.
- The integration cost of the plugin ecosystem stays flat as it
  grows. Plugin authors learn the contract once.
- Anywhere with a CPU, RAM, and a bit of storage is in scope. From
  a wearable counting heart-rate to a datacenter inference rack, the
  same fabric vocabulary applies.

The framework runs on Unix today (eight architectures, three ARM
profiles, glibc and musl). The contracts the framework defines do
not assume Unix; ports to RTOS, bare-metal, and embedded runtimes
are open invitations. The
[distributions page](/distributions/) shows the spread.

<hr class="section-divider">

## Light by design

Pure Rust. No garbage collector, no JavaScript runtime, no PHP, no
JVM, no shell pipelines holding state. The framework is one
long-running process and the plugins it admits, and every
architectural choice above this line is also an efficiency choice.

A typical evo steward uses tens of MB of memory at idle and a
fraction of a percent of CPU. The same workload on a Node.js or
Python application server uses hundreds of MB of memory, a JIT or
interpreter warmup window, and GC pauses every few seconds. The
runtime overhead a VM-hosted stack pays is simply not paid here.

The consequences are practical. A wearable's battery lasts longer.
A datacenter's cooling bill is smaller. A satellite's solar panel
covers more workload. Less work uses less power; that is the whole
story.

<hr class="section-divider">

## Three tiers

Evo is not framework + your app. It is three tiers, each with its
own release cadence, its own signing key, and its own job.

<div class="tier-diagram-wrap">
<svg class="tier-diagram" viewBox="0 0 800 660" xmlns="http://www.w3.org/2000/svg" preserveAspectRatio="xMidYMid meet" role="img" aria-labelledby="tier-diagram-title tier-diagram-desc">
<title id="tier-diagram-title">Three tiers: distribution, steward, consumers</title>
<desc id="tier-diagram-desc">A vertical architecture diagram. The distribution tier feeds the steward at the centre across four contracts at the boundary; the steward serves consumers below across the client socket.</desc>

<defs>
<linearGradient id="tier-bg" x1="0" y1="0" x2="1" y2="1">
<stop offset="0%" stop-color="#0e1d30"/>
<stop offset="55%" stop-color="#102a44"/>
<stop offset="100%" stop-color="#0a1828"/>
</linearGradient>

<linearGradient id="steward-bg" x1="0" y1="0" x2="1" y2="1">
<stop offset="0%" stop-color="#0d2236"/>
<stop offset="50%" stop-color="#143a5c"/>
<stop offset="100%" stop-color="#0a1d30"/>
</linearGradient>

<linearGradient id="brand-mark-grad" x1="0" y1="0" x2="1" y2="1">
<stop offset="0%" stop-color="#00d4aa"/>
<stop offset="100%" stop-color="#00a080"/>
</linearGradient>

<filter id="tier-glow" x="-10%" y="-15%" width="120%" height="130%">
<feGaussianBlur stdDeviation="4" result="blur"/>
<feFlood flood-color="#00d4aa" flood-opacity="0.18"/>
<feComposite in2="blur" operator="in" result="glow"/>
<feMerge>
<feMergeNode in="glow"/>
<feMergeNode in="SourceGraphic"/>
</feMerge>
</filter>

<filter id="steward-glow" x="-10%" y="-15%" width="120%" height="130%">
<feGaussianBlur stdDeviation="7" result="blur"/>
<feFlood flood-color="#00d4aa" flood-opacity="0.32"/>
<feComposite in2="blur" operator="in" result="glow"/>
<feMerge>
<feMergeNode in="glow"/>
<feMergeNode in="SourceGraphic"/>
</feMerge>
</filter>

<marker id="arrow" viewBox="0 0 12 12" refX="6" refY="6" markerWidth="9" markerHeight="9" orient="auto-start-reverse">
<path d="M2,2 L10,6 L2,10 z" fill="#00d4aa"/>
</marker>
</defs>

<g filter="url(#tier-glow)">
<rect x="80" y="20" width="640" height="140" rx="18" fill="url(#tier-bg)" stroke="#1f3a5c" stroke-width="1"/>
<rect x="80" y="20" width="640" height="3" rx="1.5" fill="url(#brand-mark-grad)" opacity="0.7"/>
</g>
<text x="400" y="64" text-anchor="middle" class="svg-tier-label">DISTRIBUTION</text>
<text x="400" y="92" text-anchor="middle" class="svg-tier-subtitle">evo-device-&lt;vendor&gt;</text>
<text x="400" y="128" text-anchor="middle" class="svg-tier-content">catalogue . plugins . branding . packaging</text>

<line class="svg-connector" x1="400" y1="170" x2="400" y2="220" stroke="#00d4aa" stroke-width="2" stroke-dasharray="4 5" marker-end="url(#arrow)"/>
<text x="420" y="200" class="svg-connector-label">four contracts at the boundary</text>

<g filter="url(#steward-glow)">
<rect x="60" y="240" width="680" height="200" rx="20" fill="url(#steward-bg)" stroke="#256086" stroke-width="1.5"/>
<rect x="60" y="240" width="680" height="4" rx="2" fill="url(#brand-mark-grad)"/>
<g transform="translate(696, 256)">
<rect width="22" height="22" rx="5" fill="url(#brand-mark-grad)"/>
</g>
</g>
<text x="400" y="294" text-anchor="middle" class="svg-tier-label svg-tier-label-strong">THE STEWARD</text>
<text x="400" y="326" text-anchor="middle" class="svg-tier-subtitle">evo-core</text>
<text x="400" y="368" text-anchor="middle" class="svg-tier-content">catalogue . admission . subjects . relations</text>
<text x="400" y="396" text-anchor="middle" class="svg-tier-content">custody ledger . projections . happenings bus</text>

<line class="svg-connector" x1="400" y1="450" x2="400" y2="500" stroke="#00d4aa" stroke-width="2" stroke-dasharray="4 5" marker-end="url(#arrow)"/>
<text x="420" y="480" class="svg-connector-label">client socket</text>

<g filter="url(#tier-glow)">
<rect x="80" y="520" width="640" height="120" rx="18" fill="url(#tier-bg)" stroke="#1f3a5c" stroke-width="1"/>
<rect x="80" y="520" width="640" height="3" rx="1.5" fill="url(#brand-mark-grad)" opacity="0.7"/>
</g>
<text x="400" y="568" text-anchor="middle" class="svg-tier-label">CONSUMERS</text>
<text x="400" y="608" text-anchor="middle" class="svg-tier-content">frontend . CLIs . bridges . automation</text>
</svg>

<p class="tier-diagram-caption">
The framework holds the middle. Distributions adopt reference generic
devices and add the vendor layer. None of these tiers know about each
other except through the contracts.
</p>
</div>

<hr class="section-divider">

## A real device exists

<div class="card-grid">

<div class="card">
<h3>evo-device-audio</h3>
<div class="card-meta">
<span class="pill pill-info">Reference generic</span>
<span class="pill pill-ok">Working</span>
</div>
<p>The reference generic device for the audio domain. Brand-neutral
plugins under <code>org.evoframework.*</code>: MPD playback, ALSA
composition, file-tag metadata, local artwork. Used today as the
canonical demonstration of every framework function.</p>
<div class="card-links">
<a href="https://github.com/foonerd/evo-device-audio">Source</a>
<a href="https://github.com/foonerd/evo-device-audio-artefacts">Artefacts</a>
</div>
</div>

<div class="card">
<h3>Your distribution here</h3>
<div class="card-meta">
<span class="pill pill-muted">Open</span>
</div>
<p>The framework supports an open set of vendor distributions. Adopt a
reference generic device by name, add catalogue choices, branding,
product-specific plugins, and packaging. The pattern is the same for
every domain. The
<a href="https://github.com/foonerd/evo-core/blob/main/docs/engineering/BOUNDARY.md">BOUNDARY</a>
document is the normative checklist.</p>
<div class="card-links">
<a href="/build/">Build a distribution</a>
</div>
</div>

</div>

<hr class="section-divider">

## Sixty seconds

A steward, a one-shelf catalogue, and a client. No audio, no hardware.
The fabric is the same shape regardless of domain.

```bash
# Clone, build, and run a steward locally:
git clone https://github.com/foonerd/evo-core.git && cd evo-core
cargo build --workspace

mkdir -p /tmp/evo
cat > /tmp/evo/catalogue.toml <<'EOF'
[[racks]]
name = "example"
family = "domain"
kinds = ["registrar"]
charter = "Minimal example rack."

[[racks.shelves]]
name = "echo"
shape = 1
description = "Echoes inputs back."
EOF

cargo run -p evo -- \
    --catalogue /tmp/evo/catalogue.toml \
    --socket /tmp/evo/evo.sock \
    --log-level info
```

In another terminal, probe it:

```python
import socket, struct, json, base64
s = socket.socket(socket.AF_UNIX); s.connect("/tmp/evo/evo.sock")
req = json.dumps({
    "op": "request", "shelf": "example.echo", "request_type": "echo",
    "payload_b64": base64.b64encode(b"hello").decode()
}).encode()
s.send(struct.pack(">I", len(req)) + req)
n = struct.unpack(">I", s.recv(4))[0]
resp = json.loads(s.recv(n).decode())
print(base64.b64decode(resp["payload_b64"]).decode())  # -> "hello"
```

The full client protocol with example clients in seven languages is
in [CLIENT_API.md](https://github.com/foonerd/evo-core/blob/main/docs/engineering/CLIENT_API.md).

<hr class="section-divider">

## The words evo uses

Evo's vocabulary is load-bearing. Every word below appears with the
same meaning across the framework, the documentation, and the
catalogue files.

<div class="vocabulary-grid">

<div class="vocab-cell">
<span class="vocab-term">Steward</span>
<span class="vocab-role">Sole authority. Admits, places, composes, dispatches, projects, notifies.</span>
</div>

<div class="vocab-cell">
<span class="vocab-term">Catalogue</span>
<span class="vocab-role">Declared data. Racks, shelves, slots, shapes, relation grammar.</span>
</div>

<div class="vocab-cell">
<span class="vocab-term">Rack</span>
<span class="vocab-role">A concern. Holds shelves. Belongs to a family and a kind.</span>
</div>

<div class="vocab-cell">
<span class="vocab-term">Shelf</span>
<span class="vocab-role">A slot or slot-set of declared shape within a rack.</span>
</div>

<div class="vocab-cell">
<span class="vocab-term">Slot</span>
<span class="vocab-role">A typed opening that admits one or more plugin contributions.</span>
</div>

<div class="vocab-cell">
<span class="vocab-term">Plugin</span>
<span class="vocab-role">Any capability that stocks a slot. The only entity besides the steward.</span>
</div>

<div class="vocab-cell">
<span class="vocab-term">Subject</span>
<span class="vocab-role">A thing the catalogue has opinions about. Canonical identity held by the steward.</span>
</div>

<div class="vocab-cell">
<span class="vocab-term">Relation</span>
<span class="vocab-role">Typed directed connection between subjects.</span>
</div>

<div class="vocab-cell">
<span class="vocab-term">Projection</span>
<span class="vocab-role">Composed view emitted by the steward, keyed to a rack or a subject.</span>
</div>

<div class="vocab-cell">
<span class="vocab-term">Happening</span>
<span class="vocab-role">A transition event the steward emits on its notification stream.</span>
</div>

<div class="vocab-cell">
<span class="vocab-term">Appointment</span>
<span class="vocab-role">Time-originated instruction.</span>
</div>

<div class="vocab-cell">
<span class="vocab-term">Watch</span>
<span class="vocab-role">Condition-originated instruction.</span>
</div>

<div class="vocab-cell">
<span class="vocab-term">Custody ledger</span>
<span class="vocab-role">Registry of active warden assignments and their state.</span>
</div>

<div class="vocab-cell">
<span class="vocab-term">Fast path</span>
<span class="vocab-role">Real-time mutation channel alongside the structural slow path.</span>
</div>

<div class="vocab-cell">
<span class="vocab-term">Distribution</span>
<span class="vocab-role">A curated catalogue plus plugin set, shipped as a branded device.</span>
</div>

<div class="vocab-cell">
<span class="vocab-term">Essence</span>
<span class="vocab-role">The single sentence the steward enforces at startup and on admission.</span>
</div>

</div>

<hr class="section-divider">

## Where to go next

<div class="cta-row">

<a class="cta" href="/concept/">
<strong>Read the concept</strong>
<span>The fabric in evo's own words.</span>
</a>

<a class="cta" href="/build/">
<strong>Build a plugin</strong>
<span>Stock a shelf in someone's catalogue.</span>
</a>

<a class="cta" href="/build/">
<strong>Build a distribution</strong>
<span>Ship a device on the evo fabric.</span>
</a>

</div>

## Project status

Pre-1.0. The framework runtime and the SDK are written; the audio
reference generic device exists and is the canonical demonstration of
every core function; vendor distributions are an open invitation,
not yet announced.

The framework is licensed under the Business Source License 1.1; the
SDK, the operator CLI, the trust primitive, the proc macro, and the
example plugins are licensed under the Apache License 2.0. The names
"evo" and "evoframework" are trademarks; see the
[trademark policy](/project/#trademark).

Single-maintainer-driven. Open to contributions on the contracts the
engineering documents name. Issue and PR flow lives on
[GitHub](https://github.com/foonerd).
