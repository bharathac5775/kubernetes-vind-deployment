# 🚀 Kubernetes Deployment with VIND — CineVerse Movie Website

A complete CI/CD pipeline that deploys a static movie website to a **virtual Kubernetes cluster** created by [**VIND (Virtual-cluster IN Docker)**](https://github.com/loft-sh/vind.git), all running inside a GitHub Actions runner — no cloud infrastructure needed.

---

## 📖 What is VIND?

**VIND** stands for **Virtual-cluster IN Docker**. It is a lightweight tool built on top of [vCluster](https://www.vcluster.com/) that spins up a **fully functional, certified Kubernetes cluster** inside a single Docker container. Think of it as "Kubernetes inside Docker" — but unlike other local K8s solutions, VIND creates **virtual clusters** that are isolated, disposable, and incredibly fast to boot.

VIND uses the **Docker driver** of vCluster to provision an entire Kubernetes control plane + worker node as a container on your machine (or in CI). There's no VM, no heavy hypervisor, and no special kernel requirements beyond a few standard network modules.

### 🔗 VIND Repository

> **GitHub:** [https://github.com/loft-sh/vind.git](https://github.com/loft-sh/vind.git)

---

## ⚔️ VIND vs kind vs Minikube — How Are They Different?

| Feature | **VIND (vCluster Docker)** | **kind** | **Minikube** |
|---|---|---|---|
| **Architecture** | Virtual cluster running inside a single Docker container | Kubernetes nodes as Docker containers | Full VM (or Docker container) with kubelet |
| **Startup Time** | ~30–60 seconds | ~60–90 seconds | ~2–5 minutes |
| **Resource Usage** | Very lightweight — shares host kernel, minimal overhead | Moderate — each "node" is a container | Heavy — runs a full VM by default |
| **Isolation** | Namespace-level virtual clusters; multiple clusters on one host easily | Container-level isolation | VM-level or container-level isolation |
| **Multi-Cluster** | Trivial — spin up multiple virtual clusters in seconds | Possible but heavier (each cluster = set of containers) | Possible via profiles, but slow and resource-heavy |
| **CI/CD Friendly** | Excellent — designed for ephemeral, disposable clusters in pipelines | Good — commonly used in CI | Usable but slower and more brittle in CI |
| **Kubernetes Certified** | Yes (CNCF conformant) | Yes (CNCF conformant) | Yes (CNCF conformant) |
| **Driver** | Docker | Docker | Docker, VirtualBox, HyperKit, KVM, etc. |
| **Best For** | CI/CD pipelines, dev/test, ephemeral environments, multi-tenancy | Local development, CI testing | Local development, full-cluster simulation |

### Why VIND Stands Out

- **Speed:** VIND clusters boot in seconds — ideal for CI pipelines where every minute counts.
- **Lightweight:** No VM overhead. The entire cluster lives inside a single container sharing the host kernel.
- **Ephemeral by design:** Create a cluster, run tests, deploy apps, tear it down — all in one pipeline run. Zero leftover resources.
- **Multi-tenancy ready:** Need 5 isolated environments? Spin up 5 virtual clusters on the same Docker host without multiplying resource usage.
- **Production-like:** Despite being lightweight, VIND provides a fully conformant Kubernetes API — your manifests, Helm charts, and kubectl commands work exactly as they would on a real cluster.

---

## 🖥️ How to Install VIND on Your System

### Prerequisites

- **Docker** must be installed and running on your machine.

### Step 1: Install the vCluster CLI

**macOS:**
```bash
brew install loft-sh/tap/vcluster
```

**Linux:**
```bash
curl -L -o vcluster "https://github.com/loft-sh/vcluster/releases/latest/download/vcluster-linux-amd64" && \
sudo install -c -m 0755 vcluster /usr/local/bin && \
rm -f vcluster
```

**Windows:**
Download the latest binary from [vCluster Releases](https://github.com/loft-sh/vcluster/releases) and add it to your PATH.

### Step 2: Configure vCluster to Use the Docker Driver (VIND Mode)

```bash
vcluster use driver docker
```

This tells vCluster to use Docker as the backend — which is what makes it **VIND**.

### Step 3: Create a Kubernetes Cluster

```bash
sudo vcluster create test-cluster
```

That's it! You now have a fully functional Kubernetes cluster running inside Docker. Verify with:

```bash
kubectl get nodes
vcluster ls
```

### (Optional) Create a Multi-Node Cluster

**Step 1:** Create a `values.yaml` file with your node configuration:

```yaml
experimental:
  docker:
    nodes:
    - name: "worker-1"
      ports:
        - "9090:9090"
    - name: "worker-2"
      volumes:
        - "/tmp/data:/data"
      env:
        - "NODE_ROLE=worker"
```

**Step 2:** If using volumes, make sure the mount path exists:

```bash
mkdir -p /tmp/data
```

**Step 3:** Create the multi-node cluster:

```bash
sudo vcluster create multi-node-cluster --values values.yaml
```

### Useful vCluster Commands

| Command | Description |
|---|---|
| `vcluster ls` | List all virtual clusters |
| `vcluster connect <name>` | Connect to a virtual cluster |
| `vcluster disconnect` | Disconnect from a virtual cluster |
| `vcluster delete <name>` | Delete a virtual cluster |
| `vcluster use driver docker` | Switch to Docker driver (VIND) |

> For more details, refer to the official VIND documentation: [https://github.com/loft-sh/vind](https://github.com/loft-sh/vind)

---

## 🎬 About This Repository

This project demonstrates a **real-world use case of VIND** — deploying a fully functional static website called **CineVerse** to a virtual Kubernetes cluster, entirely within a GitHub Actions CI/CD pipeline.

### What's Inside

```
├── index.html              # CineVerse website — curated movies & series
├── styles.css              # Dark-themed responsive stylesheet
├── Dockerfile              # nginx:alpine container serving static files
├── k8s/
│   ├── deployment.yaml     # Kubernetes Deployment manifest
│   └── service.yaml        # Kubernetes Service manifest
├── .github/
│   └── workflows/
│       └── deploy.yaml     # Full CI/CD pipeline using VIND
├── .dockerignore
└── README.md
```

### The Website — CineVerse

A beautiful, responsive, Netflix-style static movie website featuring:
- **Trending section** with the latest blockbusters
- **Top Rated Movies** — Shawshank Redemption, The Godfather, Interstellar, and more
- **Top Rated Series** — Breaking Bad, Game of Thrones, Stranger Things, and more
- **Genre browser**, newsletter signup, and a polished dark theme
- All poster images sourced from TMDB

---

## ⚙️ GitHub Actions Workflow — How It Works

The pipeline (`.github/workflows/deploy.yaml`) runs on every **push** or **pull request** to `main`. Here's what it does, step by step:

### 1️⃣ Build & Push Container Image
```
Checkout code → Build Docker image (nginx:alpine + static files) → Push to GHCR
```
The image is tagged with the short commit SHA and pushed to **GitHub Container Registry** (`ghcr.io`).

### 2️⃣ Create a VIND Cluster
```
Install vcluster CLI → Load kernel modules (overlay, bridge, br_netfilter) → vcluster create demo
```
This is the core step — VIND spins up a **virtual Kubernetes cluster** called `demo` right inside the GitHub Actions runner. The `vcluster use driver docker` command tells vCluster to use the Docker driver (VIND mode). Within ~30 seconds, you have a fully working `kubectl`-ready cluster.

### 3️⃣ Deploy the Application
```
Create GHCR pull secret → sed-inject image tag into manifest → kubectl apply → Wait for rollout
```
The deployment manifest uses `IMAGE_PLACEHOLDER` which gets replaced with the actual GHCR image path at runtime. Kubernetes pulls the image, starts the pod, and the site is live inside the cluster.

### 4️⃣ Expose via ngrok
```
Port-forward svc → Start ngrok tunnel → Print public URL → Keep alive for 10 minutes
```
Since the cluster runs inside CI, we use **ngrok** to create a temporary public tunnel. The workflow prints the live URL so you can open it in your browser and see the deployed website.

### 5️⃣ Cleanup
```
vcluster delete demo --force
```
The virtual cluster is destroyed at the end — leaving zero footprint on the runner.

### Pipeline Diagram

```
┌──────────┐    ┌──────────────┐    ┌──────────────────┐    ┌────────────┐    ┌───────────┐
│ Checkout │───▶│ Build & Push │───▶│ Create VIND      │───▶│ Deploy to  │───▶│ Expose    │
│   Code   │    │ Docker Image │    │ Cluster (demo)   │    │ Kubernetes │    │ via ngrok │
└──────────┘    └──────────────┘    └──────────────────┘    └────────────┘    └───────────┘
                                                                                    │
                                                                              ┌─────▼─────┐
                                                                              │  Cleanup   │
                                                                              │  (always)  │
                                                                              └───────────┘
```

---

## 🔑 Secrets Setup

The pipeline needs one secret to be configured manually. `GITHUB_TOKEN` is **already provided automatically** by GitHub Actions — you don't need to do anything for it.

| Secret | Required Action | Purpose |
|---|---|---|
| `GITHUB_TOKEN` | **None — auto-provided** | Used for GHCR login and image pull secrets. GitHub injects this automatically into every workflow run. |
| `NGROK_AUTH_TOKEN` | **Manual setup required** | Your ngrok auth token, needed to create the public tunnel to access the deployed site. |

### How to Get and Add `NGROK_AUTH_TOKEN` (Step by Step)

1. Go to [https://ngrok.com](https://ngrok.com) and **sign up** for a free account (or log in if you already have one).
2. After logging in, go to **Your Authtoken** page: [https://dashboard.ngrok.com/get-started/your-authtoken](https://dashboard.ngrok.com/get-started/your-authtoken).
3. Click **Copy** to copy your auth token.
4. Now go to **your GitHub repository** → click **Settings** (top tab).
5. In the left sidebar, expand **Secrets and variables** → click **Actions**.
6. Click the **"New repository secret"** button.
7. Set the **Name** to: `NGROK_AUTH_TOKEN`
8. **Paste** the auth token you copied from ngrok into the **Secret** field.
9. Click **"Add secret"**.

That's it! The secret is now available to your workflow.

---

## 🏃 How to Run

1. **Fork or clone** this repository.
2. **Set up the `NGROK_AUTH_TOKEN` secret** following the steps above.
3. **Push to `main`** or open a Pull Request — the pipeline triggers automatically.
4. Go to the **Actions** tab, open the running workflow, and wait for it to complete.
5. Open the **"🎬 Show public URL"** step in the workflow logs.
6. Click the ngrok URL — your CineVerse website is live!

---

## 📸 What You'll See

Once the pipeline completes, the workflow logs will show something like:

```
============================================
  🎬  YOUR WEBSITE IS LIVE!  🎬
============================================
  🌐  https://xxxx-xx-xxx-xxx-xx.ngrok-free.app
============================================
  Tunnel will stay open for ~10 minutes
============================================
```

Open the URL and you'll see the fully deployed CineVerse website running on a Kubernetes cluster that was created from scratch — all in under 5 minutes.

---

## 🛠️ Tech Stack

- **Website:** HTML5 + CSS3 (static, no frameworks)
- **Container:** nginx:alpine
- **Orchestration:** Kubernetes via VIND (vCluster Docker driver)
- **Registry:** GitHub Container Registry (GHCR)
- **CI/CD:** GitHub Actions
- **Tunnel:** ngrok v3
- **Images:** TMDB API (poster images)

---

## � Result Screenshots

### GitHub Actions Pipeline
![GitHub Actions Pipeline](result-images/github%20action%20pipline%20.png)

### GitHub Actions Pipeline — VIND Cluster Logs
![GitHub Actions Pipeline with VIND Logs](result-images/github%20action%20pipline%20with%20vind%20logs.png)

### Deployed CineVerse Website
![Static Website](result-images/Static%20website.png)

![Static Website Continued](result-images/Static%20website...png)

---

## �📝 License

This project is for educational and demonstration purposes.
