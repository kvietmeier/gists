# DevOps Toolkit — Public Gists

Hands-on notes from building multi-cloud lab clients and storage benchmark
platforms. Short narrative, copy-paste commands, clear outcomes — written the
way I work: birth the VM, wire the lab, prove the platform, then scale the test.

**Profile:** [gist.github.com/kvietmeier](https://gist.github.com/kvietmeier)

```text
cloud-init  →  tools-ready client
ansible     →  day-2 lab / cluster wiring
fio         →  short demos & baselines
elbencho    →  multi-client benchmark examples
```

---

## Published

| Gist | Outcome |
|------|---------|
| [Ansible Ad Hoc Commands](https://gist.github.com/kvietmeier/d497360a0e304286285220066ff0b53c) | Learn Ansible from primitives before playbooks |
| [Vagrant on Windows 11](https://gist.github.com/kvietmeier/3f296be759f201d7431cf3af4d53c2db) | Local hypervisor labs without a cloud bill |
| [Terraform multi-VM demo](https://gist.github.com/kvietmeier/dd23ca3e8792e6f4f4e0571171aef55c) | IaC path from AWS Charging DB → Azure test bed |

## Planned / draft

| Draft | Outcome | Source of truth |
|-------|---------|-----------------|
| [Cloud-init tools-ready client](https://gist.github.com/kvietmeier/80a377a5a41d7764f755a1e836a0a194) | Boot → packages + fio/elbencho binaries + `~/tools` — **no auto I/O** | `Terraform/scripts/cloud-init/` |
| FIO smoke / baseline | First proof the mount and platform work | `sys-perf-tools/fio-file/` |
| Ansible day-2 | Users, `/mount/vast`, NFS driver, lab polish | `ansible/` |
| Elbencho examples | Multi-client service + example scripts you extend | `ansible/playbooks/elbencho.yml` |
| [Toolkit index](https://gist.github.com/kvietmeier/bb24a7e0500bf3ab106edb875eaf4957) (this page) | CV landing page | `~/github/gists/` |

---

## Repos (full implementations)

| Repo | Role |
|------|------|
| [Terraform](https://github.com/kvietmeier/Terraform) | Multi-cloud VMs + cloud-init |
| [ansible](https://github.com/kvietmeier/ansible) | Day-2 client + elbencho examples |
| [sys-perf-tools](https://github.com/kvietmeier/sys-perf-tools) | FIO jobfiles / smoke / baseline |
| [system-tools](https://github.com/kvietmeier/system-tools) | Host helpers / shell env |
| [cloud-tools](https://github.com/kvietmeier/cloud-tools) | Cloud CLI utilities |

---

## Ownership boundary

| Layer | Owns | Does **not** |
|-------|------|----------------|
| Terraform + cloud-init | VM, packages, fio/elbencho **binaries**, initial `~/tools` clone | Run I/O |
| ansible | Lab users, `/mount/vast`, VAST NFS driver, elbencho **examples** | Replace cloud-init |
| sys-perf-tools | FIO smoke / baseline jobfiles | Fleet config |
| system-tools | Shell env / host helpers | Bench orchestration |

Mount convention: **`/mount/vast`** (never `/mnt/vast`). Default FIO work dir: `/mount/vast/fio`.

---

## Author

Karl Vietmeier — KCV Consulting · Apache 2.0 where noted in repos
