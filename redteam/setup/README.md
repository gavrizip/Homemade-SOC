
# Setup

Installation and configuration for the offensive tooling used across `/redteam`.

## Index

| Guide                  | Covers                                                                        |
| ---------------------- | ----------------------------------------------------------------------------- |
| `01-parrotos-setup.md` | ParrotOS VM creation, network isolation to VLAN 40, base toolset verification |

## Scope of this folder

This folder covers **getting the attacker VM operational** — installing the OS, attaching it to the correct VLAN, and confirming isolation from Management/SOC before anything else runs. It does not cover how to use each individual tool; that lives in `../tools/`.

## Adding a new setup guide

Use this folder for anything that's a one-time environment setup step (a new attacker VM, a new tool that needs installing outside ParrotOS's default bundle). If it's usage/technique guidance for a tool already installed, it belongs in `../tools/` instead.


