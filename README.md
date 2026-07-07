<h1 align="center">Carbon-Aware Routing via GNN Model</h1>

<p align="center">
  <strong>An intelligent routing simulation that minimizes network CO2 emissions.</strong>
</p>

<p align="center">
  <img alt="Python" src="https://img.shields.io/badge/Python-3.8%2B-blue.svg" />
  <img alt="PyTorch" src="https://img.shields.io/badge/PyTorch-2.0%2B-red.svg" />
  <img alt="License" src="https://img.shields.io/badge/License-MIT-green.svg" />
</p>

## 📖 Overview

This project implements carbon-aware network routing using a combination of an MLP carbon predictor and a Graph Attention Network (GAT). Instead of relying solely on the shortest path, our model predicts the carbon cost of different routes and selects the one that minimizes CO2 emissions.

The model is trained on network routing data and validated using an NS-3 network simulation with NetAnim visualization.

## ✨ Features

- **Carbon Predictor (MLP)**: A lightweight neural network to predict the carbon cost of specific routes based on telemetry.
- **Carbon-Aware GAT**: A Graph Attention Network equipped with temporal encoding to capture time-of-day patterns in carbon intensity.
- **Simulation**: Full integration with the NS-3 network simulator for realistic validation.
- **Standalone Mode**: A built-in local demo to evaluate models without requiring NS-3.

## 🛠️ Prerequisites

- Python 3.8 or higher
- PyTorch 2.0+
- *(Optional)* For full NS-3 simulation: WSL (Ubuntu) with ns-3.41 installed

## 🚀 Installation

1. **Clone the repository:**
   ```bash
   git clone https://github.com/Himanshu10-09/Carbon_Aware_Routing_Using_GNN.git
   cd Carbon_Aware_Routing_Using_GNN
   ```

2. **Set up a virtual environment:**
   ```bash
   python -m venv .venv
   source .venv/bin/activate  # On Windows use: .venv\Scripts\activate
   ```

3. **Install dependencies:**
   ```bash
   pip install -r requirements.txt
   ```

## 💻 Usage

### 1. Train the model
Pre-trained weights are available in `models/best_carbon_predictor.pth`. To retrain the model on the provided dataset:
```bash
python training/train_real_model.py
```
*This reads from `carbon_network_data.csv`, trains the MLP, and saves the weights and scaler to the `models/` directory.*

### 2. Run the standalone demo
To run the model locally without an NS-3 installation:
```bash
python demo_carbon_routing.py
```
*This creates a random network, generates traffic, and compares three routing strategies (Baseline OSPF, Threshold-based, and our Carbon-aware routing). Output plots and metrics are saved to the `results/` directory.*

### 3. Run the full NS-3 simulation
To run the full NS-3 simulation (requires WSL):

**Setup WSL Environment:**
```bash
wsl
cd /path/to/your/project/Carbon_Aware_Routing_Using_GNN
bash setup_ns3_wsl.sh
```

**Set Environment Variables:**
*(Add these to your `~/.bashrc` for convenience)*
```bash
export PATH=~/ns3-venv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
export PYTHONPATH=~/ns-allinone-3.41/ns-3.41/build/bindings/python:$PYTHONPATH
export LD_LIBRARY_PATH=~/ns-allinone-3.41/ns-3.41/build/lib:$LD_LIBRARY_PATH
```

**Run Simulation:**
```bash
source ~/ns3-venv/bin/activate
pip install -r requirements_wsl.txt
bash run_ns3_wsl.sh
```

**Customization examples:**
```bash
bash run_ns3_wsl.sh --hours 6 --nodes 10
bash run_ns3_wsl.sh --hours 48 --nodes 30 --netanim
bash run_ns3_wsl.sh --no-netanim
```

### 4. Run tests
```bash
python tests/test_gnn_model.py
```

## 📂 Project Structure

```text
├── carbon_network_data.csv         # Training dataset
├── enhanced_gnn_model.py           # GAT model with temporal encoding
├── demo_carbon_routing.py          # Standalone demo (no NS-3 needed)
├── run_ns3_demo.py                 # NS-3 simulation entry point
├── run_ns3_wsl.sh                  # WSL launch script
├── setup_ns3_wsl.sh                # NS-3 installation helper
├── requirements.txt                # Python deps
├── requirements_wsl.txt            # WSL deps (includes cppyy)
│
├── models/                         # Models and scalers
├── training/                       # Training scripts and dataloaders
├── simulation/                     # NS-3 simulation components
├── controllers/                    # Routing logic and controllers
├── integration/                    # NS-3 bindings and helpers
├── visualization/                  # Metrics and dashboard plotting
├── config/                         # Configuration files
├── tests/                          # Unit tests
└── results/                        # Generated output metrics & plots (git-ignored)
```

## 🏗️ Architecture

There are two primary models in this project:

- **CarbonPredictor (MLP):** A lightweight model used for standalone routing. It uses 9 input features (hop count, packet/byte counts, flow duration, CPU usage, carbon intensity, and protocol type) to predict the carbon emission for a given route.
- **CarbonAwareGAT:** The graph neural network used in the NS-3 simulation. It uses multi-head graph attention (4 heads, 3 layers) with temporal encoding to capture time-of-day patterns in carbon intensity. It predicts per-edge link weights that the simulation uses to update OSPF routing tables.

## 📊 Dataset

The provided `carbon_network_data.csv` contains 6,000 routing flow samples with features such as `num_hops`, `packet_count`, `byte_count`, `flow_duration`, `cpu_usage`, `carbon_intensity`, `protocol`, and the target label `carbon_emission` (actual carbon emitted in gCO2eq).

## 🐛 Troubleshooting

- **`ModuleNotFoundError: No module named 'torch'`**
  Ensure the virtual environment is activated before running the scripts.
- **Model file not found**
  Run `python training/train_real_model.py` to generate the required weights.
- **NS-3 not loading in WSL**
  Verify that the environment variables are set correctly: `python3 -c "from ns import ns; print('ok')"`.
- **cppyy freezes in WSL**
  This may happen when Windows paths leak into `$PATH`. Fix it by resetting PATH: `export PATH=~/ns3-venv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin`

## 📄 License

This project is licensed under the MIT License.