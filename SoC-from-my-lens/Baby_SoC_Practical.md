
-----


# The Definitive VSDBabySoC Guide: From RTL to GDSII Prep

Welcome, future SoC designer\! This guide is your definitive, all-in-one resource for simulating and synthesizing the **VSDBabySoC**, a fantastic open-source System-on-Chip.

I've combined insights from multiple guides and learned from real-world errors to create a single, foolproof walkthrough. Whether you're a complete beginner or just want a clean workflow, this is your starting point. We'll go from a fresh Linux setup to a fully synthesized, verified gate-level netlist ready for physical design.

### The "Why": What is VSDBabySoC?

At its core, VSDBabySoC is an **integration project**. It takes three powerful, independent, open-source IP (Intellectual Property) cores and combines them onto a single chip:

1.  **RVMYTH Core:** A simple RISC-V based CPU that acts as the "brain."
2.  **AVSDPLL:** A Phase-Locked Loop (PLL) that serves as the "heartbeat," generating a stable clock signal.
3.  **AVSDDAC:** A Digital-to-Analog Converter (DAC) that acts as the "mouth," allowing the digital chip to communicate with the analog world.

By working through this project, you're not just learning Verilog; you're learning the fundamental process of modern chip design: integrating and verifying diverse components to create a powerful system.

-----

## Phase 1: The One-Time Workshop Setup (The Golden Path)

This is the most critical phase. We will set up your Ubuntu environment perfectly to avoid all the common errors. Do these steps once, and you'll be set for success.

### Step 1: Update Your System

Always start with a fresh and up-to-date package list.

```bash
sudo apt update
sudo apt upgrade -y
```

### Step 2: Install All Essential Tools

We'll install everything you need for simulation, synthesis, and visualization in one go. This command is corrected from older guides to work on modern Ubuntu versions.

```bash
sudo apt install make git iverilog gtkwave python3-venv 
```
>`Note : if already installed then ignore this step, make file is usually pre-available in the github repository that you will copy ie; VsdBabySoC.`

### Step 3: Install Docker (The Modern, Official Way)

Docker is used to run the synthesis flow in a clean, predictable environment. We will use the official installation method, which is more reliable than using the default `apt` version.

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

### Step 4: Fix Docker Permissions Before It's a Problem\!

This is the single most common point of failure. To avoid `permission denied` errors, you **must** add your user to the `docker` group.

```bash
sudo usermod -aG docker ${USER}
```

> **🔥 MEGA IMPORTANT\! 🔥**
> For this change to take effect, you **MUST** fully log out and log back in, or simply **RESTART** your computer. Your current terminal session does not have these new permissions. Skipping this step **guarantees** you will encounter errors later.

-----

## Phase 2: The Core Workflow (The Cakewalk)

Now that your environment is perfect, running the project is incredibly simple using the provided `Makefile`.

### Step 5: Clone the Project and Set Up the Python Sandbox

We will clone the official repository and create a safe Python virtual environment to avoid system conflicts.

```bash
# Navigate to your home directory
cd ~

# Clone the project
git clone https://github.com/manili/VSDBabySoC.git
cd VSDBabySoC

# Create and activate a Python virtual environment
python3 -m venv venv
source venv/bin/activate

# Install required Python packages inside the sandbox
pip3 install pyyaml click sandpiper-saas
```

> **Note:** Your terminal prompt should now start with `(venv)`. Keep this active whenever you work on this project.

### Step 6: Run Pre-Synthesis (RTL) Simulation

This command simulates your Verilog code *before* it's turned into gates, letting you check the core logic.

```bash
make pre_synth_sim
```

### Step 7: Visualize the Results

Use GTKWave to view the generated waveform.

```bash
gtkwave output/pre_synth_sim/pre_synth_sim.vcd
```
>
><img width="3189" height="644" alt="Image" src="https://github.com/user-attachments/assets/78ce8417-db55-4765-9b0f-cc986f659d93" />

> **💡 Pro Tip: Seeing the Sine Wave**
> Your `OUT` signal might look like a blocky digital signal. To see the beautiful analog sine wave it represents:
>
> 1.  In the "Waves" panel, **right-click** the `OUT` signal.
> 2.  Go to **Data Format \> Analog \> Analog Step**.
> 3.  The view will instantly transform\!

