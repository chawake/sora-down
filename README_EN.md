# Sora Video Watermark-Free Link Extractor (Sora Video Downloader Web UI)

<img width="867" height="537" alt="image" src="https://github.com/user-attachments/assets/458b4132-f26e-4fa5-a87b-1c26890403fc" />

## ✨ Features

- **Easy to Use**: Simply paste a Sora link to get the direct download address.
- **Watermark-Free Videos**: Extracts the original video link from `encodings.source.path`.
- **Long-Term Stable Operation**: Built-in `access_token` auto-refresh mechanism, automatically renews when expired, no manual intervention required.
- **Docker Deployment**: One-click build and run, no need to worry about environment configuration.
- **Environment Variable Configuration**: All sensitive information and configuration are managed through `.env` file, secure and convenient, supports hot updates.
- **Proxy Support**: Can set HTTP/HTTPS proxy for OpenAI API requests through `HTTP_PROXY` environment variable.
- **Optional Access Protection**: Can set `APP_ACCESS_TOKEN` to add a layer of password protection to your web service to prevent abuse.

## 🛠️ Tech Stack

- **Backend**: Python, Flask, Gunicorn
- **HTTP Requests**: `curl-cffi` (used to simulate browser TLS/JA3 fingerprint, improving request success rate)
- **Frontend**: Native HTML, CSS, JavaScript
- **Configuration Management**: `python-dotenv`
- **Deployment**: Docker

## 🚀 Quick Start

### 1. Prerequisites

- Docker or Python installed

### 2. Get OpenAI Authentication Credentials
This is the most crucial step, you need to obtain `SORA_AUTH_TOKEN` (short-term valid) or `SORA_REFRESH_TOKEN` (long-term valid).

#### Method One (Recommended): Android (Root) + Packet Capture
**This method requires a rooted Android device and some hands-on ability.**

**Core Idea**: Bypass the App's SSL Pinning through Hook tools in a Root environment, then use packet capture tools to capture the App's network requests, thereby obtaining all credentials needed for authentication.

