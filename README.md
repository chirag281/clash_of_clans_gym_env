# Clash of Clans Gymnasium Environment

https://github.com/user-attachments/assets/862b62fd-7b7d-4d0c-b528-bf6e511c788d



A custom OpenAI Gym environment and Pygame GUI for simulating Clash of Clans–style attack mechanics, combined with a Proximal Policy Optimization (PPO) agent (via Stable-Baselines3) to learn optimal attacking strategies.

---

## Project Overview

* **Environment**: A Gym-compatible environment (`WarzoneEnv`) that models base layouts, troop deployment, and reward functions for RL.
* **GUI**: A Pygame + pygame\_gui application (`app.py`) with screens for:

  * Base design
  * Troop selection
  * Attack simulation (where the agent plays)
* **Agent**: A PPO-based reinforcement learning agent implemented in `model.py` using Stable-Baselines3.
Clash of Clans RL Environment
PROJECT OVERVIEW
===================================

1. MOTIVATION
------------------------------------------------
Clash of Clans exposes no public API for its combat logic, troop mechanics,  or pathfinding. To conduct reinforcement learning research on strategic  deployment, a functional approximation of the battle system was engineered  from scratch. The objective is to recreate the essential dynamics closely enough for PPO-based training, without requiring the original engine.

2. CORE FORMULATION
------------------------------------------------
The environment operates through three structured state spaces.

BaseSpace contains all building information: positions, hitpoints, defense  values, target preferences, passability masks, and resource content.

TroopSpace maintains all troop-level information: 2D positions, hitpoints, 
movement and attack speeds, timers, target assignments, and domain 
(ground/flying).

DeckSpace represents the undeployed army: troop counts, levels, stat vectors, 
and housing constraints.

Combined, these encode the full world-state at every simulation step.

3. NOVEL TECHNICAL CONTRIBUTIONS
------------------------------------------------
To approximate a closed-source system, several mechanisms were reconstructed 
from first principles.

A grid-based battlefield model was designed and scaled to represent the real 
CoC layout. An octile-distance A* pathfinder serves as a practical stand-in 
for the original navigation mesh. Combat resolution and targeting logic were 
created to emulate troop behavior such as prioritizing walls, defenses, and 
resource structures.

The simulation operates in fixed 50 ms increments. Movement, attack cooldowns, 
building damage, and targeting transitions all run on this deterministic 
frame schedule. Troop updates are vectorized to minimize per-step overhead.

All state spaces are normalized and stacked into consistent tensors to 
maintain stability when training PPO.

4. ACTION SPACE DESIGN
------------------------------------------------
An agent action consists of three components:
(y position, x position, troop category).
The category index identifies which troop to deploy; an additional entry 
serves as a no-op. The system validates placement and rejects actions 
targeting invalid or blocked tiles.

5. REWARD ENGINEERING
------------------------------------------------
The reward function integrates several components: destruction percentage, 
buildings eliminated, gold and elixir obtained, troop survival, penalties 
for illegal placements, and penalties for idle time. This shaping encourages 
meaningful deployments and reduces the sparsity of the original task.

6. ARCHITECTURE PIPELINE
------------------------------------------------
WarzoneEnv receives the agent action, deploys a troop if possible, advances 
the simulation, computes reward, and returns the new observation.

Warzone, the underlying engine, handles movement, targeting, collision with 
walls, combat resolution, destruction progression, and reward accumulation.

A renderer provides visualization for debugging and qualitative assessment 
of policy behavior.

7. TRAINING SETUP (PPO)
------------------------------------------------
The model is trained using PPO with a clipped surrogate objective. Stable 
performance is achieved due to the fixed timestep, consistent state encoding, 
and structured reward. Curriculum-based training is supported by varying base 
layouts and deck compositions.

8. OUTCOME (This section is AI generated)
------------------------------------------------
The project results in a controllable simulation that approximates the core 
combat and strategic behavior of Clash of Clans. It enables end-to-end 
reinforcement learning without relying on the proprietary system. The framework 
is extensible and provides a foundation for broader research in AI for 
real-time strategy and tactical decision-making.

---

## Dependeicies

* [OpenAI Gym](https://gym.openai.com/)
* [Stable-Baselines3](https://github.com/DLR-RM/stable-baselines3)
* [Pygame](https://www.pygame.org/)
* [pygame\_gui](https://pygame-gui.readthedocs.io/)

---

## Installation

1. Clone this repo:

   ```bash
   git clone https://github.com/your-org/ClashGymEnv.git
   cd ClashGymEnv
   ```

2. (Optional) Create and activate a virtual environment:

   ```bash
   python3 -m venv venv
   source venv/bin/activate      # macOS / Linux
   venv\Scripts\activate.bat   # Windows
   ```

3. Install dependencies:

   ```bash
   pip install -r requirements.txt
   ```

---

## Training the Agent

1. Open `model.py` and configure your hyperparameters (learning rate, timesteps, etc.).

2. Run the training script:

   ```bash
   python model.py --train --timesteps 1000000
   ```

3. After training completes, a file `ppo_model.zip` (or similar) will be saved in the project root.

---

## Running the GUI App

With a trained model in place, launch the Pygame application:

```bash
python app.py
```

1. **Main Screen**: Select your Town Hall level.
2. **Base Design**: Lay out defenses and buildings.
3. **Troop Selection**: Choose your deck composition.
4. **Attack Simulation**: Watch the PPO agent deploy troops and execute its learned strategy.

---

## Adding New Algorithms

* Create a new agent file in `agents/` (e.g., `q_learning.py`).
* Implement an `Agent` class with methods:

  * `select_action(state)`
  * `train(env, episodes)`
  * `save(path)` / `load(path)`
* In `ui_attack_screen.py`, import and instantiate your agent alongside the existing PPO agent.

---

## LICENSE
See [LICENSE](LICENSE)
