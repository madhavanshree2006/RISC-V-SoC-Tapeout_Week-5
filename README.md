# week5

# 🚀 Week 5 — OpenROAD Flow Setup, Floorplan & Placement

🌟 This week marks the transition from **transistor-level SPICE design (Week 4)** to **backend physical implementation** using **OpenROAD** — a complete open-source RTL-to-GDSII flow.

OpenROAD automates all backend stages of VLSI physical design, including:

➡️ Logic synthesis

➡️ Floorplanning

➡️ Placement

➡️ Clock-Tree Synthesis (CTS)

➡️ Routing

➡️ Final GDSII layout generation

---

## 🎯 Objectives

- Install and configure **OpenROAD Flow Scripts (ORFS)**.
- Perform **Floorplan + Placement** runs successfully.
- Understand how logical netlists map to physical layouts.
- Troubleshoot build and dependency errors effectively.

---

## ⚙️ Step-by-Step Installation

### 🧩 Step 1 — Clone the Repository

```bash
git clone --recursive https://github.com/The-OpenROAD-Project/OpenROAD-flow-scripts
cd OpenROAD-flow-scripts
```
<p align="center"><img src="./ASSETS/1.png" width="700" alt="image 1"/></p>

<p align="center"><img src="./ASSETS/2.png" width="700" alt="image 2"/></p>


### 🗂 Directory Map of the OpenROAD-flow-scripts

```
~/Desktop/OpenROAD_Week5/
├── OpenROAD-flow-scripts          → Main OpenROAD flow repository
│   ├── bazel                      → Build system support
│   ├── build_openroad.sh          → Script to build OpenROAD locally
│   ├── dependencies               → Third-party dependency setup scripts
│   ├── docker                     → Docker build configurations
│   ├── docs                       → Documentation and setup info
│   ├── flow                       → Flow scripts for PnR stages
│   ├── tools                      → Contains tool submodules
│   │   ├── AutoTuner              → Automated tuning tool
│   │   ├── install                → Dependency installation helpers
│   │   ├── OpenROAD               → ⚙️ Main OpenROAD source directory
│   │   │   ├── CMakeLists.txt     → ✅ Edit this file to fix the build issue
│   │   │   ├── src                → OpenROAD source code (C++ modules)
│   │   │   ├── third-party        → Third-party libraries (e.g., OpenSTA, Boost)
│   │   │   ├── include            → Header files
│   │   │   ├── cmake              → CMake helper scripts
│   │   │   ├── docs               → Documentation
│   │   │   ├── test               → (Disabled) Test modules
│   │   │   ├── etc, docker, jenkins → Build/config files
│   │   │   └── WORKSPACE          → Bazel workspace file
│   │   ├── yosys                  → Yosys logic synthesis tool
│   │   ├── yosys-slang            → Yosys slang front-end
│   │   └── yosys_util             → Utility scripts for yosys
│   ├── etc, env.sh, setup.sh      → Environment setup and configuration
│   └── build.log / build_openroad.log → Build output logs
├── spdlog                         → Logging dependency (external)
└── or-tools                       → Google OR-Tools for optimization

```

---

### ⚡ Step 2 — Run the Setup Script

```bash
sudo ./setup.sh

```
<p align="center"><img src="./ASSETS/3.png" width="700" alt="image 3"/></p>

This installs all major dependencies (Yosys, Magic, Netgen, etc.).

🖼️ Example:

<p align="center"><img src="./ASSETS/4.png" width="700" alt="image 4"/></p>


---

### 🛠️ Step 3 — Build OpenROAD

```bash
./build_openroad.sh --local

```
<p align="center"><img src="./ASSETS/5.png" width="700" alt="image 5"/></p>


---

## ⚠️ Error Correction — Build Stuck at ~67 %

During the build process, the compilation may stop around **67 %** due to conflicting CMake test targets or GPU definitions.

To fix this:

### 🪜 Steps to Resolve

1️⃣ Navigate to the OpenROAD source directory inside your flow scripts:

```bash
cd ~/Desktop/OpenROAD_Week5/OpenROAD-flow-scripts/tools/OpenROAD

```