**Tool Preparation:**
* A rooted Android device (usually using Magisk).
* LSPosed framework (installed in Magisk).
* Packet capture tool: **[Reqable](https://reqable.com/)** (recommended for PC use).
* SSL Pinning bypass module: **`TrustMeAlready`** (an LSPosed module).

**Operation Steps:**

1. **Prepare Android Environment:**
   * Ensure your device is rooted and has LSPosed framework installed.
   * To make Sora App run normally in a Root environment, you may need to pass Google's SafetyNet / Play Integrity checks. You can refer to this tutorial on Coolapk for configuration: [Very Easy Google Three Green Tutorial](https://www.coolapk.com/feed/68354277?s=NGRlYjI5NjQxNmI5MDZnNjkwYjE5Yzl6a1571).

2. **Install and Enable Bypass Module:**
   * Install `TrustMeAlready` module in LSPosed Manager.
   * Activate the module and ensure its scope includes **Sora App**.
   * Restart the phone to make the module take effect.

3. **Configure Packet Capture Tool (Reqable):**
   * Install and run `Reqable` on your computer.
   **Important Network Configuration**: If your computer needs proxy software (such as Clash, V2RayN, etc.) to access the external network, please make the following settings:
       * Enable "Allow LAN Connection" or similar option in your proxy software.
       * In `Reqable` settings, configure "Upstream Proxy" to point to the HTTP port provided by your computer's proxy software (for example `http://127.0.0.1:7890`). This way, `Reqable` can forward the phone's traffic through the computer's proxy to the external network.
   * Ensure the phone and computer are connected to the same Wi-Fi network.
   * Follow `Reqable`'s guide to configure HTTP proxy in the phone's Wi-Fi settings, pointing to your computer's IP and `Reqable`'s port.
   **Install Certificate**: Visit `reqable.pro/ssl` in the phone browser to download the certificate. Since the device is rooted, it's recommended to install this certificate as a **system certificate** for best packet capture results.

4. **Capture Authentication Credentials:**
   * Start packet capture in `Reqable` on your computer.
   * Open Sora App on your phone and perform **login operation**.
   * In `Reqable`'s request list, find a **POST** request sent to `auth.openai.com/oauth/token`.
   **View the "Response Body" of this request:**
       * `client_id`: Copy this value and fill it into the `SORA_CLIENT_ID` in the `.env` file.
       * `refresh_token`: Copy this value and fill it into the `SORA_REFRESH_TOKEN` in the `.env` file.

> **⚠️ Important Notes**:
> - `refresh_token` is relatively long-term valid but refreshes after each use. Please properly keep both the initial and latest `refresh_token`.
> - This operation involves Root and system modifications and has risks. Please operate with caution.
> - Please properly keep these credentials and do not leak them to others.

#### Method Two: iOS (Jailbreak)
This method requires a jailbroken iOS device. For specific tutorials, you can refer to the following projects:
- [iOS Packet Capture Tutorial (devicecheck)](https://github.com/qy527145/devicecheck)
- I don't have Apple devices, so I can't test if iOS is available in this project. Welcome friends to provide feedback and submit PRs.

### 3. Download and Configure Project

1. Clone this project to your server or local:
   ```bash
   git clone https://github.com/tibbar213/sora-downloader.git
   cd sora-downloader
   ```
2. Copy environment variable example file:
   ```bash
   cp .env.example .env
   ```
3. Edit the `.env` file, fill in the credentials you obtained in the previous step, and set `APP_ACCESS_TOKEN` and `HTTP_PROXY` as needed.
   ```ini
   # --- OpenAI Sora API Authentication ---
   SORA_AUTH_TOKEN="paste_your_access_token_here" # priority use
   SORA_REFRESH_TOKEN="paste_your_refresh_token_here"
   SORA_CLIENT_ID="paste_your_client_id_here"

   # --- App Protection (Optional) ---
   APP_ACCESS_TOKEN="set_your_own_access_password"

   # --- Network Proxy (Optional) ---
   HTTP_PROXY="http://your_proxy_address:port"
   ```

### 4. Build and Run Docker Container

In the project root directory, run the following commands:

1. **Build Docker Image:**
   ```bash
   docker build -t sora-downloader .
   ```

2. **Run Docker Container:**
   ```bash
   docker run -d -p 5000:8000 \
     -v $(pwd)/.env:/app/.env \
     --name sora-downloader \
     sora-downloader
   ```
   **Command Explanation:**
   - `-d`: Run container in background.
   - `-p 5000:8000`: Map your machine's `5000` port to the container's `8000` port. You can change `5000` to any unused port.
   - `-v $(pwd)/.env:/app/.env`: **(Key)** Mount the `.env` file on your host machine inside the container. This allows Token auto-refresh to write new values back to your `.env` file, achieving persistence.
       * *Windows PowerShell users please use `-v ${PWD}/.env:/app/.env`*
       * *Windows CMD users please use `-v %cd%\\.env:/app/.env`*
   - `--name sora-downloader`: Specify a name for the container for easy management.

### 5. Access Service

Open your browser and visit `http://localhost:5000` (or your set server IP and port). Now you can start using it!

## ⚙️ Configuration (`.env` file)

This project is configured through the `.env` file in the root directory:

| Variable Name | Required | Description |
| --- | --- | --- |
| `SORA_AUTH_TOKEN` | **Yes** | Authorization token (Access Token) used to make requests to Sora API. If left empty but `SORA_REFRESH_TOKEN` is provided, the program will automatically obtain it on startup. |
| `SORA_REFRESH_TOKEN` | **No** (for implementing auto-renewal) | Token (Refresh Token) used to refresh `SORA_AUTH_TOKEN` when it expires. |
| `SORA_CLIENT_ID` | **No** (for implementing auto-renewal) | OpenAI OAuth client ID, obtained together with `refresh_token` during packet capture. |
| `APP_ACCESS_TOKEN` | No | Access token used to protect this web service. If set, the front-end page will require entering this token. |
| `HTTP_PROXY` | No | HTTP/HTTPS proxy for requesting OpenAI API. Required if your server network is restricted. Example: `http://127.0.0.1:7890` |

## 🌟 Recommended Projects
- **[sora2api](https://github.com/TheSmallHanCat/sora2api)**: A free, unofficial, reverse-engineered Sora API project. Already adapted to this project's interface, you can choose custom parsing interface in its watermark removal configuration and fill in this project's address.

## 📄 Disclaimer

- This project is for technical learning and personal research use only.
- Please comply with OpenAI's terms of service.
- Users are responsible for any consequences arising from the use of this tool.
