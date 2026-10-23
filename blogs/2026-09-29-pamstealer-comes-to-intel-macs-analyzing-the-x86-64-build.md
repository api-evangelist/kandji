---
title: "PamStealer comes to Intel Macs: Analyzing the x86_64 build of a Rust macOS stealer"
url: "https://www.iru.com/blog/pamstealer"
date: "2026-09-29"
author: "Cristian Molina"
feed_url: "https://www.iru.com/blog/rss.xml"
---
In late August 2026, automated telemetry identified an in-the-wild sample of PamStealer , an emerging Rust-based macOS information stealer first documented by Jamf Threat Labs . Prior public analysis observed PamStealer operating exclusively as an Apple Silicon ( arm64 ) payload delivered via trojanized disk images impersonating the Maccy clipboard utility. This analyzed artifact is the Intel ( x86_64 ) architecture slice of a multi-architecture fat binary, confirming that the threat actors have expanded their build pipeline to ensure uniform execution across legacy and modern Apple hardware.
