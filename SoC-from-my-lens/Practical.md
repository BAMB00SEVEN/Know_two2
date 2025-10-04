Certainly, here is the complete README file.

# The Ultimate VSDBabySoC Simulation Guide: From Zero to Waveform

Welcome, aspiring SoC designer\! This guide is your ultimate companion for setting up and running the VSDBabySoC project. If you're new to the world of Verilog, Docker, and Linux, don't worry. We've designed this walkthrough to be a cakewalk, learning from common mistakes to give you a smooth, error-free journey.

We'll go from a fresh Ubuntu setup all the way to visualizing the beautiful sine wave output of our simulated chip.

### So, What Are We Doing?

We are about to simulate a "System on a Chip" (SoC) called **VSDBabySoC**. Our goal is to run a digital simulation of this chip's design, which includes a RISC-V processor core, and then synthesize it into a netlist of digital logic gates. Finally, we'll visualize the output to confirm it's working as expected.

-----

##  Phase 1: Setting Up Your Digital Workshop (Prerequisites)

This is the most crucial phase. A tiny mistake here can cause headaches later. We'll get your Ubuntu system ready with all the necessary tools.

### Step 1: Update Your System

First things first, let's make sure your system's package list is up-to-date.

```bash
sudo apt update
sudo apt upgrade -y
```

### Step 2: Install Essential Design & Simulation Tools

Next, we'll install the core software for Verilog simulation (`iverilog`), waveform viewing (`gtkwave`), and other command-line essentials.

```bash
sudo apt install make git iverilog gtkwave
```

> ** What are these?**
>
>   - `make`: A build automation tool that reads a `Makefile` to execute complex command sequences for us.
>   - `git`: The version control system we'll use to download the project code.
>   - `iverilog`: Our Verilog compiler and simulator.
>   - `gtkwave`: The graphical tool we'll use to see the final waveforms.

### Step 3: Installing Docker (The Right Way)

Our project uses Docker to ensure the simulation environment is identical for everyone, avoiding "it works on my machine" issues. Installing it correctly is key.

**1. Install Docker's Prerequisites:**

```bash
sudo apt install ca-certificates curl
```

**2. Add Docker’s Official GPG Key (for security):**

```bash
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

**3. Set Up the Docker Repository:**

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

**4. Install Docker Engine:**

```bash
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Step 4: Solving the "Permission Denied" Error Before It Happens\!

This is a classic stumbling block. To run Docker without typing `sudo` every time, you must add your user to the `docker` group.

```bash
sudo usermod -aG docker ${USER}
```

> **🔥 MEGA IMPORTANT\! 🔥**
> For this change to take effect, you **MUST** fully log out and log back in, or simply **RESTART** your computer. Your current terminal session does not have these new permissions. Skipping this step **will** cause a "permission denied" error later.

-----

## Phase 2: Cloning and Running the Simulation

With our workshop set up, it's time to get the project files and run the simulations.

### Step 5: Clone the Project Repository

Let's download the VSDBabySoC project code from GitHub. We'll do this in our Home directory.

```bash
cd ~
git clone https://github.com/manili/VSDbabySoC.git
cd VSDbabySoC
```

### Step 6: Setting Up a Python Virtual Environment

Modern Ubuntu protects its system Python installation. To install the project's Python dependencies safely, we must create a virtual environment—a private sandbox for our project.

**1. Install the `venv` tool:**

```bash
sudo apt install python3-venv
```

**2. Create the virtual environment (we'll call it `venv`):**

```bash
python3 -m venv venv
```

**3. Activate the environment:**

```bash
source venv/bin/activate
```

> You'll know it worked because your terminal prompt will now start with `(venv)`. You must always have this active when working on this project.

**4. Install Python packages inside the sandbox:**

```bash
pip3 install pyaml click sandpiper-saas
```

### Step 7: Run the Pre-Synthesis Simulation

This first simulation checks the basic functionality of our design *before* it's converted into logic gates.

```bash
make pre_synth_sin
```

This will run for a bit and generate a waveform file.

### Step 8: Visualizing the First Waveform\!

Let's open the output file with GTKWave and see our result.

```bash
gtkwave output/pre_synth_sin.vcd
```

A GTKWave window will open.

1.  In the top-left "SST" panel, click on the `...` to expand the design hierarchy. Click on `vsd_tb`.
2.  In the "Signals" panel below, click on the signals you want to view (e.g., `CLK`, `OUT`). They will appear in the "Waves" panel.

> **💡 Pro Tip: Analog vs. Digital View**
> Your `OUT` signal might look like a blocky green bar, while the tutorial shows a smooth sine wave. This is just a display setting\!
>
>   - **Right-click** on the `OUT` signal name in the "Waves" panel.
>   - Go to **Data Format \> Analog \> Analog Step**.
>   - Voila\! Your blocky digital value is now visualized as the beautiful analog wave it represents.

-----

## Phase 3: Synthesis and Post-Synthesis Simulation

Now for the magic. We'll use a tool called **Yosys** (run via Docker) to synthesize our human-readable Verilog code into a machine-level netlist of logic gates.

### Step 9: Run the Post-Synthesis Simulation

This command uses Docker to run the synthesis process and then simulates the resulting netlist.

```bash
make post_synth_sim
```

> **Troubleshooting Corner:**
>
>   - If you get a `permission denied` error related to `docker.sock`, it means you forgot to **restart or log out** after the `usermod` command in Step 4.
>   - If you get a `File ... not found or is a directory` error inside Yosys, it might mean a file was accidentally deleted. Try running `git restore .` in your project folder to restore any missing files and then run the `make` command again.

