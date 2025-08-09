<p align="left">
  <img src="https://github.com/user-attachments/assets/2de4e418-8574-4b52-996c-3819f4f76abe" alt="headscale3-dots" width="160" />
</p>

# Headscale Self-Hosting Guide

## 📦 Introduction

This guide provides detailed instructions for setting up and configuring a self-hosted **Headscale** instance on your **Umbrel server**.

---

## ⚙️ Initial Setup After Installation

Once you’ve installed Headscale from the Umbrel App Store, a few configuration steps are required.

### 🔧 Configure Server Host Address

1. Open the **Files** app from the Umbrel dashboard.  
2. Navigate to: `apps → dasa-headscale`.  
3. Download the `exports.sh` file to your computer.  
4. Open it in a text editor and update the following line with your Umbrel's IP address or hostname:

   ```bash
   export UMBREL_HOST="172.17.0.2"  # your value may differ
   ```

5. Save the file and upload it back to the same location.

### 🛠️ Update `config.yaml`

1. Go to the folder: `apps → dasa-headscale → config`.  
2. Download the `config.yaml` file.  
3. Open it in a text editor.  
4. Locate the `server_url:` setting and replace `umbrel.local` with your actual Umbrel IP or hostname:

   ```yaml
   server_url: http://172.17.0.2:37070
   # Or if you prefer to use the local hostname:
   # server_url: http://umbrel.local:37070
   ```

5. Save the file and upload it back to the `config` folder.  
6. **Restart the Headscale app** from the Umbrel dashboard.

> ✅ Proper configuration is essential for Headscale to function correctly. Ensure both the server’s host and the client connection URL are properly set.

---

## 🔐 Generating an API Key

To create an API key for using the web interface, run the following command on your Umbrel host:

```bash
sudo docker exec -it dasa-headscale_headscale_1 headscale apikeys create
```

> ⚠️ **Important:** The key will only be shown once. Make sure to store it securely.  
> You can generate multiple API keys if needed.

---

## 🌐 Accessing the Web UI

The Headscale web interface can be accessed in your browser at:

```text
http://<YOUR-UMBREL-IP>:37070/web
http://<YOUR-UMBREL-IP>:37070/admin
```

Examples:

- Using IP address: `http://172.17.0.2:37070/web`  
- Using hostname: `http://umbrel.local:37070/admin`

---

## ➕ Adding Devices

For instructions on how to connect devices to your Headscale server, please refer to the official guide:

🔗 [Connecting Devices (Headscale Docs)](https://github.com/juanfont/headscale/tree/main/docs/usage/connect)

---
