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
│   └── create-astral/       # Create Astral modpack server
│       ├── kustomization.yaml
│       ├── deployment-patch.yaml
│       ├── service-patch.yaml
│       ├── pvc-patch.yaml
│       └── persistent-volume.yaml
├── argocd.yaml             # Self-managed ArgoCD application
└── kustomization.yaml      # Root configuration

```

## Adding a New Minecraft Server

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

- **create-astral**: Create Astral modpack (Forge 1.20.1)
  - Memory: 14Gi / 8 CPU
  - Storage: 50Gi
  - Ports: 30565 (game), 30575 (RCON)

## Deployment

The servers are automatically deployed via ArgoCD from this repository.