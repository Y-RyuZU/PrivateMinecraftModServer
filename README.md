# Private Minecraft Mod Server

Multi-server Minecraft infrastructure using Kustomize overlays.

## Structure

```
├── base/                    # Base Kubernetes resources
│   ├── namespace.yaml
│   ├── deployment.yaml      # Generic Minecraft server
│   ├── service.yaml
│   ├── persistent-volume-claim.yaml
│   └── kustomization.yaml
├── overlays/
│   ├── create-astral/       # Create Astral modpack server
│   │   ├── kustomization.yaml
│   │   ├── deployment-patch.yaml
│   │   ├── service-patch.yaml
│   │   ├── pvc-patch.yaml
│   │   └── persistent-volume.yaml
│   └── sample-mod/          # Sample modded server template (commented)
│       ├── kustomization.yaml
│       ├── deployment-patch.yaml
│       ├── service-patch.yaml
│       ├── pvc-patch.yaml
│       └── persistent-volume.yaml
├── argocd.yaml             # Self-managed ArgoCD application
└── kustomization.yaml      # Root configuration

```

## Adding a New Minecraft Server

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

## Current Servers

- **create-astral**: Create Astral modpack using pre-built container
  - Image: `ghcr.io/claraphyll/create-astral:latest`
  - Memory: 14Gi / 8 CPU
  - Storage: 50Gi
  - Service: LoadBalancer (direct WAN access)
  - RCON: Enabled (password: rcon123)

## Deployment

The servers are automatically deployed via ArgoCD from this repository.