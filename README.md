# Couchbase Grafana Dashboard

[![License: MIT](https://img.shields.io/badge/License-MIT-EA2328.svg)](LICENSE)
[![Grafana](https://img.shields.io/badge/Grafana-9.0%2B-F46800.svg)](https://grafana.com/)
[![Couchbase](https://img.shields.io/badge/Couchbase-7.0%2B-EA2328.svg)](https://www.couchbase.com/)
[![Prometheus](https://img.shields.io/badge/Prometheus-2.40%2B-E6522C.svg)](https://prometheus.io/)

A comprehensive, Couchbase-branded Grafana dashboard for monitoring
Couchbase Server clusters via the Prometheus metrics endpoint.

> **Status:** community project. Not affiliated with Couchbase, Inc.

---

## What's in the box

- **`dashboards/couchbase-grafana-dashboard.json`** — the Grafana dashboard,
  68 panels across 10 functional sections, ready to import.
- **`docs/Couchbase-Grafana-Dashboard-Implementation-Guide.docx`** — a 30+
  page Word document with full installation, configuration, JSON reference,
  customization guide, and troubleshooting.
- **`docs/images/`** — PNG screenshots of each section (also embedded in
  the Word document).
- **`examples/`** — drop-in `prometheus.yml`, `docker-compose.yml`,
  Grafana provisioning YAML, and Prometheus alert rules.
- **`scripts/`** — the generators that produce the dashboard JSON, the
  Word document, and the screenshots, all reproducible from source.

## Sections covered

| # | Section | Metrics |
|---|---|---|
| 1 | Cluster & Node Health | `sys_cpu_*`, `sys_mem_*`, `sys_swap_*`, `sys_disk_*`, `sys_network_*`, `sysproc_*` |
| 2 | Data Service (KV) | `kv_ops`, `kv_cmd_*`, `kv_get_hits/misses`, `kv_curr_items`, `kv_ep_*`, `kv_mem_used_bytes`, `kv_dcp_*` |
| 3 | Query (N1QL) | `n1ql_active_requests`, `n1ql_queued_requests`, `n1ql_request_time`, `n1ql_errors`, `n1ql_warnings`, `n1ql_requests_*ms` |
| 4 | Index (GSI) | `index_memory_*`, `index_disk_size`, `index_num_docs_*`, `index_num_requests`, `index_avg_scan_latency`, `index_cache_hit_percent`, `index_resident_percent`, `index_frag_percent`, `index_num_scan_errors/timeouts` |
| 5 | Search (FTS) | `fts_total_queries*`, `fts_doc_count`, `fts_total_bytes_*`, `fts_num_bytes_used_*`, `fts_memory_quota` |
| 6 | Analytics (CBAS) | `cbas_incoming_records_count`, `cbas_failed_at_parser_count`, `cbas_pending_merge_ops`, `cbas_disk_used`, `cbas_heap_used` |
| 7 | Eventing | `eventing_on_update_*`, `eventing_on_delete_*`, `eventing_dcp_backlog`, `eventing_processed_count` |
| 8 | XDCR | `xdcr_docs_processed_total`, `xdcr_changes_left_total`, `xdcr_docs_failed_cr_source_total`, `xdcr_docs_filtered_total`, `xdcr_num_failedckpts_total` |
| 9 | Backup | `backup_location_total_size_bytes`, `backup_tasks_running` |
| 10 | Legacy `cb_*` (collapsed) | `cb_bucket_*`, `cb_node_*` from the older `couchbase-exporter` |

## Quick start

### 1. Make sure your Prometheus is scraping Couchbase

Couchbase Server 7.0+ ships a native Prometheus endpoint on every node. See
[`examples/prometheus.yml`](examples/prometheus.yml) for a complete scrape
config that uses HTTP service discovery via
`/prometheus_sd_config`.

You'll need a Couchbase user with the **External Stats Reader** role:

```bash
couchbase-cli user-manage \
  --cluster https://node1:18091 \
  --username Administrator --password "$CB_ADMIN_PASS" \
  --set --rbac-username prometheus \
  --rbac-password "$PROM_PASS" \
  --rbac-name 'Prometheus Scraper' \
  --roles external_stats_reader \
  --auth-domain local
```

### 2. Try the whole stack with Docker Compose

```bash
cd examples
cp ../dashboards/couchbase-grafana-dashboard.json grafana/dashboards/
docker compose up -d
```

Open http://localhost:3000 (admin / admin), and the dashboard is already
loaded.

### 3. Or import into an existing Grafana

1. **Dashboards → New → Import**
2. Upload `dashboards/couchbase-grafana-dashboard.json`
3. Select your Prometheus data source for the `DS_PROMETHEUS` variable
4. **Import**

## Variables

| Variable | Type | Behavior |
|---|---|---|
| `DS_PROMETHEUS` | datasource | Resolved at import time. |
| `$cluster` | query | Multi-select. Auto-populated from `sys_cpu_utilization_rate` labels. |
| `$node` | query | Multi-select. Cascades from `$cluster`. |
| `$bucket` | query | Multi-select. Cascades from `$cluster`. |

If your Prometheus uses different label names (`instance` instead of
`node`, etc.), see **§7.1 Renaming Variables and Labels** in the Word
document.

## Screenshots

| Cluster & Node Health | Data Service (KV) |
|---|---|
| ![](docs/images/02_cluster_health.png) | ![](docs/images/03_data_service_kv.png) |
| **Query Service (N1QL)** | **Index Service (GSI)** |
| ![](docs/images/04_query_n1ql.png) | ![](docs/images/05_index_gsi.png) |

See [`docs/images/`](docs/images/) for the full set.

## Documentation

The full implementation and configuration guide is in
[`docs/Couchbase-Grafana-Dashboard-Implementation-Guide.docx`](docs/Couchbase-Grafana-Dashboard-Implementation-Guide.docx).
It covers:

1. Executive summary
2. Architecture
3. Prerequisites
4. Installation (bare metal, Docker Compose, Kubernetes)
5. Dashboard walk-through (per-section screenshots)
6. JSON reference — what every field does
7. Customization (variables, thresholds, alerts, re-skinning)
8. Troubleshooting
9. References

## Building from source

The dashboard, screenshots, and Word document are all generated
programmatically. To rebuild everything:

```bash
# Dashboard JSON (Python 3.8+)
python3 scripts/build_dashboard.py

# Screenshots (Python with matplotlib + numpy)
pip install matplotlib numpy
python3 scripts/generate_screenshots.py

# Word document (Node.js 18+ with docx-js)
npm install -g docx
node scripts/generate_doc.js
```

## Contributing

Pull requests welcome. Please:

1. Open an issue first if you're adding a section or making non-trivial
   changes to the JSON.
2. Re-run `scripts/build_dashboard.py` after editing the generator — do
   not hand-edit `dashboards/couchbase-grafana-dashboard.json`.
3. Keep brand styling within the Couchbase palette (red `#EA2328`,
   deep red `#B81E22`, charcoal `#1A1A1A`, etc.).

## License

MIT — see [LICENSE](LICENSE).

Couchbase® is a registered trademark of Couchbase, Inc. This project is
an unofficial community effort and is not affiliated with, endorsed by,
or sponsored by Couchbase, Inc.
