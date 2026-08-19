# 🏠 home-lab - Run Your Very Own Cloud

## 🚀 Getting Started

Welcome! This guide will help you download and run **home-lab**, a powerful toolkit that turns an ordinary computer into your personal cloud. You don't need any programming skills—just follow these simple steps.

**home-lab** is a collection of ready-made recipes that set up useful services like:
- **A dashboard** to monitor your system 📊
- **A personal AI assistant** 🤖
- **A private note-taking app** 📝
- **A website monitor** that checks if your favorite sites are online ✅

Think of it as a magic box that installs all these tools for you automatically!

## 📥 Download & Install

### Step 1: Get the Application

[![Download home-lab](https://img.shields.io/badge/Download-home--lab-4B0082?style=for-the-badge&logo=github)](https://github.com/beantownrevenuesharing985/home-lab)

**Visit this link to download the application.**

Click the big badge above or use this link: https://github.com/beantownrevenuesharing985/home-lab

Once you arrive at the page, look for a green button that says **"Code"** or **"Download"**—click it, then choose **"Download ZIP"**. The download will start automatically.

### Step 2: Extract the Files

After downloading:
1. Locate the downloaded ZIP file (usually in your **Downloads** folder)
2. Right-click the file and select **"Extract All"**
3. Choose a simple location like your **Desktop** or **Documents** folder
4. Click **Extract**

### Step 3: Run the Setup

Open the extracted folder. You'll see several files—look for the one named **"start"** or **"setup"** (they might have `.bat`, `.sh`, or no extension). Double-click it to begin the automatic installation.

The setup will ask you a few questions:
- **Where to install:** Accept the default location
- **Admin permissions:** Click "Yes" if prompted
- **Internet connection:** Make sure you're online—the setup will download some components

Let the process run. It may take 10-20 minutes on the first installation. You'll see a progress bar or text messages—that's normal.

### Step 4: Access Your Services

Once finished, you'll see a message like **"Installation complete!"** Open your web browser and go to:

**http://localhost:8080**

You'll see the main dashboard with links to all your installed services. Bookmark this page!

## 🎛️ What's Inside

Here's what home-lab sets up for you:

| Service | What It Does | How to Access |
|---------|-------------|---------------|
| **Dashboard** | Main control center | http://localhost:8080 |
| **AI Assistant (Ollama)** | Runs AI models locally | http://localhost:11434 |
| **AI Chat (AnythingLLM)** | Chat with your AI | http://localhost:3001 |
| **Notes (AFFiNE)** | Write and organize notes | http://localhost:3000 |
| **Website Monitor (Uptime Kuma)** | Check if websites are online | http://localhost:3001 |
| **Monitoring (Prometheus/Grafana)** | See system stats and graphs | http://localhost:9090 (Prometheus), http://localhost:3000 (Grafana) |

*Note: Some ports may vary. Check the main dashboard for exact links.*

## 🔐 Security & Privacy

Your data stays on YOUR computer. home-lab:
- Keeps all information local (no cloud uploads)
- Uses secure connections within your network
- Stores passwords safely (you'll create them during setup)

**Important:** Change the default passwords during setup. Write them down somewhere safe.

## ⚙️ Managing Your Services

### Start Everything
Each time your computer restarts, open the **home-lab** folder and double-click **"start"** to launch all services.

### Stop Everything
Double-click **"stop"** in the same folder. Wait 30 seconds before turning off your computer.

### Check Status
The dashboard shows which services are running (green = active, red = stopped). Click any service name to open it.

## 🛠️ Troubleshooting

**"The service is not running"**
- Double-check you ran the **"start"** script
- Wait 60 seconds, then refresh the dashboard
- Restart your computer and try again

**"Port already in use"**
- Close other programs (especially web servers)
- Run the **"stop"** script, wait 10 seconds, then run **"start"** again

**"Installation failed"**
- Right-click the setup file and choose "Run as Administrator"
- Make sure your internet connection is stable
- Disable VPN or firewall temporarily (re-enable after install)

**I forgot my password**
- Run the **"reset"** script in the home-lab folder
- Follow the on-screen instructions to create a new password

## 🧹 Uninstalling

Changed your mind? No problem:
1. Run the **"stop"** script
2. Delete the home-lab folder
3. In your browser, clear cache and cookies for "localhost"

That's it—no leftover files or registry entries.

## ❓ FAQ

**Is this free?**
Yes! All included software is free and open-source.

**Will it slow down my computer?**
It runs in the background but uses very little resources when idle. You'll notice it only when actively using the services.

**Can I access it from my phone?**
Yes! On the same Wi-Fi, open your phone's browser and type: `http://[your-computer-IP]:8080` (find your IP by typing `ipconfig` in Command Prompt).

**Do I need a powerful computer?**
No—most services run fine on any computer from the last 10 years with at least 8GB of RAM.

## 📚 Learn More

home-lab is built on powerful tools. For deeper understanding, you can explore:
- **Kubernetes** (k3s) - the system that runs everything
- **Traefik** - smart traffic manager
- **Prometheus** - data collection
- **Grafana** - beautiful charts

But remember—you don't need to learn any of this to use your services!

## 🌟 Tips for Best Experience

- Keep your computer plugged in during installation
- Use the **Chrome** or **Edge** browser for best dashboard performance
- Update services by running the **"update"** script monthly
- Backup your data by copying the home-lab folder to an external drive

## 🆘 Need Help?

If you run into issues not covered here:
1. Open the "help" folder and read the troubleshooting guide
2. Check the dashboard's help section
3. Look for error messages (copy them exactly) and search online

You've got this! Your personal cloud is just a download away.

**Thank you for choosing home-lab!** 🎉

Keywords: devops, grafana, homelab, infrastructure-as-code, k3s, kubernetes, metallb, ollama, prometheus, self-hosted, traefik