# debug-platform

Throwaway test namespaces for the debug agent. Everything here is the guardrail
side; the `debug-*` namespaces themselves are created with kubectl and are not
in git.

- `rbac.yaml`: `debug-agent` (the identity the agent uses), `debug-ns-admin`
  (what it can do inside a debug namespace), `debug-controller`.
- `admission-policy.yaml`: the agent can only touch `debug-*` namespaces, must
  set a `debug.firekatt.ca/ttl` (hours, max 720), may only publish hosts named
  `<namespace>[-x].k8s.firekatt.ca`, and CNPG must use `longhorn-debug` at 5Gi or less.
- `controller.yaml`: reaps expired namespaces and gives new ones a quota, limits
  and the agent RoleBinding.
- `storageclass.yaml`: single-replica `longhorn-debug`, never backed up.

Why `<namespace>.k8s.firekatt.ca` and not `<name>.debug.k8s.firekatt.ca`: the
wildcard cert only covers one label.
