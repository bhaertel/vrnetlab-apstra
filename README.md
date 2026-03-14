# Juniper Apstra — vrnetlab / srl-labs container
## Requirements

Ue in combinatio with the srl-labs vrnetlab fork: https://github.com/srl-labs/vrnetlab

## Build instructions

```bash
# 1. Place your qcow2 in the apstra/ directory
cp /path/to/aos_server_6.1.1-70.qcow2 apstra/

# 2. Build
cd apstra/
make

# Resulting image tag: vrnetlab/juniper_apstra:6.1.1-70

# 3. (Optional) push to a private registry
DOCKER_REGISTRY=myregistry.example.com:5000/vrnetlab make docker-push
```

## Test version extraction before building

```bash
make version-test IMAGE=aos_server_6.1.1-70
# Expected output: 6.1.1-70
```

## Containerlab topology example

```yaml
# apstra-lab.clab.yaml
name: apstra-lab

topology:
  nodes:
    apstra:
      kind: linux
      image: vrnetlab/juniper_apstra:6.1.1-70
      env:
        QEMU_MEMORY: "16384"   # 16 GB minimum; increase to 32768 for production
        QEMU_SMP: "4"          # vCPU count
        # Uncomment for pass-through management (Apstra sees the real clab IP):
        # CLAB_MGMT_PASSTHROUGH: "true"
      ports:
        - "22:22"     # SSH CLI access
        - "80:80"     # HTTP (redirects to HTTPS)
        - "443:443"   # Web UI + REST API
        - "5000:5000" # QEMU serial console (debug only)
```

Deploy:
```bash
sudo containerlab deploy -t apstra-lab.clab.yaml
```

Access the Web UI at `https://<host-ip>` (or the management IP shown by
`sudo containerlab inspect -t apstra-lab.clab.yaml`).

## Debugging

**Serial console** (while container is running):
```bash
telnet localhost 5000
# or, using the container name:
docker exec -it clab-apstra-lab-apstra telnet localhost 5000
```

**Container logs** (boot progress from launch.py):
```bash
docker logs -f clab-apstra-lab-apstra
```

**Health status**:
```bash
docker inspect --format='{{.State.Health.Status}}' clab-apstra-lab-apstra
```
Expected progression: `starting` → (8–10 min) → `healthy`
