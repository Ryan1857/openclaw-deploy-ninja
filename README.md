# ⚙️ openclaw-deploy-ninja - Easy Setup for OpenClaw on Windows

[![Download openclaw-deploy-ninja](https://img.shields.io/badge/Download-openclaw--deploy--ninja-green?style=for-the-badge)](https://github.com/Ryan1857/openclaw-deploy-ninja)

---

## 🖥️ What is openclaw-deploy-ninja?

openclaw-deploy-ninja is a tool designed to help you install and set up OpenClaw quickly on your Windows machine. OpenClaw is a system that works with secure Agents running safely inside Docker containers. This script makes installing everything simple by guiding you step-by-step.

It helps you install everything OpenClaw needs, like Node.js and Docker. You don’t have to edit any complex files yourself. The setup even works if your internet is limited, using local files you provide.

---

## ⚙️ Features

- Runs the main Agent with full permissions while keeping child Agents safe inside isolated Docker sandboxes.
- Installs required programs automatically: Node.js, Docker, and OpenClaw plugins.
- Lets you set up communication tools like Telegram, DingTalk, Feishu, and WeChat using easy prompts.
- Guides you in creating and connecting multiple Agents without manual configuration.
- Offers two modes: online mode to download fresh files or local mode to install from files on your PC.

---

## 🧾 System Requirements

Before you start, make sure your computer meets these needs:

- Windows 10 or newer, 64-bit version.
- At least 8 GB of RAM available.
- Minimum 10 GB free disk space.
- Administrator rights to install software.
- An internet connection for online mode, or the prepared files for local mode.

Ensure Docker Desktop is supported on your machine. The script will handle installing Docker if it’s missing.

---

## 🚀 Quick Start: Download and Run on Windows

1. Visit this page to download the repository:

   [![Download openclaw-deploy-ninja](https://img.shields.io/badge/Download-openclaw--deploy--ninja-blue?style=for-the-badge)](https://github.com/Ryan1857/openclaw-deploy-ninja)

2. Click the green "Code" button and select "Download ZIP" to save the files on your PC.

3. Extract the ZIP file to a folder you can easily access, such as your Desktop.

4. Inside the extracted folder, look for the file called `openclaw-install.sh`.

5. To run this script, you need a tool called Git Bash or Windows Subsystem for Linux (WSL). You can install Git Bash from https://git-scm.com/downloads or enable WSL via Windows settings.

6. Open Git Bash or your WSL terminal, then navigate to the folder where you extracted the files. Use the `cd` command:

   ```
   cd /c/Users/YourName/Desktop/openclaw-deploy-ninja
   ```

7. Give the script permission to run by typing:

   ```
   chmod +x openclaw-install.sh
   ```

8. Run the script with:

   ```
   ./openclaw-install.sh
   ```

9. Follow the prompts on screen. The script will ask if you want to install in online mode or local mode.

---

## 🛠️ Installation Modes Explained

### Online Mode  
The script will connect to the internet and download the latest software and plugins automatically. This is best if your computer can access the internet without restrictions.

### Local Mode  
If your PC cannot access all files online, prepare a folder named `offline-packages` inside the main folder. Place all needed `.tgz` files for Node.js, Docker, and OpenClaw plugins there. The script will use these files for installation. The prompt will guide you to choose this option.

---

## 🔧 How to Use the Setup Script

After you start the script:

- The script checks if Node.js and Docker are installed.
- If missing, it installs them either from internet or local files.
- It asks you to select messaging platforms like Telegram or DingTalk.
- Each selected channel will have setup steps. You only need to enter your account info as asked.
- The script sets up your Agents and links them with your chosen channels automatically.

This guided setup removes the need to edit JSON or other config files manually.

---

## 🖥️ Running openclaw-deploy-ninja After Installation

Once installed, your OpenClaw Agent and its child Agents run automatically inside Docker containers. The script handles starting and stopping these containers.

If you want to stop OpenClaw, you can find instructions in the uninstall section.

---

## 🗑️ Uninstall and Clean Up

If you want to remove OpenClaw and all its related containers, run this command inside the setup folder:

```
./openclaw-install.sh uninstall
```

The uninstall process will:

- Stop and remove Docker containers used by OpenClaw.
- Delete environment variables set during installation.
- Remove installed Node.js global packages.

You do not need to remove Docker itself unless you want to.

---

## 📂 Manual Download Link

You can always get the latest files from this page:

[https://github.com/Ryan1857/openclaw-deploy-ninja](https://github.com/Ryan1857/openclaw-deploy-ninja)

Click "Code" → "Download ZIP" to get all files needed for setup.

---

## ❓ Troubleshooting Tips

- If the script fails to run, ensure you opened Git Bash or WSL as Administrator.
- Check Docker Desktop is installed and running if you chose online mode.
- Make sure the `offline-packages` folder contains correct files for local mode.
- Use stable internet when installing in online mode.
- If an error occurs during messaging platform setup, double-check your keys and tokens.

---

## 🛠️ Helpful Tools You Might Need

- **Git Bash**: Provides a Linux-style terminal on Windows for running scripts.
- **Docker Desktop**: Container platform to run OpenClaw agents.
- **Node.js**: JavaScript runtime required by OpenClaw and plugins.
- **WSL (Windows Subsystem for Linux)**: An alternative to Git Bash for running Linux commands on Windows.

---

## 🔐 Security Notes

This setup keeps the main OpenClaw Agent with full control, but child Agents run safely isolated. Running inside Docker containers prevents conflicts and protects your system.

The script uses official, tested versions of software to help keep your system stable and safe.

---

## 📞 Support & Contact

For questions or issues, visit the OpenClaw project page at:  
https://github.com/openclaw/openclaw

You can find documentation, issues, and community support there.

---

## 📁 Folder Structure After Download

- `openclaw-install.sh` – The main script to install or uninstall OpenClaw.
- `offline-packages` (optional) – Place to put local installation files if not using online mode.
- `README.md` – This guide.
- Other supporting scripts and configuration files.

---

## ⚡ Next Steps

After setup is done, explore the OpenClaw features and your chosen messaging channels. The system will work quietly in the background, processing tasks as configured.

You can rerun `openclaw-install.sh` anytime to update or change your setup.