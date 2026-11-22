# NextOps Frontend

## Industrial monitoring dashboard

The NextOps frontend is a React and Vite dashboard for monitoring an industrial thermal power environment. It presents live equipment telemetry, asset health, operating trends, alarms, plant topology, and AI-assisted operational insights in one control-room interface.

The dashboard is designed as a static web application: build the frontend once, copy the generated files to an Azure virtual machine, and serve them through Nginx.

## Azure VM deployment guide

This guide shows how to deploy the NextOps Dashboard on an Ubuntu VM in Azure using Node.js, Nginx, and static hosting from `/var/www/html`.

## What we are doing

The deployment follows a simple flow:

1. Allow browser traffic to reach the VM.
2. Install the tools needed to build the frontend.
3. Download and configure the NextOps frontend.
4. Connect the dashboard to the Query Service and Copilot APIs.
5. Create an optimized production build.
6. Serve the build through Nginx.
7. Open the public IP in a browser and verify the dashboard.

Each step is independent, but they must be completed in this order because the final dashboard needs both the VM web server and the backend API addresses.

---

## 1. Open Port 80 in Azure NSG

Before deployment, allow incoming HTTP traffic so users can open the dashboard from a browser:

1. Open the [Azure Portal](https://portal.azure.com).
2. Go to your Virtual Machine > **Networking** > **Network settings** (or your **Network Security Group**).
3. Under **Inbound port rules**, click **Add inbound rule**:
   - **Destination port ranges**: `80`
   - **Protocol**: `TCP`
   - **Action**: `Allow`
   - **Priority**: `1010`
   - **Name**: `Allow-HTTP-80`
4. Ensure **Port 22 (SSH)** is also allowed.

---

## 2. Connect to the Azure VM

From your local machine, connect to the Azure VM over SSH. Replace `<YOUR_AZURE_PUBLIC_IP>` with the public IP address shown in the Azure Portal:

```bash
ssh azureuser@<YOUR_AZURE_PUBLIC_IP>
```

---

## 3. Install Node.js & npm (v20 LTS)

The frontend uses Node.js to install packages and create the production build. Install Node.js 20.x LTS, which is recommended for Vite and React:

```bash
# Update system packages
sudo apt update && sudo apt install -y curl git

# Add NodeSource Node.js 20 repository
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -

# Install Node.js and npm
sudo apt install -y nodejs
```

Verify installed versions:
```bash
node -v   # Output should be v20.x.x
npm -v    # Output should be v10.x.x
```

---

## 4. Clone the Repository

Download the frontend source code to the VM and enter the project folder:

```bash
git clone <YOUR_REPO_URL>
cd nextops-dashboard
```

---

## 5. Update API Endpoint Details

The dashboard communicates with two backend services. Configure their public addresses so the browser can retrieve live data:

- **Query Service:** supplies device and telemetry data.
- **Copilot Service:** supplies the AI-powered operational assistant.

### Option A: Using a `.env` file (Recommended)

Create a `.env` file in the project root. This keeps environment-specific API addresses out of the source code:

```bash
nano .env
```

Add your backend API Public IP and port:

```env
VITE_API_BASE_URL=http://<YOUR_QUERY_SERVICE_PUBLIC_IP>:8000
VITE_COPILOT_API_BASE_URL=http://<YOUR_COPILOT_PUBLIC_IP>:8002
```

Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`).

### Option B: Directly edit `src/config/api.ts`

Use this option only if you prefer to configure the API addresses directly in the application:

```bash
nano src/config/api.ts
```

Update line 6 with your Query Service endpoint:

```typescript
export const API_BASE_URL = import.meta.env.VITE_API_BASE_URL || 'http://<YOUR_QUERY_SERVICE_PUBLIC_IP>:8000';
```

---

## 6. Install Dependencies & Build

Install the frontend dependencies, then build the optimized static website:

```bash
# Install npm dependencies
npm install

# Build static production assets
npm run build
```

This compiles the application and creates deployable static assets in the `./dist` folder.

---

## 7. Install Nginx

Install Nginx, enable it to start automatically, and start the web server:

```bash
sudo apt install -y nginx
sudo systemctl enable nginx
sudo systemctl start nginx
```

---

## 8. Copy Build Artifacts to `/var/www/html`

Publish the generated frontend files from the `dist` folder to Nginx's default website directory:

1. Remove default Nginx placeholder files:
   ```bash
   sudo rm -rf /var/www/html/*
   ```

2. Copy the compiled assets from `dist/` to `/var/www/html`:
   ```bash
   sudo cp -r dist/* /var/www/html/
   ```

---

## 9. Test in Browser

Open the dashboard in a browser using the VM public IP:

```text
http://<YOUR_AZURE_PUBLIC_IP>
```

If the page loads successfully, the NextOps dashboard is running through Nginx and should connect to the configured backend APIs.

