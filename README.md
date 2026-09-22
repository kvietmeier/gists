# Local gist workspace

Clones of public gists + drafts for the DevOps CV series.

## Published (git remotes → gist.github.com)

| Dir | Gist | Description |
|-----|------|-------------|
| `ansible-adhoc/` | [d497360…](https://gist.github.com/kvietmeier/d497360a0e304286285220066ff0b53c) | Ansible Ad Hoc Commands |
| `vagrant-windows11/` | [3f296be…](https://gist.github.com/kvietmeier/3f296be759f201d7431cf3af4d53c2db) | Vagrant on Windows 11 |
| `terraform-multi-vm/` | [dd23ca3…](https://gist.github.com/kvietmeier/dd23ca3e8792e6f4f4e0571171aef55c) | Terraform multi-VM demo |

Each subdirectory is its **own** git repo. Edit → `git commit` → `git push` updates the gist (requires `gh auth login` / SSH to GitHub).

## Drafts (not published yet)

See `drafts/` — mirrored from `personal/docs/gists/`.

```bash
# publish a new gist when ready
gh gist create drafts/00-index.md -d "DevOps Toolkit — Public Gists" -p
gh gist create drafts/01-cloud-init-tools-ready-client.md -d "Cloud-init: Tools-Ready Lab Client" -p
```

Profile: https://gist.github.com/kvietmeier