### Step 8: Run Synthesis and Post-Synthesis (Gate-Level) Simulation

This is the magic button. This single command will:

1.  Launch Docker.
2.  Run the **Yosys** tool to synthesize your RTL into a gate-level netlist based on the Sky130 PDK.
3.  Run a new simulation on that gate-level netlist to prove it's still functionally correct.

<!-- end list -->

```bash
make post_synth_sim
```

After it completes, view the gate-level simulation results. They should look identical to the pre-synthesis waveforms.

```bash
gtkwave output/post_synth_sim/post_synth_sim.vcd
```
>
><img width="3199" height="915" alt="Image" src="https://github.com/user-attachments/assets/9568c010-4ce3-4001-a351-7282f0567f30" />

**Congratulations\!** You have now run a complete RTL-to-verified-netlist flow using the automated, modern method.

-----

## Phase 3: Diving Deeper (The Manual "Under-the-Hood" Flow)

For those who want to understand the nitty-gritty, this section details the manual commands that the `Makefile` runs for you. This is fantastic for learning because every command has a distinct and important purpose.

### Part A: Manual Synthesis with Yosys

This part is all about converting your abstract Verilog code (RTL) into a concrete list of physical logic gates (a netlist).

#### **Step 3.1: Launch Yosys**

  * **Use:** Starts the Yosys Synthesis Suite, your interactive digital workshop.
  * **Importance:** You are entering a stateful environment where you will load your design, libraries, and run a sequence of commands to build your chip's logic.

<!-- end list -->

```bash
yosys
```

#### **Step 3.2: Load Verilog RTL and Stub Files**

  * **Use:** Loads your Verilog source files into the tool's memory.
  * **Importance:** You are loading the blueprints for your design. A critical concept here is loading **stub files** for the analog parts (PLL and DAC). Synthesis tools only understand digital logic, so you provide them with an empty "black box" that just defines the inputs and outputs. This tells the tool, "Trust me, something with these connections will exist here, but don't try to build it yourself."

<!-- end list -->

```yosys
read_verilog src/module/vsdbabysoc.v
read_verilog -I src/include src/module/rvmyth.v
read_verilog -I src/include src/module/clk_gate.v
read_verilog src/module/avsddac_stub.v
read_verilog src/module/avsdpll_stub.v
```
* Special Note : For stub files if you encounte any error then try using these lines `-I src/include` , before src/module/avsddac_stub.v and src/module/avspll_stub.v. This will solve the error and if not then the troubleshooting is on your side as the reasons for error might vary from person to person.
  
#### **Step 3.3: Load Technology Libraries**

  * **Use:** Loads the technology library files (`.lib`).
  * **Importance:** This is where the physical reality of the chip comes in. The `.lib` files are a catalog of available logic gates (AND, OR, Flip-Flops) for a specific factory (the Sky130 technology). They tell Yosys exactly what physical components are available, how fast they are, and how much power they use. Without this, synthesis is impossible.

<!-- end list -->

```yosys
read_liberty -lib src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
read_liberty -lib src/lib/avsddac.lib
read_liberty -lib src/lib/avsdpll.lib
```
><img width="3199" height="1999" alt="Image" src="https://github.com/user-attachments/assets/f09f488e-4431-4892-83b1-419e3b0d8d87" />
><img width="1205" height="1999" alt="Image" src="https://github.com/user-attachments/assets/dac9c7e3-8df3-42b5-9312-1ba8bef6d120" />
#### **Step 3.4: Run the Synthesis & Mapping Flow**

  * **Use:** This is the core sequence that transforms your design from abstract code to physical gates.
  * **Importance:** This is the "magic" of synthesis.
      * `synth -top vsdbabysoc`: This is the main command. It takes your high-level Verilog (e.g., `a = b + c;`) and translates it into a generic structure of logic gates.
      * `dfflibmap` & `abc`: These commands perform **technology mapping**. They take the generic gates and replace them with the *specific, real* gates from the Sky130 `.lib` file you loaded. They also run powerful optimization algorithms to make the design smaller and faster.
      * `flatten` & `clean`: These are housekeeping steps. They simplify the final design structure, making it easier for the next stage of tools (physical design) to understand.

