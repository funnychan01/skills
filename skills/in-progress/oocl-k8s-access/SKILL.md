---
name: oocl-k8s-access
description: Reach OOCL Kubernetes clusters with kubectl from WSL — inspect DEV pods, read logs, open a shell. Use when a task needs kubectl against an OOCL Rancher context (nonprod-fl-ooclhomea and friends), pod status, pod logs, or exec into a deployed pod. Pins the kubeconfig and context on every command so a run cannot drift onto a production cluster.
---

# OOCL kubectl access

Reach OOCL clusters from this WSL2 machine. Two habits make it safe: **pin** the kubeconfig and context on every command, and **verify** before you act. To pin is to pass `--kubeconfig` and `--context` explicitly each time, so no command depends on — or changes — the machine's global default.

## Paths

Read the binary and kubeconfig locations from `C:\k8s\k8s.env`; they drift, so never hardcode them. From WSL that file is `/mnt/c/k8s/k8s.env`. It exports two paths that are deliberately different:

- `KUBECTL_BIN` — the **WSL** path used to *execute* the binary (`/mnt/c/k8s/kubectl.exe`).
- `KUBECONFIG_LOCAL` — the **Windows** path passed *as the value of* `--kubeconfig` (`C:\k8s\kubeconfig.yaml`).

`kubectl.exe` is a Windows binary: it opens the kubeconfig by its Windows path and fails with Access denied on a `/mnt/c/...` path. So the binary is exec'd via WSL, while `--kubeconfig` receives the Windows form.

## Steps

1. **Load paths.** `source /mnt/c/k8s/k8s.env`. Done when `KUBECTL_BIN` and `KUBECONFIG_LOCAL` are both set.
2. **Verify the target context exists.**
   `"$KUBECTL_BIN" --kubeconfig "$KUBECONFIG_LOCAL" config get-contexts -o name`
   Done when the intended context (e.g. `nonprod-fl-ooclhomea`) is listed. If it is missing, the kubeconfig drifted — go to Recovery.
3. **Pin and probe read-only.** Confirm reachability with a cheap read before anything that changes state:
   `"$KUBECTL_BIN" --kubeconfig "$KUBECONFIG_LOCAL" --context nonprod-fl-ooclhomea get pods -n <namespace>`
   Done when the pod list returns. Every later command repeats these same two pinned flags.
4. **Act.** Run the intended command with the pinned flags. A mutating command (`delete`, `scale`, `exec` that changes something) is run only after the step-3 probe has confirmed the right cluster and namespace.

## Guardrails

- **Pin the context on every command** with `--context <name>`. The kubeconfig's current-context may point at production, so the explicit flag is what holds a command on the intended cluster.
- **Target nonprod** (`nonprod-fl-ooclhomea`) for routine work. Production contexts (`prod-fl-ooclhomea`, `prod-cw-ooclhomea`) are reached only on explicit human instruction that names the cluster.
- **Read-only before mutating**: a `get`/`describe` confirming cluster and namespace precedes any state change.

## Recovery (drifted kubeconfig)

When the pinned path fails or the context is missing:
1. Enumerate candidates: `ls /mnt/c/k8s/`, `find /mnt/c/Users/<user> -iname '*kubeconfig*'`.
2. Pick the file that exposes the target context: `"$KUBECTL_BIN" --kubeconfig <candidate> config get-contexts`.
3. Update `C:\k8s\k8s.env` to the winning path, then resume at step 2.

## Project specifics

Cluster-and-namespace facts live beside each repo, not here. For **iba_remark_agent**, see `docs/agents/dev-kubectl.md` (namespace `iba-dev-d`, pod prefix `iba-remark-agent-`, container-name rule). The all-projects human cheat-sheet is `C:\k8s\kubectl.md`.
