# The Definitive VSDBabySoC Guide: From RTL to GDSII Prep

Welcome, future SoC designer! This guide is your definitive, all-in-one resource for simulating and synthesizing the **VSDBabySoC**, a fantastic open-source System-on-Chip.

We've combined insights from multiple guides and learned from real-world errors to create a single, foolproof walkthrough. Whether you're a complete beginner or just want a clean workflow, this is your starting point. We'll go from a fresh Linux setup to a fully synthesized, verified gate-level netlist ready for physical design.

### The "Why": What is VSDBabySoC?

At its core, VSDBabySoC is an **integration project**. It takes three powerful, independent, open-source IP (Intellectual Property) cores and combines them onto a single chip:

1.  **RVMYTH Core:** A simple RISC-V based CPU that acts as the "brain."
2.  **AVSDPLL:** A Phase-Locked Loop (PLL) that serves as the "heartbeat," generating a stable clock signal.
3.  **AVSDDAC:** A Digital-to-Analog Converter (DAC) that acts as the "mouth," allowing the digital chip to communicate with the analog world.

By working through this project, you're not just learning Verilog; you're learning the fundamental process of modern chip design: integrating and verifying diverse components to create a powerful system.

## Phase 1: The One-Time Workshop Setup (The Golden Path)

This is the most critical phase. We will set up your Ubuntu environment perfectly to avoid all the common errors we discovered. Do these steps once, and you'll be set for success.

### Step 1: Update Your System

Always start with a fresh and up-to-date package list.

`sudo apt update`
`sudo apt upgrade -y`

### Step 2: Install All Essential Tools

We'll install everything you need for simulation, synthesis, and visualization in one go. This command is corrected from older guides to work on modern Ubuntu versions.

`sudo apt install make git iverilog gtkwave python3-venv xdot graphviz`

### Step 3: Install Docker (The Modern, Official Way)

Docker is used to run the synthesis flow in a clean, predictable environment. We will use the official installation method, which is more reliable than using the default `apt` version.

**1. Install Docker's Prerequisites:**

`sudo apt install ca-certificates curl`

**2. Add Docker’s Official GPG Key (for security):**

`sudo install -m 0755 -d /etc/apt/keyrings`
`sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc`
`sudo chmod a+r /etc/apt/keyrings/docker.asc`

**3. Set Up the Docker Repository:**

`echo \ "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \ $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \ sudo tee /etc/apt/sources.list.d/docker.list > /dev/null`

**4. Install Docker Engine:**

`sudo apt update`
`sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin`

### Step 4: Fix Docker Permissions Before It's a Problem!

This is the single most common point of failure. To avoid `permission denied` errors, you **must** add your user to the `docker` group.

`sudo usermod -aG docker ${USER}`

> **🔥 MEGA IMPORTANT! 🔥**
> For this change to take effect, you **MUST** fully log out and log back in, or simply **RESTART** your computer. Your current terminal session does not have these new permissions. Skipping this step **guarantees** you will encounter errors later.

## Phase 2: The Core Workflow (The Cakewalk)

Now that your environment is perfect, running the project is incredibly simple using the provided `Makefile`.

### Step 5: Clone the Project and Set Up the Python Sandbox

We will clone the official repository and create a safe Python virtual environment to avoid system conflicts.

`# Navigate to your home directory`
`cd ~`
`# Clone the project`
`git clone https://github.com/manili/VSDBabySoC.git`
`cd VSDBabySoC`
`# Create and activate a Python virtual environment`
`python3 -m venv venv`
`source venv/bin/activate`
`# Install required Python packages inside the sandbox`
`pip3 install pyyaml click sandpiper-saas`

> **Note:** Your terminal prompt should now start with `(venv)`. Keep this active whenever you work on this project.

### Step 6: Run Pre-Synthesis (RTL) Simulation

This command simulates your Verilog code *before* it's turned into gates, letting you check the core logic.

`make pre_synth_sim`

### Step 7: Visualize the Results

Use GTKWave to view the generated waveform.

`gtkwave output/pre_synth_sim/pre_synth_sim.vcd`

> **💡 Pro Tip: Seeing the Sine Wave**
> Your `OUT` signal might look like a blocky digital signal. To see the beautiful analog sine wave it represents:
>
> 1.  In the "Waves" panel, **right-click** the `OUT` signal.
>
> 2.  Go to **Data Format > Analog > Analog Step**.
>
> 3.  The view will instantly transform!

### Step 8: Run Synthesis and Post-Synthesis (Gate-Level) Simulation

This is the magic button. This single command will:

1.  Launch Docker.
2.  Run the **Yosys** tool to synthesize your RTL into a gate-level netlist based on the Sky130 PDK.
3.  Run a new simulation on that gate-level netlist to prove it's still functionally correct.

`make post_synth_sim`

After it completes, view the gate-level simulation results. They should look identical to the pre-synthesis waveforms.

`gtkwave output/post_synth_sim/post_synth_sim.vcd`

**Congratulations!** You have now run a complete RTL-to-verified-netlist flow using the automated, modern method.

## Phase 3: Diving Deeper (The Manual "Under-the-Hood" Flow)

For those who want to understand the nitty-gritty, this section details the manual commands that the `Makefile` runs for you. This is fantastic for learning but less efficient for day-to-day work.

### Part A: Manual Synthesis with Yosys

1.  **Start a fresh Yosys session** from your `VSDBabySoC` directory:
    `yosys`
2.  **Load Verilog RTL and Stub files:**
    `read_verilog src/module/vsdbabysoc.v`
    `read_verilog -I src/include src/module/rvmyth.v`
    `read_verilog -I src/include src/module/clk_gate.v`
    `# Use STUB files for analog black boxes`
    `read_verilog src/module/avsddac_stub.v`
    `read_verilog src/module/avsdpll_stub.v`
3.  **Load Technology Libraries:**
    `read_liberty -lib src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
    `read_liberty -lib src/lib/avsddac.lib`
    `read_liberty -lib src/lib/avsdpll.lib`
4.  **Run the Synthesis & Mapping Flow:**
    `synth -top vsdbabysoc`
    `dfflibmap -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
    `opt`
    `abc -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib`
    `flatten`
    `clean -purge`
5.  **Visualize the Gate-Level Netlist:**
    `show vsdbabysoc`
6.  **Write the Final Netlist and Exit:**
    `write_verilog -noattr reports/vsdbabysoc_netlist.v`
    `exit`

### Part B: Manual Post-Synthesis (Gate-Level) Simulation

1.  **Get the Sky130 Verilog Models:** For a gate-level simulation, you need the behavioral models of the gates themselves. The `make` flow handles this, but manually you'd need to ensure `primitives.v` and `sky130_fd_sc_hd.v` are available.
2.  **Compile with Icarus Verilog:** This command compiles your testbench against the synthesized netlist and all the required behavioral models.
    `iverilog -DFUNCTIONAL -DUNIT_DELAY=\#1 -o simulation/post_synth_sim.out \`
    `src/gls_model/primitives.v \`
    `src/gls_model/sky130_fd_sc_hd.v \`
    `reports/vsdbabysoc_netlist.v \`
    `src/module/avsdpll.v \`
    `src/module/avsddac.v \`
    `src/module/testbench.v`
3.  **Run and View:**
    `vvp simulation/post_synth_sim.out`
    `gtkwave simulation/dump.vcd`
