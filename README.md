# RoboRescueEnv: Autonomous Maze Navigation & Q-Learning Robot

A Python simulation and hardware-ready framework for a rescue robot navigating customizable 2D mazes. The project compares classical pathfinding algorithms (**BFS** and **A\***) against Reinforcement Learning (**Q-Learning**), visualizes how the agent learns from mistakes over time, tests **dynamic re-learning** when sudden obstacles appear, and sends the final movement commands to an **ESP32** microcontroller.

---

## Key Features

- **General 2D Grid Environment (`RoboRescueEnv`):** Supports any maze dimensions, customizable start/goal coordinates, and configurable reward/penalty values.
- **Classical Pathfinding Benchmarks:** Includes **Breadth-First Search (BFS)** and **A\* Search** (with Manhattan Distance heuristic) to measure optimal path length and explored nodes.
- **Q-Learning with Progress Tracking:** Trains an agent from scratch using an $\epsilon$-greedy policy with automatic exploration decay, tracking mistakes, steps, rewards, and success rate per episode.
- **Dynamic Obstacle & Transfer Re-Learning:** Automatically places an unexpected wall along the learned path (while verifying an alternate route exists) and re-trains the agent from its existing Q-table to adapt quickly.
- **Visual Analytics Dashboards:**
  1. **Learning Stages Comparison:** Side-by-side maze snapshots at early, mid, and final training episodes.
  2. **4-Panel Training Dashboard:** Tracks mistakes drop, reward growth, step reduction, and success rate curve.
  3. **Dynamic Adaptation View:** Compares the original path, the collision with the sudden red wall, and the newly adapted route.
  4. **Shock & Recovery Timeline:** Continuous timeline across both training phases showing recovery after the obstacle appears.
- **ESP32 Integration:** Converts the optimal path into directional commands (`U`, `D`, `L`, `R`) and sends them via **WiFi (HTTP POST)** or **USB Serial**.

---

## Environment & Reward System

| Event | Reward / Penalty |
| :--- | :--- |
| Reaching the Goal | `+100` |
| Valid Step | `-1` |
| Hitting a Wall or Boundary (Mistake) | `-20` |

---

## Project Structure

```text
RoboRescueEnv/
├── RoboRescueEnv.ipynb      # Main Google Colab / Jupyter Notebook (or robo_rescue.py)
├── esp32_receiver.ino       # Arduino C++ sketch for ESP32 WiFi HTTP server
├── requirements.txt         # Python dependencies
└── README.md                # Project documentation
```

---

## Installation & Usage

### 1. Install Dependencies
```bash
pip install -r requirements.txt
```
*(Note: `pyserial` is only required if you use USB Serial communication with the ESP32.)*

### 2. Run the Experiment
You can run the notebook directly in **Google Colab** or run the Python script locally by calling `run_maze_experiment(...)`:

```python
run_maze_experiment(
    maze=maze_8x8,
    start=(0, 0),
    goal=(7, 7),
    episodes=1500,
    test_dynamic_obstacle=True,
    new_wall=None,           # Automatically picks a valid mid-path obstacle
    relearn_episodes=600,
    esp32_mode="NONE"        # Options: "NONE", "WIFI", "SERIAL"
)
```

---

## ESP32 Hardware Integration

To send the final path commands (`U` = Up, `D` = Down, `L` = Left, `R` = Right) to your ESP32:

1. Upload `esp32_receiver.ino` to your ESP32 board (via Arduino IDE) after entering your WiFi SSID and password.
2. Copy the IP address printed in the Serial Monitor.
3. Update the experiment call in Python:
   ```python
   run_maze_experiment(
       maze=maze_8x8,
       start=(0, 0),
       goal=(7, 7),
       esp32_mode="WIFI",
       esp32_ip="192.168.1.100"  # Replace with your ESP32 IP
   )
   ```
   Or for USB connection, set `esp32_mode="SERIAL"` and specify your `serial_port` (e.g., `"COM3"` on Windows or `"/dev/ttyUSB0"` on Linux).
