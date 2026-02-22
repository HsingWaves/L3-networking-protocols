# Network Protocols Lab & Troubleshooting Notes

This repo is a hands-on portfolio for core network protocols commonly required in data-center and HPC networking roles:
**TCP/IP, DHCP, BGP, OSPF/IS-IS, MPLS**.

## What’s inside
- **Labs**: reproducible lab write-ups with configs and troubleshooting steps
- **Notes**: concise protocol cheat sheets (10–20% most useful concepts)
- **Troubleshooting Playbooks**: production-style investigation checklists
- **Tools/Scripts**: small helpers for parsing outputs and analyzing PCAPs

## Principles
- Every lab includes: **symptoms → root cause → fix → verification**
- Prefer **failure-driven learning**: break things intentionally, then fix them

## Quick Start
- Lab environment: EVE-NG / GNS3 / FRR + Linux namespaces
- Start with:
  - `labs/ospf/lab-01-neighbor-stuck-init.md`
  - `labs/bgp/lab-01-ibgp-rr-clos.md`

## Progress Log
See `ROADMAP.md` for the 6-month plan and milestones.