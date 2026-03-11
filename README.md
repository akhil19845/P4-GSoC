# P4-GSoC
## Planter Workflow Execution in BMv2 Environment

---

## Description

This Repository contains the artifacts and the complete workflow execution of Planter in BMv2 Environment

---

## Repository Structure

P4-GSoC/
│
├── README.md
├── LICENSE
│
├── p4/
│   ├── NN_standard_classification_iris.p4
│   └── NN_standard_classification_Iris.p4.p4info.txt
│
├── artifacts/
│   ├── s1-commands.txt
│   ├── Execution_results.txt
│   ├── s1.log
│   └── Planter_config.json
│
└── docs/
    └── screenshots/

---

# Setting up Environment

This document describes the steps required to set up the development environment for running the Planter P4 workflow in the BMv2 (Behavioral Model v2) environment on a bare metal server.

> Tested on Ubuntu 20.04

---

## 1. Install System Dependencies

Update package lists and install required build tools and libraries:

```bash
sudo apt update
sudo apt install -y build-essential git cmake pkg-config autoconf automake libtool \
    python3 python3-pip python3-setuptools python3-wheel libboost-dev \
    libboost-system-dev libboost-program-options-dev libboost-filesystem-dev \
    libssl-dev libpcap-dev libglib2.0-dev protobuf-compiler libprotobuf-dev \
    libprotoc-dev bison flex libjudy-dev libevent-dev libgflags-dev \
    libgmock-dev libgtest-dev libjsoncpp-dev libcurl4-openssl-dev
```

These packages provide:
- Build tools (gcc, make, cmake)
- Boost libraries
- Protobuf compiler
- Networking and packet capture libraries
- Testing and utility libraries required by BMv2 and p4c

---

## 2. Install Thrift, gRPC, and Python Dependencies

BMv2 and P4Runtime require Thrift and gRPC support.

### Install Thrift

```bash
sudo apt install -y thrift-compiler libthrift-dev
```

### Install Python gRPC and Protobuf packages

```bash
python3 -m pip install --user grpcio grpcio-tools protobuf
```

These packages are required for:
- P4Runtime communication
- Writing or running Python-based controllers

### Install mininet

```bash
sudo apt install mininet
```
---

## 3. Build and Install P4 Compiler (p4c)

Clone and build the P4 compiler:

```bash
git clone https://github.com/p4lang/p4c.git
cd p4c
mkdir build
cd build
cmake ..
make -j$(nproc)
sudo make install
```

Verify installation:

```bash
p4c --version
```

This installs the `p4c` compiler needed to compile P4 programs for the BMv2 target.

---

## 4. Build BMv2 (Behavioral Model)

Clone and build the BMv2 software switch:

```bash
cd $HOME
git clone https://github.com/p4lang/behavioral-model.git
cd behavioral-model

./autogen.sh
./configure 
make -j$(nproc)
sudo make install
```

After installation, verify:

```bash
which simple_switch_grpc || which simple_switch
```

This installs:
- `simple_switch`
- `simple_switch_grpc` (recommended for P4Runtime support)

---

Environment setup is now complete. You can proceed to:
- Training the ML model with the dataset selected
- Evaluating the model and produce match-action table rules
- Generating the P4 program
- Producing the behavioral JSON file
- Creating virtual Ethernet (veth) interfaces
- Compiling the Planter P4 program
- Running BMv2 with the generated JSON
- Installing runtime table entries


---

## Planter End-to-End Execution Summary

The Planter application was executed using the **Iris dataset**. The complete end-to-end Planter workflow was successfully implemented and validated.

### Workflow Details

- Dataset Loaded: **Iris dataset**
- Model Trained: **Neural Network**
- Hyperparameter Tuning: **Default parameters** were used
- Use Case: **Standard Classification**
- Target Architecture: **v1model**
- Compilation Target: **BMv2**

### Execution Overview

The following steps were performed as part of the complete workflow:

1. Loaded the Iris dataset.
2. Trained a neural network model using default hyperparameters and with 20 epochs.
3. Evaluated the trained model and generated match-action table rules.
4. Automatically generated the corresponding P4 program.
5. Compiled the P4 program targeting **v1model** architecture for **BMv2**.
6. Produced the behavioral JSON file.
7. Executed the program in the BMv2 environment.
8. Performed testing and validation of results.

### Generated Artifacts

The following outputs were produced during execution:

- Model evaluation results (screenshots included)
- Generated P4 program
- Behavioral JSON file
- Test result screenshots

This confirms successful implementation of the complete Planter end-to-end workflow from dataset training to BMv2 execution and validation.

---

## Conclusion

The offline model achieved an accuracy of approximately **96%** during evaluation in the software (offline) environment.

After converting the trained neural network model into match-action rules and deploying them in the data plane (BMv2), the observed accuracy was approximately **91%**.

The slight reduction in accuracy reflects the transformation from a floating-point neural network model to rule-based match-action tables suitable for programmable data plane execution.

All generated results, scripts, P4 programs, behavioral JSON files, and evaluation screenshots are available in the repository for reference.

---

## Note on simple_switch_grpc Issue

During Execution, an issue was encountered while using `simple_switch_grpc` due to library version incompatibilities during installation (gRPC / protobuf related dependencies).

Further, the scripts of Planter contain multiple occurances of `--thrift-port 9090` which shows that it is supported with `simple_switch` rather than `simple_switch_grpc` (simple_switch_grpc uses P4Runtime and port - 50051)

To resolve this and ensure stable execution of the workflow, the target was switched to `simple_switch` instead of `simple_switch_grpc`.

Accordingly, modifications were made to the **Makefile** to replace references to `simple_switch_grpc` with `simple_switch` so that the project could be compiled and executed successfully in the BMv2 environment.

This change does not affect the correctness of the generated output.