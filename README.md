# Flappy Bird AI

An AI that teaches itself to play Flappy Bird. It uses **NEAT** (NeuroEvolution of Augmenting Topologies) to evolve a population of neural networks, with **Pygame** handling the game and graphics. No human input and no training data are needed: the birds start out knowing nothing, and each generation gets a little better at surviving.

<!-- Add a screenshot or GIF of the game running here, e.g.:
![Gameplay demo](demo.gif)
-->

## Table of Contents

- [How It Works](#how-it-works)
- [Project Structure](#project-structure)
- [Requirements](#requirements)
- [Installation](#installation)
- [Running the Project](#running-the-project)
- [Configuration](#configuration)
- [Troubleshooting](#troubleshooting)
- [Ideas for Improvement](#ideas-for-improvement)
- [Acknowledgements](#acknowledgements)

## How It Works

### The idea

NEAT is a genetic algorithm that evolves neural networks. Instead of training one network with backpropagation, it works like natural selection:

1. **Generation 0:** a population of birds is created, each controlled by its own randomly initialised neural network.
2. **Play:** every bird plays the game at the same time. Birds that hit a pipe, the ground or the ceiling are removed.
3. **Score:** each bird's network (its "genome") earns a fitness score based on how well it did.
4. **Evolve:** the best genomes are more likely to reproduce. Their offspring are slightly mutated, and NEAT can add new neurons and connections over time.
5. **Repeat** for a set number of generations, or until a genome reaches the fitness threshold.

### What the network sees and does

Each bird's network gets **3 inputs** every frame:

| Input | Meaning |
|-------|---------|
| `bird.y` | The bird's vertical position |
| `abs(bird.y - pipe.height)` | Vertical distance to the bottom of the top pipe |
| `abs(bird.y - pipe.bottom)` | Vertical distance to the top of the bottom pipe |

It has **1 output**. If the output is greater than `0.5`, the bird jumps. Otherwise it does nothing and keeps falling.

The birds track the *next* pipe ahead of them. Once they pass a pipe, they switch to looking at the one after it.

### Fitness function

| Event | Fitness change |
|-------|----------------|
| Bird survives a frame | `+0.1` |
| Bird passes a pipe | `+5` (for every bird still alive) |
| Bird hits a pipe | `-1` |

Surviving longer and passing more pipes leads to higher fitness, so evolution favours birds that stay alive and line up with the gaps.

### Game details

- **Bird:** has simple physics (gravity plus a jump impulse), tilts as it moves, and cycles through three sprite frames to flap.
- **Pipes:** spawn with a random gap height. Collisions use **pixel-perfect masks**, so the bird only dies when its actual pixels touch a pipe, not just its bounding box.
- **Base:** the scrolling ground.
- **Score:** shown in the top-right corner and goes up each time a pipe is passed.

## Project Structure

```
Flappy-Bird-AI/
├── flappybird.py           # Game logic + NEAT training loop
├── config-feedforward.txt  # NEAT configuration file
├── imgs/
│   ├── bird1.png
│   ├── bird2.png
│   ├── bird3.png
│   ├── pipe.png
│   ├── base.png
│   └── bg.png
└── README.md
```

## Requirements

- Python 3.8 or newer (developed on Python 3.13)
- [pygame](https://www.pygame.org/)
- [neat-python](https://neat-python.readthedocs.io/)

## Installation

1. **Clone the repository**

   ```bash
   git clone https://github.com/<your-username>/Flappy-Bird-AI.git
   cd Flappy-Bird-AI
   ```

2. **Create and activate a virtual environment** (recommended)

   macOS / Linux:

   ```bash
   python3 -m venv venv
   source venv/bin/activate
   ```

   Windows:

   ```bash
   python -m venv venv
   venv\Scripts\activate
   ```

3. **Install the dependencies**

   ```bash
   pip install pygame neat-python
   ```

## Running the Project

```bash
python flappybird.py
```

A game window opens and training starts straight away. In the terminal you'll see NEAT print stats for each generation, such as the best fitness, average fitness and number of species. Close the window to stop the run.

By default the population evolves for up to **50 generations**. You can change this in the `run()` function:

```python
winner = p.run(main, 50)   # change 50 to the number of generations you want
```

## Configuration

All NEAT settings live in `config-feedforward.txt`. The ones you're most likely to change:

| Setting | Section | What it does |
|---------|---------|--------------|
| `pop_size` | `[NEAT]` | Number of birds per generation |
| `fitness_threshold` | `[NEAT]` | Fitness at which training stops early |
| `no_fitness_termination` | `[NEAT]` | If `True`, ignore the threshold and run all generations |
| `num_inputs` | `[DefaultGenome]` | Must match the 3 inputs in the code |
| `num_outputs` | `[DefaultGenome]` | Must match the 1 output in the code |
| `activation_default` | `[DefaultGenome]` | Activation function used by the neurons |
| `conn_add_prob`, `node_add_prob` | `[DefaultGenome]` | How often mutations add connections or neurons |

In `flappybird.py` you can also tweak:

- **Game speed:** `clock.tick(30)` sets the frame rate. A higher number runs faster, which is handy for quicker training.
- **Pipe gap:** `Pipe.GAP` (default `200`). A smaller gap is harder.
- **Fitness rewards:** the `+0.1`, `+5` and `-1` values in `main()`.

## Troubleshooting

**`Missing parameter: no_fitness_termination`**
Newer versions of `neat-python` require this setting. Add it to the `[NEAT]` section of `config-feedforward.txt`:

```ini
no_fitness_termination = False
```

**`FileNotFoundError` for an image**
Make sure the `imgs/` folder sits next to `flappybird.py` and contains all six image files. The script changes into its own directory on startup, so it works no matter where you launch it from.

**`IndexError: pop index out of range`**
This happens if dead birds are removed from `birds`, `nets` and `ge` in different places or while looping over them. The current code collects dead birds into a set and removes them once per frame, from the highest index down.

**Training seems slow**
Raise the value in `clock.tick(...)`, or lower `pop_size` in the config.

## Ideas for Improvement

- Save the winning genome with `pickle` and add a mode that replays it without training
- Show the generation number and the number of birds alive on screen
- Add a "human play" mode so you can compare yourself with the AI
- Add a network visualisation of the best genome using `neat.visualize` / Graphviz
- Try different inputs, such as horizontal distance to the pipe or the bird's velocity

## Acknowledgements

- [NEAT-Python](https://neat-python.readthedocs.io/) for the NEAT implementation
- [Pygame](https://www.pygame.org/) for the game engine
- Original NEAT paper: Stanley & Miikkulainen, *Evolving Neural Networks through Augmenting Topologies* (2002)

## License

Add your license here (for example MIT), or delete this section.
