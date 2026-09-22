# Cloud-init: Tools-Ready Lab Client

**Outcome:** A Linux VM boots on AWS, Azure, or GCP and finishes cloud-init as a
**tools-ready** client — devops packages, `labuser`, chrony, optional
fio/iperf/elbencho builds, and `~/tools` clones — **without running any I/O**.

Birth the host here. Wire mounts and day-2 polish with Ansible. Prove the
platform with fio. Scale multi-client tests with elbencho examples.

```text
Terraform apply  →  cloud-init  →  tools-ready client
                                      ↓
                              (you run I/O later)
```

**Source of truth:**
[Terraform/scripts/cloud-init](https://github.com/kvietmeier/Terraform/tree/master/scripts/cloud-init)

| File | Role |
|------|------|
| `lab_bootstrap.sh` | OS/cloud branching — **edit this** |
| `cloud-init-universal.yaml` | Committed render (for `file(...)` stacks) |
| `cloud-init-universal.yaml.tftpl` | Template for `templatefile(...)` |
| `render_cloud_init.sh` | Rebuild `.yaml` after script edits |

---

## What you get after first boot

| Layer | What | Where |
|-------|------|--------|
| Packages | vim, git, curl, python3, tmux, jq, htop, sysstat, numactl, … | YAML `packages:` |
| Lab user | `labuser` + passwordless sudo; cloud SSH key copied onto it | `/home/labuser` |
| Time | Cloud-aware chrony (AWS / Azure / GCP / OCI metadata) | chrony |
| Bench binaries (optional) | fio, iperf, dool, sockperf, elbencho | `/usr/local` (or RPM) |
| Tool repos (optional) | `sys-perf-tools`, `system-tools` | `/home/labuser/tools/` |

**Does not:** mount NFS, create `/mount/vast`, start elbencho `--service`, or
run fio. Those are day-2 / manual.

---

## Design (why it looks this way)

- **YAML** = portable packages + users + embed script.
- **`lab_bootstrap.sh`** = everything that branches by distro or cloud.
- Script is **base64-embedded** so indentation cannot corrupt the shell.
- Bench compiles **soft-fail** — one tool failing must not brick the boot.

Older per-cloud YAML copies live under `deprecated/` and are not maintained.

---

## Workflow A — existing Terraform stacks (`file(...)`)

Most modules point at the rendered file:

```hcl
cloudinit_configfile = "../../../scripts/cloud-init/cloud-init-universal.yaml"
```

```bash
terraform apply
```

When you change bootstrap behavior:

```bash
vim scripts/cloud-init/lab_bootstrap.sh
cd scripts/cloud-init && ./render_cloud_init.sh
git add lab_bootstrap.sh cloud-init-universal.yaml
```

---

## Workflow B — new modules (`templatefile`)

Embed the script at plan/apply — no render step:

```hcl
locals {
  ci = "${path.module}/../../../scripts/cloud-init"
}

data "cloudinit_config" "lab" {
  gzip          = false
  base64_encode = false

  part {
    content_type = "text/cloud-config"
    content = templatefile("${local.ci}/cloud-init-universal.yaml.tftpl", {
      bootstrap_b64 = base64encode(file("${local.ci}/lab_bootstrap.sh"))
    })
    filename = "lab.yaml"
  }
}
```

Attach `data.cloudinit_config.lab.rendered` as user-data / custom-data / metadata.

---

## Env knobs

Set on the guest before bootstrap (or wrap `runcmd`). Defaults target a
**benchmark lab image**.

| Variable | Default | Meaning |
|----------|---------|---------|
| `INSTALL_BENCH_TOOLS` | `true` | Compile fio / iperf / dool / sockperf / elbencho |
| `CLONE_LAB_SCRIPTS` | `true` | Clone helper scripts into `/home/labuser` |
| `CLONE_TOOLS_REPOS` | `true` | Clone `sys-perf-tools` + `system-tools` into `~/tools` |

Packages-only image (no compiles):

```bash
# example: wrap runcmd so the script sees the flag
INSTALL_BENCH_TOOLS=false bash /tmp/lab_bootstrap.sh
```

Standalone debug (as root on a running VM):

```bash
sudo bash /path/to/lab_bootstrap.sh
# logs: /tmp/lab_bootstrap-out.log  /tmp/cloud-init-out.txt
# marker: /root/CONFIGURED_BY_LAB_BOOTSTRAP
```

---

## Verify (SSH as `labuser`)

```bash
# cloud-init finished?
cloud-init status --wait
test -f /root/CONFIGURED_BY_LAB_BOOTSTRAP && echo bootstrap_ok

# binaries present (when INSTALL_BENCH_TOOLS=true)
command -v fio; fio --version | head -1
command -v elbencho; elbencho --version 2>/dev/null | head -1
command -v iperf3 || command -v iperf

# tool repos
ls -la ~/tools
ls ~/tools/sys-perf-tools/fio-file | head

# SSH key landed on labuser (Ansible target)
test -s ~/.ssh/authorized_keys && echo ansible_ssh_ok
```

**Pass criteria:** `fio` and `elbencho` on `PATH`, `~/tools/sys-perf-tools` present,
SSH to `labuser` works with the same key Terraform injected for the cloud
default user. No I/O required.

---

## What comes next

| Step | Where |
|------|-------|
| Day-2: users, `/mount/vast`, NFS driver | [ansible](https://github.com/kvietmeier/ansible) |
| Short proof: FIO smoke / baseline | [sys-perf-tools/fio-file](https://github.com/kvietmeier/sys-perf-tools/tree/master/fio-file) |
| Multi-client examples | `ansible/playbooks/elbencho.yml` |
| Gist index (this series) | [gist.github.com/kvietmeier](https://gist.github.com/kvietmeier) |

```bash
# after ansible mounts /mount/vast — first proof (manual)
cd ~/tools/sys-perf-tools/fio-file
./quick-smoke.sh /mount/vast/fio
```

---

## Author

Karl Vietmeier — Apache 2.0 (see Terraform repo)
