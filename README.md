# PyMOL on WSL Ubuntu: Installation & Troubleshooting Guide

Running Graphical User Interface (GUI) applications via Windows Subsystem for Linux (WSL) can introduce complex environment cross-contamination and 3D graphics virtualization bugs. This repository serves as an open-source technical playbook documenting how to successfully deploy, troubleshoot, and optimize PyMOL within an isolated Linux environment on Windows.

Whether you are hitting terminal dependency deadlocks, frozen windows, or invisible taskbar icons (`[WARN: COPY MODE]`), this guide breaks down why these virtualization layers fail and details the exact hardware-rendering overrides needed to establish a stable workflow.

---

## 📋 Pre-requisites & System Context
* **Host OS:** Windows 11 (or Windows 10 with active WSLg support)
* **Linux Distribution:** Ubuntu (WSL2 Architecture)
* **Environment Manager:** Miniconda / Anaconda Python Distribution

---

## ❌ The Roadblocks: What Didn't Work (And Why)

### Attempt 1: The Global System Install (`apt`)
* **The Action:** I updated my system index and installed PyMOL globally using `sudo apt install pymol`.
* **The Failure:** When I typed `pymol`, the terminal threw immediate module errors.
* **The "Why":** My terminal was running inside a Miniconda environment (`base`). The global launcher tried searching my Conda Python path for the PyMOL files instead of the system path, causing a messy cross-contamination.
* **Lesson Learned:** Clean up your environment before moving forward (`sudo apt purge pymol && sudo apt autoremove`).

### Attempt 2: Shoving It Into the Conda `base` Environment
* **The Action:** I tried installing PyMOL directly into my root Conda `base` profile using `conda install`.
* **The Failure:** Conda entered an infinite dependency resolution loop, spitting out massive red text errors about library conflicts (specifically involving `libgfortran`).
* **The "Why":** The root `base` environment already had too many pre-existing packages. Forcing a heavy GUI tool into it caused a massive domino effect of version mismatches.

### Attempt 3: The Ghost App & `[WARN: COPY MODE]` Bug
* **The Action:** I built a separate environment, but when I ran PyMOL, the software icon appeared on my Windows taskbar, but clicking it did nothing. The interface was entirely invisible. Hovering over it showed a warning: `[WARN: COPY MODE]`.
* **The Failure:** The software was running perfectly in Linux, but it could not display itself on my desktop.
* **The "Why":** PyMOL relies on heavy 3D hardware graphics acceleration (OpenGL). The virtualized bridge between WSL and Windows (WSLg) was completely choking on my GPU driver, failing to draw the window layout.

---

## ✅ The Game Plan: What Actually Worked

### 1️⃣ Isolate Your Work in a Clean Sandbox
Never dump complex graphical tools into your root `base` environment. Create a dedicated container. Lock your environment to Python 3.10 for maximum stability, as many of PyMOL's external packages haven't been updated for newer Python builds yet.
```bash
conda clean --all -y
conda create -n pymol_env -c conda-forge pymol-open-source python=3.10 -y
conda activate pymol_env
```

### 2️⃣ Bypass the GPU (The Golden Fix)
Since the virtualized hardware acceleration bridge is prone to freezing, force the underlying Linux graphics framework to skip your GPU entirely. Tell it to compute the 3D viewport using your CPU via software rendering:
```bash
LIBGL_ALWAYS_SOFTWARE=1 pymol
```

### 3️⃣ Automate the Fix Permanently
You do not want to type that long graphics command every single time. Close PyMOL (`Ctrl + C`), step back to your base profile (`conda deactivate`), and append a global alias directly to your `.bashrc` file. This tells Ubuntu to apply the graphics override silently in the background:
```bash
echo "alias pymol='LIBGL_ALWAYS_SOFTWARE=1 pymol'" >> ~/.bashrc && source ~/.bashrc
```

### 4️⃣ Create a One-Click Windows Desktop Shortcut
You can launch straight into your structural biology work from Windows without opening your terminal manually:
1. Right-click your Windows Desktop ➡️ **New** ➡️ **Shortcut**.
2. Paste this exact target path:
   ```text
   wsl.exe -d Ubuntu -e bash -lic "conda activate pymol_env && pymol"
   ```
3. Name it **PyMOL (WSL)** and save it.

---

## 💡 Key Technical Takeaways
1. **Isolate:** Keep your development environments compartmentalized using separate tool environments.
2. **Fallback:** If graphics freeze or stay invisible over a virtualized bridge, try dropping back to software rendering (`LIBGL_ALWAYS_SOFTWARE=1`).
3. **Automate:** Once a command works, turn it into an alias or a desktop shortcut immediately to protect workflow efficiency.
