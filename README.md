# Private Minecraft Mod Server

Minecraft infrastructure using Kustomize overlays and ArgoCD.

## Structure

```
├── base/                    # Base Kubernetes resources
│   ├── namespace.yaml
│   ├── deployment.yaml      # Generic Minecraft server
│   ├── service.yaml
│   ├── persistent-volume-claim.yaml
│   └── kustomization.yaml
├── overlays/
│   ├── atm10sky/             # ATM10: To the Sky + server-compatible addons
│   ├── atm10/                # Previous ATM10 overlay (inactive)
│   ├── atm9/                 # Previous ATM9 overlay (inactive)
│   └── sample-mod/           # Sample modded server template
│       ├── kustomization.yaml
│       ├── deployment-patch.yaml
│       ├── service-patch.yaml
│       ├── pvc-patch.yaml
│       └── persistent-volume.yaml
├── argocd.yaml             # Self-managed ArgoCD application
└── kustomization.yaml      # Root configuration

```

## Legacy overlay template

**Option 1: Using the sample template**
1. Copy the sample-mod overlay:
   ```bash
   cp -r overlays/sample-mod overlays/new-server
   ```
2. Uncomment and modify the kustomization.yaml in the new overlay

**Option 2: Create from scratch**
1. Create a new overlay directory:
   ```bash
   mkdir overlays/new-server
   ```

2. Create `overlays/new-server/kustomization.yaml`:
   ```yaml
   apiVersion: kustomize.config.k8s.io/v1beta1
   kind: Kustomization
   
   namePrefix: new-server-
   commonLabels:
     app.kubernetes.io/instance: new-server
     minecraft-type: vanilla  # or modded
   
   resources:
     - ../../base
     - persistent-volume.yaml
   
   patches:
     - path: deployment-patch.yaml
       target:
         kind: Deployment
         name: minecraft-server
   ```

3. Create patches for server-specific configuration:
   - `deployment-patch.yaml` - Server type, version, memory, etc.
   - `service-patch.yaml` - NodePort configuration
   - `pvc-patch.yaml` - Storage requirements
   - `persistent-volume.yaml` - Local storage path

4. Add the new overlay to root `kustomization.yaml`:
   ```yaml
   resources:
     - argocd.yaml
     - overlays/create-astral
     - overlays/new-server  # Add this line
   ```

## Current Server

- `atm10sky`: Latest ATM10: To the Sky from CurseForge
- Minecraft 1.21.1 / NeoForge selected by the modpack
- Additional server-compatible mods and required libraries are declared in `overlays/atm10sky/deployment-patch.yaml`
- 12G heap, 14Gi request, 16Gi limit
- 100Gi local PV on the `private-minecraft` Kubernetes node
- NodePorts: 32568 (Minecraft), 31027 (RCON)

## Previous World

The previous ATM10 world was archived on `minecraft-node` before its PV was cleaned. The archive is not managed by Git.

## Deployment

The servers are automatically deployed via ArgoCD from this repository.
