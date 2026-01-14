# Deploying DeepTutor on Google Cloud Platform (GCP)

This guide provides two methods to deploy DeepTutor on Google Cloud Compute Engine (GCE):
1.  **Manual Deployment (Docker Compose):** Full control, standard Docker deployment.
2.  **Dokploy Deployment (Recommended):** Easier management via a web interface (similar to Vercel/Netlify).

---

## Prerequisites

1.  A Google Cloud Platform account.
2.  Billing enabled for your project.
3.  Google Cloud SDK installed (optional, can use Cloud Console).

---

## Method 1: Manual Deployment (Docker Compose)

### Step 1: Create a VM Instance

1.  Go to the [Google Cloud Console](https://console.cloud.google.com/).
2.  Navigate to **Compute Engine** > **VM instances**.
3.  Click **Create Instance**.
4.  **Configuration Recommendations:**
    *   **Name:** `deeptutor-server`
    *   **Region:** Choose one close to you (e.g., `asia-southeast1` for Singapore).
    *   **Machine Type:** `e2-standard-2` (2 vCPU, 8GB RAM) or higher. DeepTutor requires memory for RAG and processing.
    *   **Boot Disk:** Change to **Ubuntu 22.04 LTS**. Increase size to at least **50GB** (Standard Persistent Disk).
    *   **Firewall:** Check "Allow HTTP traffic" and "Allow HTTPS traffic".
5.  Click **Create**.

### Step 2: Configure Firewall Rules

DeepTutor uses port `3782` (Frontend) and `8001` (Backend). You need to open these ports.

1.  Go to **VPC network** > **Firewall**.
2.  Click **Create Firewall Rule**.
3.  **Name:** `allow-deeptutor`
4.  **Targets:** All instances in the network.
5.  **Source IPv4 ranges:** `0.0.0.0/0` (Allow access from anywhere).
6.  **Protocols and ports:** Check `tcp` and enter `3782,8001`.
7.  Click **Create**.

### Step 3: Setup the Server

1.  SSH into your VM instance (click the **SSH** button in the console).
2.  Clone the DeepTutor repository:
    ```bash
    git clone https://github.com/HKUDS/DeepTutor.git
    cd DeepTutor
    ```
3.  Run the setup script to install Docker and dependencies:
    ```bash
    chmod +x deploy/gcp/setup_vm.sh
    ./deploy/gcp/setup_vm.sh
    ```
4.  **Important:** Log out and log back in (close SSH window and reconnect) for permission changes to take effect.

### Step 4: Configuration

1.  Create your environment file:
    ```bash
    cp .env.example .env
    ```
2.  Edit `.env` with your API keys:
    ```bash
    nano .env
    ```
3.  **CRITICAL:** Set `NEXT_PUBLIC_API_BASE_EXTERNAL` to your VM's Public IP.
    *   Find your External IP in the VM instances list.
    *   Add/Update this line in `.env`:
        ```bash
        NEXT_PUBLIC_API_BASE_EXTERNAL=http://<YOUR_VM_EXTERNAL_IP>:8001
        ```
    *   *Note: If you are setting up a custom domain with SSL later, use `https://your-domain.com` instead.*

### Step 5: Launch

Start the application:

```bash
docker compose up -d --build
```

Access DeepTutor at: `http://<YOUR_VM_EXTERNAL_IP>:3782`

---

## Method 2: Dokploy Deployment (Easy Management UI)

[Dokploy](https://dokploy.com/) is a free, self-hostable alternative to Vercel/Heroku.

### Step 1 & 2: Create VM and Firewall

Follow **Step 1** and **Step 2** from the Manual Deployment method above.
*   *Addition:* Also open port `3000` in the Firewall rule (for Dokploy UI). So the ports list should be: `3000,3782,8001`.

### Step 3: Install Dokploy

1.  SSH into your VM.
2.  Run the installation command:
    ```bash
    curl -sSL https://dokploy.com/install.sh | sh
    ```
3.  Wait for the installation to finish.

### Step 4: Configure Dokploy

1.  Open your browser and go to: `http://<YOUR_VM_EXTERNAL_IP>:3000`
2.  Create an admin account.
3.  Go to **Projects** > **Create Project**.
4.  **Connect GitHub:** Go to settings and connect your GitHub account.
5.  **Create Application (Docker Compose):**
    *   Select **Compose** type.
    *   Select Repository: `HKUDS/DeepTutor`
    *   Branch: `main`
    *   Path: `./docker-compose.yml`
6.  **Environment Variables:**
    *   Copy content from your `.env` file into the Environment Variables section in Dokploy.
    *   Add `NEXT_PUBLIC_API_BASE_EXTERNAL=http://<YOUR_VM_EXTERNAL_IP>:8001`

### Step 5: Deploy

Click **Deploy**. Dokploy will handle pulling the code, building the image, and starting the containers. You can view build logs directly in the UI.

Access DeepTutor at: `http://<YOUR_VM_EXTERNAL_IP>:3782`
