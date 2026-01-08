---
link: https://enroll.sh/
excerpt: Harvest a host's real configuration and turn it into Ansible roles/playbooks. Safe-by-default, with optional SOPS encryption.
slurped: 2026-01-02T08:58
title: Enroll - Reverse-engineering servers into Ansible
tags:
  - ansible
---


```
# Harvest → Manifest in one go
enroll single-shot --harvest ./harvest --out ./ansible

# Then run Ansible locally
ansible-playbook -i "localhost," -c local ./ansible/playbook.yml
```

Good for

Disaster recovery snapshots, "make this one host reproducible", and carving a golden role set you'll refine over time.

---

Want templates for structured configs? Install [JinjaTurtle](https://git.mig5.net/mig5/jinjaturtle) and use `--jinjaturtle` (or let it auto-detect).

```
# Remote harvest over SSH, then manifest locally
enroll single-shot \
  --remote-host myhost.example.com \
  --remote-user myuser \
  --harvest /tmp/enroll-harvest \
  --out ./ansible \
  --fqdn myhost.example.com
```

If you don't want/need sudo on the remote host, add `--no-sudo` (expect a less complete harvest).

```
# Multi-site mode: shared roles, host-specific state in inventory
enroll harvest --out /tmp/enroll-harvest
enroll manifest --harvest /tmp/enroll-harvest --out ./ansible --fqdn "$(hostname -f)"

# Run the per-host playbook
ansible-playbook ./ansible/playbooks/"$(hostname -f)".yml
```

Rule of thumb: single-site for "one server, easy-to-read roles"; `--fqdn` for "many servers, high abstraction, fast adoption".

```
# Compare two harvests and get a human-friendly report
enroll diff --old /path/to/harvestA --new /path/to/harvestB --format markdown

# Send a webhook when differences are detected
enroll diff \
  --old /path/to/harvestA \
  --new /path/to/harvestB \
  --webhook https://example.net/webhook \
  --webhook-format json \
  --webhook-header 'X-Enroll-Secret: ...' \
  --exit-code
```