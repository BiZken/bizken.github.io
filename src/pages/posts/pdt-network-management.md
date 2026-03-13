---
layout: ../../layouts/Post.astro
title: "Mobile Network Device Management"
description: "A lightweight network monitoring system using RESTCONF and Flask, enabling mobile-friendly device management via QR codes."
date: "2025-01-15"
type: project
tags: ["RESTCONF", "Python", "Flask", "Networking", "Project"]
backLink: /projects
backLabel: Back to projects
---

**Course:** PDT401 – Praktisk datateknik (Computer Engineering in Practice)  
**University:** University West (Högskolan Väst)  
**Date:** January 2025  
**Team:** Anton Olsson, Oskar Klerestam, Linus Gieryluk Lindstrand, David Skoglund Landström  
**Tech Stack:** Python, Flask, RESTCONF, HTML/CSS, Jinja2

---

## Abstract

This project introduces a lightweight network management system using RESTCONF and a Flask-based web app, emphasizing ease of use, mobile accessibility, and simple deployment without containerization or databases. Key features include QR code integration, dynamic GUIs, and mobile compatibility.

Unlike existing solutions reliant on CLI tools or complex setups, our system directly manages devices via RESTCONF, providing a user-friendly and efficient approach to network monitoring and management.

---

## Introduction

This project was done in collaboration with **Stora Enso**, who wanted to explore how a mobile solution for network device monitoring could work. An agreement was made for a RESTCONF-based solution using a web interface. The Python Flask framework was used for the main application, and QR codes were implemented to allow selecting specific devices — making it well-suited for handheld devices.

---

## Background

### RESTCONF

RESTCONF is a network management protocol developed under the IETF to enable configuration and monitoring of network devices (routers, switches) using standard web-based methods. It leverages HTTP/HTTPS and REST principles such as stateless communication and resource-based interactions, offering a more flexible and modern alternative to older protocols like SNMP and NETCONF. RESTCONF plays an important role in the growth of Software-Defined Networking (SDN) and Network Functions Virtualization (NFV).

### Flask

Flask is a lightweight WSGI web framework written in Python, designed to be simple and versatile. Its core components include:

- **Werkzeug** — a comprehensive WSGI web application library
- **Jinja2** — a templating engine for designing HTML pages with Python-like syntax

---

## Method

The system consists of three main components:

1. **Flask-based web server** — manages incoming HTTP requests and serves the user interface
2. **Network devices** — Cisco routers/switches configured with RESTCONF support
3. **Communication server** — handles RESTCONF API calls to the network devices

### How It Works

- The Flask server is deployed on an Ubuntu server and acts as a middleman between users and network devices.
- When a user scans a **QR code** associated with a network device, the server sends a GET request to that device using the RESTCONF API.
- Authentication uses base64-encoded credentials in the request header; responses come in JSON format.
- Device management interfaces use static IP addresses on a management VRF, while client devices receive addresses dynamically via wireless.
- The UI is built with HTML/CSS and Jinja2 templates, dynamically adapting to the specific interfaces and capabilities of each device model.

**Example request flow:**
```
User scans QR → GET http://server-ip:5000/device_ip → Flask queries device via RESTCONF → JSON response → Rendered on dynamic template
```

<img src="/PDT/PDTTOPOLOGY" alt="DashBoard" style="max-width: 500px; width: 100%;">


---

## Results

The project delivered a functional network monitoring and management system with the following key outcomes:

- **RESTCONF + Flask Integration** — successful monitoring of network devices with a mobile-friendly web interface
- **QR Code-Based Access** — users can quickly access device information by scanning physical QR codes, reducing the need for a computer
- **Dynamic GUI** — the web interface adapts dynamically to the number of interfaces on each device using Jinja2 templates
- **Simple Deployment** — no containers, no databases — just a Flask server running on Ubuntu

<img src="/PDT/InterFAce.png" alt="DashBoard" style="max-width: 250px; width: 100%;">

---

### Challenges

- Some older IOS versions on network devices didn't support RESTCONF, resolved by upgrading firmware or using supported hardware.
- Different device models use different URI syntaxes for RESTCONF — handled by referencing each device's documentation.

---
