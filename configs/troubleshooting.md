# Troubleshooting Log

| Problem | Error Message | Solution Tried | Status |
|---------|--------------|----------------|--------|
| Prometheus service not starting | "exit code too quickly" | Fixed service file (removed placeholder `yourusername`) | ✅ Solved |
| Grafana failing to load | start request repeated | Corrected service user & paths | ✅ Solved |
| Prometheus targets show UNKNOWN | "context deadline exceeded" | Restarted service, waited for refresh | ✅ Solved |
| UFW blocking ports | Node Exporter not reachable | Opened 9090, 9100, 9182, 3000 | ✅ Solved |
| Windows node not scraping | "context deadline exceeded" | Opened firewall ports, disabled Norton temporarily | 🔴 Pending |

Notes:
- Norton antivirus may interfere with port 9182.
- Ping (ICMP) rules were added, but still unreachable from Linux.
- Next: Try adding Norton exclusions or running exporter as admin.
