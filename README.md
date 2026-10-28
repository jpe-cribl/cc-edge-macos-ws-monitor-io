# cc-edge-macos-ws-monitor-io

Cribl Edge pack for macOS WindowServer monitor telemetry.

The pack ships three pipelines, one breaker, and routes. Sources are configured outside the pack and connect to these pipelines by pipeline id.

## Installation

Install the released `cc-edge-macos-ws-monitor-io` `.crbl` file as a Cribl Edge pack.
The pack installs worker-level pipelines and breakers. It starts no sources.

## Usage

Point a file source on the Edge host at the matching pipeline:

- JSONL monitor samples go to `macos_ws_monitor`.
- Spindump summary files go to `macos_ws_monitor_captures`. Attach the `WS Monitor Captures` breaker to this source.
- Monitor error lines go to `macos_ws_monitor_errors`.

Sources that send events to routes are matched by the route filters below.

## Pipelines

| Pipeline | Index | Sourcetype | Input |
| --- | --- | --- | --- |
| `macos_ws_monitor` | `mac_perf` | `ws-monitor` | JSONL samples, one JSON object per line |
| `macos_ws_monitor_captures` | `mac_perf` | `ws-monitor:spindump` | One spindump summary file per event |
| `macos_ws_monitor_errors` | `mac_perf` | `ws-monitor:err` | Monitor error log lines |

Each pipeline sets constant `index` and `sourcetype` values with one eval function. Event fields are not changed.

## Breaker

`WS Monitor Captures` (`default/breakers.yml`) emits one event per file, with a maximum event size of 1 MiB.

## Routes

`default/pipelines/route.yml` matches these input ids and sends each to its pipeline:

| Input id | Pipeline |
| --- | --- |
| `file:in_ws_monitor` | `macos_ws_monitor` |
| `file:in_ws_monitor_captures` | `macos_ws_monitor_captures` |
| `file:in_ws_monitor_errors` | `macos_ws_monitor_errors` |

All routes use `output: __group`. No route targets `main`. No configured source connects to `macos_ws_monitor_errors` yet.

## Numeric fields in `macos_ws_monitor` events

Each JSONL event carries these JSON numbers. The pipeline passes them through unchanged, so a metric publish can read them directly.

| Field | Meaning | Unit |
| --- | --- | --- |
| `gap_s` | Loop timing gap | seconds |
| `io_probe_pending_s` | IO probe pending time | seconds |
| `memory_free_pct` | Free memory | percent |
| `swap_used_mb` | Swap in use | megabytes (may have decimals) |

## Fixtures and samples

- `tests/fixtures/<pipeline>/<case>.json` holds input events. `<case>.expected.json` holds the expected output fields.
- `tests/fixtures/macos_ws_monitor/ws_monitor_lines.json` holds three JSONL lines from a live monitor log.
  Personal paths in them are replaced with placeholders.
- `data/samples/ws_monitor_lines.json` holds the same three events. `default/samples.yml` lists it for the Cribl UI preview.
