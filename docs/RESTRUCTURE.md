# Repository restructure (code organization)

This document describes the code organization changes made to improve readability and maintainability **without changing functionality**.

## High-level structure

Most of the code already lives in cohesive module folders (e.g., `src/hostif/**`, `src/unittest/**`). To avoid destabilizing include paths and automake build definitions, this reorganization focuses on **safe, low-ripple moves** first, while keeping large existing cohesive trees in place.

Current key directories:

- `src/core/`  
  Core/top-level utilities and entrypoint-adjacent helper programs.

- `src/hostif/`  
  Main TR-069 host interface implementation:
  - `handlers/` request/message handler modules
  - `profiles/` TR-181 profile implementations (WiFi, IP, Ethernet, DeviceInfo, etc.)
  - `httpserver/`, `parodusClient/`, `snmpAdapter/`, `include/`, `src/`

- `src/unittest/`  
  Unit tests and stub infrastructure (kept in place to preserve existing include and build assumptions):
  - `src/unittest/stubs/**` (HAL/RBUS/IARM/WebPA stubs, secure wrapper, etc.)

## Old → new path mapping (moved files)

| Old path | New path | Notes |
|---------|----------|------|
| `src/backgroundrun.c` | `src/core/backgroundrun.c` | Updated `src/Makefile.am` `backgroundrun_SOURCES` accordingly |

## Files intentionally left in place

- `src/hostif/**`: already domain-structured; moving would require widespread automake and include changes.
- `src/unittest/**`: tests/stubs depend on stable include paths; kept unchanged.

## Build updates

- Automake (`src/Makefile.am`) updated to compile `backgroundrun` from its new location:
  - `backgroundrun_SOURCES = $(top_srcdir)/src/core/backgroundrun.c`

If additional domain-folder moves are required later, they should be done module-by-module with corresponding updates to the relevant `Makefile.am` in that module and verification that `#include` directives still resolve via existing `-I...` flags.
