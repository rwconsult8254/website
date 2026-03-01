---
title: "Zero False Alarms — Detecting Safety Valve Failures by Reading the Physics"
description: "A physics-based detector for spurious DHSV closure achieves 100% detection with zero false alarms across 1,119 real well instances from the Petrobras 3W dataset — no machine learning required."
pubDate: 2026-03-01
tags:
  - anomaly-detection
  - oil-and-gas
  - process-safety
  - 3W-dataset
  - physics-based-modelling
---

This article describes how we built a detector for spurious DHSV closure using the Petrobras [3W dataset](https://github.com/petrobras/3W). No machine learning — just the physics of the failure. The result is 100% detection with zero false alarms. The code is available on [GitHub](https://github.com/rwconsult8254/3w-dataset).

## The Problem

The downhole safety valve is the last line of defence — its failure to close on the Macondo well in 2010 contributed to the Deepwater Horizon disaster and catastrophic environmental damage.

The downhole safety valve — more commonly the sub-surface safety valve — is a valve placed in the production tubing that acts to prevent flow to the surface should the wellhead or production flowline become damaged. It is a fail closed valve with hydraulic pressure keeping it open. Should the hydraulic system that keeps it open fail, the valve will fail to a safe position, that is it fails closed. The valve is situated within the well below the surface. Depending on the type, they require wireline or a workover rig to retrieve should they need maintenance. This can be expensive and especially so offshore.

The 3W dataset contains 22 real instances of spurious DHSV closure across 7 wells. In 97% of those closures, the valve positioner did not pick up on the fact the valve had failed closed. The actual failure mechanism is unknown. We could assume perhaps that the hydraulic system keeping the valve open failed. There may be some common mode failure that also causes the positioner to fail.

In any case, the signature of the failure is unmistakable, and directly derived from the physics of the failure.

## Why Early Detection Matters

This is a rare failure and the operator might not easily diagnose what is going on. This can lead to slower response and perhaps even a misdiagnosis of the issue.

There may not be an individual flowmeter on each well. They are multiphase flows and hard to measure using conventional means. Unless a multiphase flowmeter is installed, flows of oil, water and gas will only be measured periodically during a well test. The first indication that an operator might receive that a well is not flowing might be a drop in production or drop in inlet pressure on the flowline from that well. The value here with early detection is that the operator will know with some certainty that the cause of the failure is a DHSV failure, even if the positioner did not indicate it as such.

If the DHSV has failed, a workover rig will be required and these are not available simply on standby. Worse, if a DHSV is not the root cause and a workover rig is mobilised to fix it, then there will be significant expense wasted with the problem unresolved.

## Sensor Quality

In any machine learning problem, we must clean data. This includes finding and removing features that are clearly giving incorrect figures and if left in, might degrade the ML model.

Running a quality check across all 18 continuous sensors in every one of the 1,119 real instances reveals that some sensors are far less reliable than others. The check is simple: flag sensors that are all-zero (dead), frozen at a constant value (stuck gauge), reading physically impossible values (negative pressure), or toggling erratically between samples.

The downhole sensors are the worst performing. P-PDG (downhole pressure) is present in 827 of 1,119 instances, but 80% of those are broken — mostly all-zero readings from dead sensors. T-PDG (downhole temperature) fails 60% of the time, predominantly frozen or all-zero. We should not be too surprised. These sensors are installed downhole and operate in harsh conditions. They are also not easy to replace, most likely requiring a workover to do so. When they fail, they are most likely simply left in place.

Of the 22 real DHSV instances, 8 were excluded for broken P-PDG sensors (dead, frozen, or noisy), leaving 14 usable real instances and 16 simulated — 30 usable in total.

## False Alarms and the Physics Insight

Our aim is to provide information to operators that might not be easily apparent. We want to detect issues early and provide valuable information to fix issues as quickly as possible. These are fantastic goals and it is hard to see any operator that would not want that. If this information comes at the cost of many false alarms, however, it can be worse than having no information at all.

Poor alarm management is one of the factors that run through all process major accidents. False alarms erode confidence in the algorithm and multiple will lead to it being ignored. Worse than if the algorithm had never been put in place.

The signature of a DHSV failure should be very clear — a rapid rise in pressure across the DHSV. Our first attempt flagged any large 1-second pressure spike. This did identify most failures, but sensor noise regularly produces single-sample spikes that look statistically extreme but are physically meaningless. It detected 9/14 (64%) of real events but generated 15 false alarms per hour — unusable.

The insight was that a real closure produces a **sustained** pressure rise — 4.4 to 95.7 bar over 30 seconds — while normal noise produces less than 0.1 bar over the same window. By switching to a 30-second rolling window and requiring a physical minimum of 3.5 bar, a rising pressure direction, and persistence over several seconds, we eliminated every false alarm across all 1,119 real instances while detecting 100% of failures at 50-second median latency.

The change wasn't adding filters to V1. It was recognising that the physics gives you a sustained signal, not a spike, and matching the detector to that reality.

The 3.5 bar threshold is not a tuned parameter — it's derived from the smallest observed DHSV closure (4.4 bar over 30 seconds) with a margin. It represents the physical reality that reservoir pressure buildup against a closed valve produces bar-scale changes, not millibar noise.

## Results

| Metric | V1 | V2 |
|---|---|---|
| Approach | 1s dP/dt, 4-sigma threshold | 30s rolling ΔP, direction filter, 3.5 bar physical minimum, 4σ stat threshold, persistence debounce |
| Detection rate (real) | 9/14 (64%) | 14/14 (100%) |
| Detection rate (simulated) | — | 16/16 (100%) |
| Median latency (real) | 25s | 50s |
| Max latency (real) | 1,283s | 435s |
| False alarm rate | 15.45/hour | 0.00/hour |
| Validation scope | Class 2 only | All 1,119 real instances, 10 event classes |
| Cross-fire on hydrate | — | 2 instances (physically expected) |
| ESTADO-DHSV | — | Shows closure in 1/38 instances |

We place false alarms above missed detection. This of course can be tailored to operator preference and tuned as required, but it highlights the difference in machine learning for operations and machine learning for science.

## Differential Diagnosis

DHSV closure and hydrate formation both cause rising P-PDG because both restrict flow downstream of the gauge. The detector correctly fires on hydrate instances — because something is wrong. The discriminant is onset speed: DHSV is a step change (under 1 second), hydrate is a gradual ramp (minutes to hours).

Cross-class physics overlap is real and informative, not a bug. A production system flags the restriction, then diagnoses the type. This is how operators think — "the well stopped, what happened?" not "is this DHSV or hydrate?"

## How This Compares to the Published Literature

Our literature review looked at 121 published works citing the 3W dataset.  We were unable to find any other work where false alarms are given any discussion in operational terms.  We also found no other work where a physics-based approach was used first before machine learning.

Our philosophy is to understand the problem from a technical and operational view first.  If a machine learning approach is to be undertaken, it should have a basis in the physics of the operation.  This ensures that the results have the end user in mind and are more easily explained without appearing to be a 'black-box'.

## The Broader Lesson

The DHSV detection work on the 3W dataset demonstrated that a physics-based threshold detector — no machine learning — was sufficient to achieve 100% detection on quality-gated instances with zero false alarms across all 1,119 real well instances.

When the underlying physics creates a clear separation between real events and noise, a learned decision boundary adds nothing — and risks over-fitting to the noise characteristics of the training data. Start with domain physics, only add ML when the physics isn't enough.

In a future article we will look at the rapid increase in BS&W anomaly where physics alone is not sufficient and hits limits.
