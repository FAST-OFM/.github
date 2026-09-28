<!--
SPDX-FileCopyrightText: 2026 Alexander Fridman
SPDX-License-Identifier: CC-BY-NC-SA-4.0
-->

<p align="center">
  <img src="https://github.com/FAST-OFM.png?size=240" alt="Fast OFM logo" width="144">
</p>

<h1 align="center">Fast OFM</h1>

<p align="center">
  <strong>A modular whole-slide imaging research prototype built around OpenFlexure</strong>
</p>

<p align="center">
  Motion control · Multichannel illumination · Predictive RG autofocus · Pyramidal OME-BigTIFF
</p>

<p align="center">
  <sub>Research prototype · Not clinically validated · Not for diagnostic use</sub>
</p>

Fast OFM is Alexander Fridman's research initiative exploring how an
accessible microscope can be extended into a practical
whole-slide imaging pipeline. It brings motion, illumination, focus estimation,
scan orchestration and image assembly together while keeping each modification
separate enough to inspect, reproduce or reuse independently.

I am releasing this working **prototype checkpoint** to support the
development of accessible digital microscopy. It is not a
finished product and does not claim clinical performance.

## What changed

| Area | Fast OFM contribution | Where to inspect it |
| --- | --- | --- |
| Motion | NEMA 11 motion driven by an MKS Robin Mini V2.0 with Klipper and Moonraker | [`controller`](https://github.com/FAST-OFM/controller) and [`hardware`](https://github.com/FAST-OFM/hardware) |
| Illumination | White and red/green illumination under Arduino brightness control and MKS timing gates | [`controller`](https://github.com/FAST-OFM/controller) |
| Focus | RG calibration, sparse focus sampling, focus-surface fitting and prediction behind a process boundary; white-light fallback remains in the GPL workflow | [`fast-ofm-core`](https://github.com/FAST-OFM/fast-ofm-core) and [`openflexure-wsi`](https://github.com/FAST-OFM/openflexure-wsi) |
| Scanning | Independent route policy plus tissue-aware OpenFlexure acquisition and telemetry | [`fast-ofm-core`](https://github.com/FAST-OFM/fast-ofm-core) and [`openflexure-wsi`](https://github.com/FAST-OFM/openflexure-wsi) |
| WSI output | Accelerated registration and rendering into pyramidal OME-BigTIFF through a replaceable LGPL worker | [`fast-ofm-stitching-openflexure`](https://github.com/FAST-OFM/fast-ofm-stitching-openflexure) |

The system uses an existing custom Cartesian linear stage; this release
documents its controller, calibration and imaging interfaces as part of the
complete scanner architecture.

## Measured prototype checkpoint

One retained tissue-aware 4K scan provides the current acquisition baseline.
Its adaptive spiral planner used a 7.5 mm centre-radius limit; that limit is
not the size of the retained mosaic:

| Metric | Measured value |
| --- | ---: |
| Addressed positions | 132 |
| Retained 4K fields | 97 |
| Background positions skipped | 35 |
| RG focus anchors | 30 |
| Predicted-Z captures | 67 |
| Acquisition time | 9 min 32 s |
| Retained-field axis-aligned bounding footprint | approximately 10.1 × 14.2 mm |

The retained field centres span 8.808 × 13.200 mm. Adding the calibrated 4K
field span gives an axis-aligned retained-field bounding footprint of
approximately 10.108 × 14.179 mm. This is not a claim that the whole bounding
rectangle is populated. The `15mm` text in the historical scan identifier
records the planner's nominal 15 mm diameter; the measured result above uses
the actual retained-field footprint.
The source frames were 4056 × 3040 pixels. The released system remains
host-orchestrated and stop-and-shoot; hardware-triggered continuous acquisition
is roadmap work.

## Repository index

Each repository corresponds to a distinct reuse boundary.

| Repository | Use it for | Start here |
| --- | --- | --- |
| [`fast-ofm`](https://github.com/FAST-OFM/fast-ofm) | Project map, architecture, evidence, release manifest and roadmap | [`FEATURE_INDEX.md`](https://github.com/FAST-OFM/fast-ofm/blob/main/FEATURE_INDEX.md) |
| [`openflexure-wsi`](https://github.com/FAST-OFM/openflexure-wsi) | Modified GPL OpenFlexure server/UI and hardware/workflow adapters | [`MODIFICATIONS.md`](https://github.com/FAST-OFM/openflexure-wsi/blob/main/MODIFICATIONS.md) |
| [`controller`](https://github.com/FAST-OFM/controller) | MKS/Klipper and Arduino source, configuration and timing experiments | [`README.md`](https://github.com/FAST-OFM/controller/blob/main/README.md) |
| [`hardware`](https://github.com/FAST-OFM/hardware) | Motion, optics, illumination, wiring and bounded bring-up guidance | [`README.md`](https://github.com/FAST-OFM/hardware/blob/main/README.md) |
| [`fast-ofm-core`](https://github.com/FAST-OFM/fast-ofm-core) | Source-available noncommercial RG, calibration, focus-surface and route-policy process | [`README.md`](https://github.com/FAST-OFM/fast-ofm-core/blob/main/README.md) |
| [`fast-ofm-stitching-openflexure`](https://github.com/FAST-OFM/fast-ofm-stitching-openflexure) | Replaceable LGPL registration, rendering and OME-BigTIFF worker | [`README.md`](https://github.com/FAST-OFM/fast-ofm-stitching-openflexure/blob/main/README.md) |

For the exact state of verified, offline-only, experimental and roadmap
features, read [`STATUS.md`](https://github.com/FAST-OFM/fast-ofm/blob/main/STATUS.md).

## System at a glance

```text
plan a region
     ↓
move the linear stage → select illumination → measure or predict Z
     ↓                                            ↓
capture retained 4K fields                 update focus evidence
     └───────────────────→ register → render → pyramidal OME-BigTIFF
```

The architecture deliberately separates commanded motion, optical measurement
and image assembly. This makes it possible to evaluate the RG autofocus or the
stitching work without adopting the entire hardware stack.

## Engineering story

A long-form Medium article and an OpenFlexure Forum post are being prepared.
Their permanent links will be added here before publication.

<!--
Publication checklist:
- Medium: replace this comment with the final article link.
- OpenFlexure Forum: add the final discussion link beside the Medium article.
-->

## Roadmap, not a release claim

The next architecture considered for Fast OFM uses a triggered 4K
global-shutter camera and executes the prepared motion, illumination and
camera-trigger schedule on the existing MKS/Klipper timeline. A companion MCU
is a fallback if the MKS interface cannot supply the required trigger/event
semantics. The aim is to overlap physical motion, exposure and host-side
focus-map updates. A greater-than-2× acquisition-throughput gain is plausible
but unmeasured; none of these items is presented as implemented in this
snapshot.

## Attribution and licensing

Fast OFM is an independent project built around the OpenFlexure Microscope. It
is not affiliated with or endorsed by the OpenFlexure project. Modified
OpenFlexure software retains its upstream identity, attribution and GPL-3.0
terms in `openflexure-wsi`.

Original Fast OFM documentation, controller material and hardware documentation
are source-available under noncommercial terms; GPL-derived material remains
GPL. Consult each repository's license files and the
[`LICENSING_BOUNDARY.md`](https://github.com/FAST-OFM/fast-ofm/blob/main/LICENSING_BOUNDARY.md)
before reuse.