2️⃣ Open and edit `CMakeLists.txt`:

```bash
nano CMakeLists.txt

```

3️⃣ Replace the file contents with this **patched version** (disables tests and ensures GPU flags are properly handled):

```
# SPDX-License-Identifier: BSD-3-Clause
# Copyright (c) 2019-2025, The OpenROAD Authors

cmake_minimum_required (VERSION 3.16)
cmake_policy(SET CMP0078 NEW)
cmake_policy(SET CMP0086 NEW)
cmake_policy(SET CMP0074 NEW)
cmake_policy(SET CMP0071 NEW)
cmake_policy(SET CMP0077 NEW)

option(LINK_TIME_OPTIMIZATION "Flag to control link time optimization: off by default" OFF)
if (LINK_TIME_OPTIMIZATION)
  set(CMAKE_INTERPROCEDURAL_OPTIMIZATION TRUE)
endif()

option(USE_SYSTEM_BOOST "Use system shared Boost libraries" OFF)
option(USE_SYSTEM_OPENSTA "Use system shared OpenSTA library" OFF)
option(USE_SYSTEM_ABC "Use system shared ABC library" OFF)
set(ENABLE_TESTS OFF)

project(OpenROAD VERSION 1 LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 17 CACHE STRING "the C++ standard to use for this project")
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

add_subdirectory(third-party)
add_subdirectory(src)
target_compile_definitions(openroad PRIVATE GPU)

```

4️⃣ Save and exit (Ctrl + O, Enter, Ctrl + X).


5️⃣ Return to your main directory and rebuild:

```bash
cd ~/Desktop/OpenROAD_Week5/OpenROAD-flow-scripts
./build_openroad.sh --local

```


<p align="center"><img src="./ASSETS/6.png" width="700" alt="image 6"/></p>

<p align="center"><img src="./ASSETS/7.png" width="700" alt="image 7"/></p>



✅ This fixes the 67 % halt issue and completes the build successfully.

---

### 🔍 Step 4 — Verify Installation

```bash
source ./env.sh
yosys -help
openroad -help

```
<p align="center"><img src="./ASSETS/8.png" width="700" alt="image 8"/></p>

<p align="center"><img src="./ASSETS/9.png" width="700" alt="image 9"/></p>


---

### 🧪 Step 5 — Run the Flow

```bash
cd flow
make

```

This executes the default test flow and generates floorplan + placement results.
<p align="center"><img src="./ASSETS/10.png" width="700" alt="image 10"/></p>

<p align="center"><img src="./ASSETS/11.png" width="700" alt="image 11"/></p>

<p align="center"><img src="./ASSETS/12.png" width="700" alt="image 12"/></p>


<p align="center"><img src="./ASSETS/13.png" width="700" alt="image 13"/></p>

---

### 🖥️ Step 6 — Launch the GUI

```bash
make gui_final

```

Opens the OpenROAD GUI to visualize layout.
<p align="center"><img src="./ASSETS/14.png" width="700" alt="image 14"/></p>

<p align="center"><img src="./ASSETS/15.png" width="700" alt="image 15"/></p>


---

## 📁 ORFS Directory Structure

```
OpenROAD-flow-scripts/
├── docker        → Docker-based setup
├── docs          → Documentation
├── flow          → Core RTL-to-GDSII flow
├── jenkins       → Regression tests
├── tools         → Tool sources and binaries
├── etc           → Dependency scripts
├── setup_env.sh  → Environment setup script

```

Inside `flow/` directory:

```
├── design     → Example test designs
├── platform   → Technology libraries (LEF, GDS, etc.)
├── makefile   → Flow automation rules
├── scripts    → Stage-specific flow scripts
├── tutorials  → Example learning designs
├── util       → Utility functions

```

---

## 🧾 Outcome Summary

✅ Successfully installed **OpenROAD Flow Scripts**

✅ Resolved build-halt issue (~67 %) by modifying `CMakeLists.txt`

✅ Verified tool execution (Yosys + OpenROAD help)

✅ Ran floorplan + placement stages

✅ Explored GUI for layout visualization

---
