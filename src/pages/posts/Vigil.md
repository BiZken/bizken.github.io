---
layout: ../../layouts/Post.astro
title: "Vigil"
description: "A hardening system and dashboard. Agents run CIS benchmark checks on hosts and report back to a central dashboard."
type: project
tags: ["Python", "FastAPI", "Security", "Linux", "CIS", "Project"]
backLink: /projects
backLabel: Back to projects
---

# Vigil - Reasoning

I decied to learn FastAPI and needed some sort of project for it and decided to go with a project that check CIS benchmarks on systems and then reports back to a centralized dashboard.
A small two-part system for keeping track of how well hardened hosts are.

## What it does

An **agent** runs on each host amnd executes a set of CIS benchmark checks, and POSTs the results to a central **dashboard**. The dashboard scores each host, evaluates the results against rules, and serves a web UI so you can see everything in one place.

## The checks

Each agent runs checks across the differencet CIS rankings ("CAT I-III)
Prio is CAT III checks and then update as we go 

## How it works
<img src="/vigil/simple.png" alt="Res1" style="max-width: 500px; width: 100%;">
<img src="/vigil/overall.png" alt="Res1" style="max-width: 500px; width: 100%;">

## Features to add:
 - Auto create a report for each host
 - Custom checklist of rules, being able from UI to pick benchmarks that will be checked
 - Auto fix issues detected
 - VPN connection
 - Currently only working on Ubuntu, will have more OS later
 - Ticket system for multi user dashboard


Agents can run as a one-shot (`python run_agent.py`) or as a daemon (`--daemon`) that loops on a configurable interval.
