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
<img width="1074" height="526" alt="1" src="https://github.com/user-attachments/assets/7388f708-829d-4667-a639-f2a8f40c9bdc" />

<img width="1074" height="526" alt="2" src="https://github.com/user-attachments/assets/aa9fce0f-36ee-42d0-8511-cbdc8959f0f2" />


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
<img width="1032" height="509" alt="3" src="https://github.com/user-attachments/assets/d2ad825a-4a71-4de1-9f17-a963ed92be2c" />

This installs all major dependencies (Yosys, Magic, Netgen, etc.).

🖼️ Example:

<img width="1074" height="526" alt="4" src="https://github.com/user-attachments/assets/dd04a200-e75b-4b8c-b445-7f3486ab1140" />


---

### 🛠️ Step 3 — Build OpenROAD

```bash
./build_openroad.sh --local

```
<img width="1074" height="526" alt="5" src="https://github.com/user-attachments/assets/2173a65e-6865-4bc5-ba98-554b3d542562" />


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


<img width="1064" height="471" alt="6" src="https://github.com/user-attachments/assets/55a3fde0-9a6c-46a6-a75c-1f35f8550f03" />

<img width="1051" height="548" alt="7" src="https://github.com/user-attachments/assets/34506596-ff09-4fcc-9356-b18d26a0636b" />



✅ This fixes the 67 % halt issue and completes the build successfully.

---

### 🔍 Step 4 — Verify Installation

```bash
source ./env.sh
yosys -help
openroad -help

```
<img width="1108" height="620" alt="8" src="https://github.com/user-attachments/assets/e3368158-02b7-4a40-b1af-39ba75fb4d0a" />

<img width="1143" height="620" alt="9" src="https://github.com/user-attachments/assets/877502c8-7540-4a38-9282-0c236eec0bb3" />


---

### 🧪 Step 5 — Run the Flow

```bash
cd flow
make

```

This executes the default test flow and generates floorplan + placement results.
<img width="1143" height="620" alt="10" src="https://github.com/user-attachments/assets/69d1873d-3d0e-4c93-807e-2668788bec64" />

<img width="1143" height="620" alt="11" src="https://github.com/user-attachments/assets/27f7cb33-5891-47e7-bd68-b1797f15ec15" />

<img width="1143" height="620" alt="12" src="https://github.com/user-attachments/assets/706001dd-1560-47a2-bf09-47d880046f5d" />


<img width="1143" height="620" alt="13" src="https://github.com/user-attachments/assets/bbd00437-0ffb-4257-8b7e-0a7894251921" />

---

### 🖥️ Step 6 — Launch the GUI

```bash
make gui_final

```

Opens the OpenROAD GUI to visualize layout.
<img width="1920" height="1200" alt="14" src="https://github.com/user-attachments/assets/1ea71449-6e0b-48cf-b54e-a7c6810c26ab" />

<img width="1920" height="1200" alt="15" src="https://github.com/user-attachments/assets/04f19949-de9e-4491-807f-0bc435e6ffc8" />


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
