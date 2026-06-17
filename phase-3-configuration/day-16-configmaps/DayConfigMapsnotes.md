cat > ~/OneDrive/Desktop/kubernetes-learning/phase-3-configuration/day-16-configmaps/notes-claude.md << 'EOF'
# Day 16: ConfigMaps

## What Problem Does ConfigMap Solve?
Before ConfigMaps, application configuration was either:
- **Baked into the Docker image** → any config change = rebuild image = new pipeline run = slow and expensive
- **Scattered across files/folders** → no standard, hard to manage across environments
- **Hardcoded in YAML** → config ends up in Git, secrets exposed, no separation of concerns

**ConfigMap = Kubernetes' way of separating configuration from the container image.**
Same image + different ConfigMap = different behaviour in dev vs prod. No rebuild needed.

---

## What is a ConfigMap?
- A Kubernetes object that stores **non-sensitive** configuration data as key-value pairs
- Lives in the cluster, referenced by Pods at runtime
- `apiVersion: v1`, `kind: ConfigMap`
- Data stored under the `data:` field

---

## ConfigMap YAML Structure
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: default
data:
  APP_ENV: "production"
  APP_PORT: "8080"
  LOG_LEVEL: "info"
```

Key rules:
- Each config item is a **direct key-value pair** under `data:`
- Use `:` not `=` (YAML syntax, not shell syntax)
- Values should be quoted strings

---

## Four Ways to Consume a ConfigMap in a Pod
1. Inside a container command and args
2. **Environment variables** ← covered today
3. **Add a file in read-only volume** ← Day 19
4. Write code using the Kubernetes API directly

---

## Method 1: envFrom (Load ALL keys as env vars)
```yaml
envFrom:
  - configMapRef:
      name: app-config
```
- Loads **every key** from the ConfigMap as an environment variable
- Key names become env var names automatically
- Most common pattern in production when you own the ConfigMap
- Clean, minimal YAML

## Method 2: valueFrom (Load ONE specific key)
```yaml
env:
  - name: CUSTOM_VAR_NAME
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: APP_ENV
```
- Loads a single key from the ConfigMap
- Lets you **rename** the env var (CUSTOM_VAR_NAME ≠ APP_ENV)
- Used when a Pod needs specific keys from **multiple** ConfigMaps
- More verbose but more precise

---

## Today's Lab: Full Pod with envFrom
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-configmap
spec:
  containers:
    - name: app
      image: busybox:1.25
      command: ["sh", "-c", "echo APP_ENV=$APP_ENV && echo APP_PORT=$APP_PORT && echo LOG_LEVEL=$LOG_LEVEL && sleep 3600"]
      envFrom:
        - configMapRef:
            name: app-config
```

Verified with:
```bash
kubectl logs env-configmap
# Output:
# APP_ENV=production
# APP_PORT=8080
# LOG_LEVEL=info
```

---

## Break/Fix Lab: Missing ConfigMap
**Scenario:** ConfigMap name in Pod spec is wrong (`app-config-wrong` instead of `app-config`)

**What happens:**
- `kubectl apply` SUCCEEDS — Kubernetes accepts the Pod spec without validating if the ConfigMap exists
- Pod gets scheduled and image pulls successfully
- Container **never starts** — kubelet fails at inject-time
- Pod stays in `Pending` state, `ContainersReady: False`

**Error in `kubectl describe pod`:**

**Production impact:** If you delete a ConfigMap that running Pods depend on, those Pods will crash on next restart. This is a common production incident.

**Fix:** Correct the ConfigMap name in the Pod YAML, delete and redeploy the Pod.

---

## Key Debug Commands
```bash
# Check if ConfigMap exists and what it contains
kubectl get configmap app-config
kubectl describe configmap app-config

# Verify env vars reached the container
kubectl logs <pod-name>

# Debug a stuck Pod
kubectl describe pod <pod-name>   # check Events section at bottom
```

---

## Production Rules
- ConfigMaps are for **non-sensitive** data only (no passwords, tokens, keys)
- Secrets (Day 17) handle sensitive data
- `envFrom` = all keys, `valueFrom` = specific key with optional rename
- Deleting a ConfigMap while Pods reference it = Pods crash on restart
- Always fix YAML and redeploy — never patch in production (GitOps rule)
- `Optional: false` (default) = Pod fails if ConfigMap missing; `Optional: true` = Pod starts anyway with empty env vars

---

## Windows/CRLF Reminder
- Always check line endings: `cat -A file.yaml | head -5`
- Clean = lines end with `$`
- Broken = lines end with `^M$`
- Fix: VS Code status bar → click CRLF → change to LF → save
- Fallback: use `cat > file.yaml << 'EOF'` heredoc in terminal

---

## Interview Q&A

**Q: What is a ConfigMap and why use it?**
A: A ConfigMap is a Kubernetes object that decouples configuration from container images. Instead of baking config into the image or hardcoding it in Pod YAML, you store it in a ConfigMap and inject it at runtime. This means the same image can run in dev, staging, and prod with different config — no rebuild needed.

**Q: What's the difference between envFrom and valueFrom with configMapKeyRef?**
A: `envFrom` loads all keys from a ConfigMap as environment variables in one shot — clean and simple. `valueFrom.configMapKeyRef` loads a single specific key and lets you rename it. Use `envFrom` when you own the ConfigMap and want everything. Use `valueFrom` when you need specific keys from multiple ConfigMaps or want custom env var names.

**Q: What happens if a Pod references a ConfigMap that doesn't exist?**
A: `kubectl apply` succeeds — Kubernetes doesn't validate ConfigMap existence at apply time. The Pod gets scheduled and the image pulls, but the container never starts. The kubelet reports `Error: configmap "<name>" not found` in the Pod Events. The Pod stays in Pending state with `ContainersReady: False`.

**Q: What's the difference between ConfigMap and Secret?**
A: ConfigMap stores non-sensitive configuration (env names, ports, feature flags). Secret stores sensitive data (passwords, API keys, TLS certs) and is base64-encoded at rest with additional access controls. Never store passwords in ConfigMaps.

**Q: Can you update a ConfigMap without restarting Pods?**
A: It depends on how the ConfigMap is consumed. For environment variables (envFrom/valueFrom) — NO, the Pod must be restarted because env vars are injected at container start. For volume-mounted ConfigMaps — YES, Kubernetes automatically updates the file inside the container (with a short delay). This is a key reason why volume mounts are preferred for config that changes frequently.

**Q: What is Optional: false on a ConfigMap reference?**
A: It's the default behaviour — if the referenced ConfigMap doesn't exist, the Pod will not start. Setting `optional: true` allows the Pod to start even if the ConfigMap is missing, with empty env vars for those keys. Useful for optional feature flags.

---

## Day 16 Checklist
- [x] Understood why ConfigMaps exist (decouple config from image)
- [x] Created ConfigMap with key-value pairs under data:
- [x] Used envFrom to inject all keys as env vars into a Pod
- [x] Verified values with kubectl logs
- [x] Broke it: wrong ConfigMap name → container never starts
- [x] Read the error in Events: configmap "x" not found
- [x] Fixed it: correct name → Pod healthy again
- [x] Understood envFrom vs valueFrom difference

## Next: Day 17 — Secrets
Same concept as ConfigMap but for sensitive data. Base64 encoding, access controls, and why Secrets are not truly secure by default.
EOF