<!-- end list -->

```yosys
synth -top vsdbabysoc
dfflibmap -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
opt
abc -liberty src/lib/sky130_fd_sc_hd__tt_025C_1v80.lib
flatten
clean -purge
```

#### **Step 3.5: Visualize the Gate-Level Netlist**

  * **Use:** Generates and displays a graphical schematic of the design in its current state.
  * **Importance:** This is your visual confirmation. It allows you to see the "sea-of-gates" that you've just created. It's an invaluable debugging step to ensure the structure looks reasonable before you move on.

<!-- end list -->

```yosys
show vsdbabysoc
```

#### **Step 3.6: Write the Final Netlist and Exit**

  * **Use:** Saves the final gate-level netlist to a new Verilog file and exits the tool.
  * **Importance:** This command produces the main output of the entire synthesis process. The `vsdbabysoc_netlist.v` file is the final blueprint, which will be used for post-synthesis simulation and is the direct input for the physical design (Place and Route) tools.

<!-- end list -->

```yosys
write_verilog -noattr reports/vsdbabysoc_netlist.v
exit
```

### Part B: Manual Post-Synthesis (Gate-Level) Simulation

The goal here is to prove that the complex netlist of gates you just created behaves identically to your original, simple Verilog code.

#### **Step 3.7: Prepare Sky130 Verilog Models**

  * **Use:** Ensure the behavioral Verilog models for the Sky130 gates are available.
  * **Importance:** The simulator (`iverilog`) doesn't understand the physical data in the `.lib` files. It needs a separate set of Verilog files that describe the *logical behavior* of each gate. For example, it needs a file that tells it an `AND2_X1` gate should perform a logical AND operation. The `make` flow handles copying these, but manually you must ensure they are present.

#### **Step 3.8: Compile with Icarus Verilog**

  * **Use:** Compiles all the necessary files for a gate-level simulation.
  * **Importance:** This is a complex step where you manually link everything together:
    1.  The `testbench.v` (the stimulus, or the "driver" of the test).
    2.  The `vsdbabysoc_netlist.v` (the synthesized design you are testing).
    3.  The Sky130 behavioral models (so the simulator knows what an `AND` gate is).
    4.  The behavioral models for the analog black boxes (`avsdpll.v`, `avsddac.v`) so the simulation can run end-to-end.

<!-- end list -->

```bash
iverilog -DFUNCTIONAL -DUNIT_DELAY=\#1 -o simulation/post_synth_sim.out \
src/gls_model/primitives.v \
src/gls_model/sky130_fd_sc_hd.v \
reports/vsdbabysoc_netlist.v \
src/module/avsdpll.v \
src/module/avsddac.v \
src/module/testbench.v
```

#### **Step 3.9: Run and View the Final Waveform**

  * **Use:** Runs the compiled simulation and then views the output waveform.
  * **Importance:** This is the final verification. You run the simulation to generate a waveform (`.vcd` file) and then you open it in GTKWave. The ultimate goal is to place this waveform side-by-side with the pre-synthesis waveform. **If they match perfectly, you have successfully proven that your synthesis was correct.**

<!-- end list -->

```bash
vvp simulation/post_synth_sim.out
gtkwave simulation/dump.vcd
```
------------
### The Conclusion 
* The crux of this whole task and process was to get a perspective of how an SoC looks like , feels like , and works like.
* There will be lots of error while doing this experiment and AI tools will come to your rescue, make sure to have a strong mindset as the sheer number of errors can make you overwhelmed.
* Errors will vary from system to system and configuration to configration, that is what handled by these SoC's underlying your system.
* In the end you will be able to see your BABY, obviously BabySoC.
* While learning about `functional modelling` we can clearly observe that Pre synthesis and Post synthesis file were giving the same waveform, signalling that we successfully conserved our modules functionality across the synthesis flow.
* This validates that the synthesized netlist is functionally equivalent to the RTL design. 
-----------------------

