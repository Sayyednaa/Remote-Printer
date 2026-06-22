# WireToWeb - Cloud Printing Solution for Legacy & Non-Wi-Fi Printers

WireToWeb is a secure, self-hosted cloud printing middleware platform that bridges physical, local-only (USB, parallel, or non-Wi-Fi) desktop printers to the web. It enables users to submit files and design custom page layouts from anywhere in the world and have them print silently on their local hardware without complex network configurations, VPNs, or port forwarding.

---

## 💡 The Problem

Most legacy and office printers lack wireless interfaces, native web APIs, or cloud capabilities. Connecting them to the internet to support remote print requests (e.g., printing order receipts from an e-commerce backend, or printing shipping labels from a remote warehouse) traditionally requires:
- Exposing local networks to the public internet (port forwarding/DMZ).
- Setting up static IP addresses or DDNS.
- Configuring complex VPN tunnels or expensive proprietary enterprise print servers.
- Manually compiling document drafts on the target computer before printing.

## 🚀 The Solution: WireToWeb

WireToWeb solves this by splitting the printing pipeline into a secure, public-facing web hub and an outbound-only polling desktop agent.

```
┌─────────────────────────────────┐
│     Any Device Worldwide        │
│  (Laptop, Tablet, Mobile, Web)   │
└────────────────┬────────────────┘
                 │
                 │ 1. Uploads PDF or designs on A4 Canvas
                 ▼
┌─────────────────────────────────┐
│     WireToWeb Web Server        │
│  (Centralized Django Queue Hub) │
└────────────────┬────────────────┘
                 ▲
                 │ 2. Outbound HTTP short-polling (Secure Queue)
                 │    No incoming ports open on local router!
┌────────────────┴────────────────┐
│      Windows Desktop Agent       │
│  (Local daemon running win32)   │
└────────────────┬────────────────┘
                 │
                 │ 3. silent native print command (USB / spooler)
                 ▼
┌─────────────────────────────────┐
│     Legacy / Non-Wi-Fi Printer  │
│  (HP LaserJet, USB Thermal, etc)│
└─────────────────────────────────┘
```

1. **Clay-Inspired Web Workspace**: A centralized hub where users register user profiles, manage active printers, configure print jobs (copies, duplex, orientation, quality), upload files, and compose documents inside an interactive A4 Page Editor.
2. **Outbound polling Desktop Agent**: A lightweight background daemon that runs on the Windows computer connected to the physical printer. It discovers local spoolers, registers them on the hub, and securely polls the server queue for print jobs using outbound-only HTTP/HTTPS requests.
3. **Silent Hardware execution**: The agent pulls down the document payload and prints it silently using local Windows print drivers (assisted by robust helpers like `PDFtoPrinter.exe`).

---

## 🎨 Interactive A4 Page Editor

The front-end includes a complete **A4 Canvas Designer** for building custom pages (shipping labels, receipts, collage, or photo layout cards) directly inside the browser:
- **Dynamic Elements**: Drag, drop, scale, and layer text boxes and image uploads.
- **Layer Reordering**: Move elements forward or backward step-by-step using DOM-swapping.
- **Freeform Image Cropping**: Crop images to any rectangular frame using a custom resizable crop box overlay and crop handles.
- **WYSIWYG Compiles**: Compiles layout views client-side using `html2canvas` and `jsPDF` into multi-page print-ready PDFs.

---

## 🔒 Security & Privacy

- **Outbound-Only Connection**: The desktop agent communicates exclusively via outbound HTTP/HTTPS requests. The local network's router configuration remains locked—no port forwarding, VPN configuration, or static IPs are required.
- **Data Protection (Zero-Retain)**: To protect personal and corporate document privacy, PDF payloads are automatically purged from the server immediately after successful delivery to the printer spooler. Cancelled or failed print jobs are auto-deleted after 24 hours.

---

## 🛠️ Repository Architecture

- `server/`: Central Django 6.0 web application. Contains models (`Printer`, `Document`, `PrintJob`), forms, CSS styling (`style.css`), AJAX polling views, and canvas scripts (`canvas.js`).
- `agent/`: Local Windows daemon (`agent.py`) using `win32print` queues and background polling workers.

---

## 🚀 Getting Started

### 1. Launch the Django Web Server
1. Navigate to the `server/` directory:
   ```bash
   cd server
   ```
2. Initialize virtual environment and start the development server:
   ```bash
   ..\.venv\Scripts\python manage.py runserver
   ```
3. Visit `http://127.0.0.1:8000` in your web browser. Create a new user profile and copy your unique **API Token** from the dashboard profile card.

### 2. Configure the Windows Desktop Agent
1. Navigate to the `agent/` directory:
   ```bash
   cd agent
   ```
2. Ensure dependency packages are installed:
   ```bash
   ..\.venv\Scripts\pip install -r requirements.txt
   ```
3. Start the agent:
   ```bash
   ..\.venv\Scripts\python agent.py
   ```
4. Enter the server address (`http://127.0.0.1:8000`) and paste your account credentials when prompted.
5. The agent will auto-register all physical USB/local printers connected to the machine and begin polling for print tasks.
