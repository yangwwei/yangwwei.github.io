---
title: Docker Network Tests under Host/Bridge Mode
tags:
  - Borg
  - YARN
---

Some reading notes about Google's Borg.

<!--more-->

## Google Borg

In 2015, Google published a paper introducing their large scale cluster management system `Borg`. The paper is available [here](https://research.google.com/pubs/archive/43438.pdf). It is the ancestor of `Kubernetes`, which manages distributed jobs in Google for decades in following 2 types,

* Producation (`Prod`)
	* Long-running services that should “never” go down, and handle short-lived latency-sensitive requests (a few μs to a few hundred ms). These are the high-priority jobs.
* Non-producation (`Non-Prod`)
	* Batch jobs that take from a few seconds to a few days to complete, they are much less sensitive to short-term performance fluctuations.

In a more fine-grained angle, every job has a `priority`. A high-priority task can obtain resources at the expense of a lower-priority one, even if that involves `preempting` (killing) the latter. Borg defines non-overlapping priority bands for different uses, including (in decreasing-priority order):

* monitoring (3)
* production (2)
* batch (1)
* best-effort (also known as testing or free) (0)

`prod` jobs are the ones in the `monitoring` and `production` bands.

User needs to specify job resource quato along with the priority during the submission, such as

	{
	  "priority" : "producation",
	  "CPU-quota" : 100,
	  "Memory-quota" : 20GB
	}

Prod quota is limited to actual resources available in the cell, so that a user who submits a production-priority job that fits in their quota can expect it to run, modulo fragmentation and constraints. At the same time, lower-priority jobs have infinite quota at priority zero. A low-priority job may be admitted but remain pending (unscheduled) due to insufficient resources.

Borg supports to specify `job constrains`, such as processor architecture, OS version, external IP address, to force jobs running on specific nodes. A constraint can be specified as hard (required) or soft (preferred).
