# Pac-Man Simulation with MARS

This project implements a grid-based simulation of the classic **Pac-Man** game using the **MARS framework**. Agents such as Pac-Man and the ghosts operate within a grid environment and interact based on simple AI rules. The model is fully visualizable through a separate Python visualization component, allowing real-time monitoring of agent behavior.

## Project Structure

Here's an overview of the structure of the project:

- `Program.cs`: The main entry point that starts the simulation.
- `config.json`: Configures simulation parameters like runtime, visualization, and agent setup. See [Configuration](#configuration).
- `Model/`: Contains all the core logic including agent behavior, game rules, environment setup, and utility classes.
- `Resources/`: Contains initialization files for the grid and agents.
- `Visualization/main.py`: A Python-based visualization tool that connects via WebSocket to render the simulation in real-time.

## Model Description

This simulation includes the following main agent and layer types:

### Agents

- **PacManAgent**: The main agent controlled by logic defined in its `Tick()` method. It can collect pellets, power pellets, and eat ghosts if powered up. The behavior is customizable by adjusting the logic inside `Tick()`.
- **GhostAgent**: Enemies that alternate between chasing and scattering, with specific AI behaviors based on their type (Blinky, Pinky, Inky, Clyde). Ghosts can be eaten only if Pac-Man is powered up.
Each **GhostAgent** has a behavior mode that determines its current strategy. You can check the mode of a ghost using its `Mode` property. The possible modes are:

  - `Chase`: The ghost actively pursues Pac-Man.
  - `Scatter`: The ghost moves to a predefined area on the map and wanders there.
  - `Frightened`: The ghost is vulnerable and can be eaten by Pac-Man (after he eats a power pellet).
  - `Eaten`: The ghost has been eaten and is returning to its home tile to regenerate.
- **Pellet & PowerPellet**: Consumable items that increase the score. Power pellets grant Pac-Man the temporary ability to eat ghosts.

### Layers

- **MazeLayer**: The main layer where all agents interact. It manages the environment grid, agent spawning, and the game loop, including score, collisions, and lives.
- **Environments**: Spatial hash environments that manage agent placement and exploration for Pac-Man and ghosts.

## Pac-Man Behavior Customization

Pac-Man's behavior is defined in the `Tick()` method of `PacManAgent.cs`. This method can be customized to test different strategies. You can control Pac-Man’s decisions using the following helper methods:

| Method | Description |
|--------|-------------|
| `List<GhostAgent> ExploreGhosts()` | Returns a list of ghost agents near Pac-Man (within visual range). |
| `List<Position> ExploreGhostPositions()` | Returns the positions of nearby ghosts. |
| `List<Position> ExplorePelletPositions()` | Returns the positions of nearby pellets. |
| `List<Position> ExplorePowerPelletPositions()` | Returns the positions of nearby power pellets. |
| `List<Position> ExploreOccupiablePositions()` | Returns all valid and reachable positions Pac-Man can move to. |
| `double GetDistance(Position target)` | Returns the Euclidean distance between Pac-Man and the target position. |
| `int GetScore()` | Returns the current score of the game. |

To move Pac-Man, use:

- `MoveTowardsGoal(Position target)` – Moves Pac-Man **one field** toward the target. **Diagonal movement is supported**.

An example logic for prioritizing power pellets, pellets, or running from ghosts is already implemented in the code.

## SmartGhostAgent – Customizable Ghost Agent

In addition to the classic **GhostAgent**, this project includes the **SmartGhostAgent**, a programmable variant designed for implementing custom AI behavior. Much like the `PacManAgent`, the `SmartGhostAgent` can be fully controlled via its `Tick()` method, allowing for dynamic and personalized logic. To ensure correct integration with the state and timing mechanisms, the first line of the `Tick()` method should remain `if (ProcessGhostState()) return;`. 

To use `SmartGhostAgents` in the simulation, you must update the `config.json` file to control which agent type is active.  
Only **one** of the two, `GhostAgent` or `SmartGhostAgent`, should have a `count` of `4`, while the other must be set to `0`.


### Behavior and Characteristics
- **Release Timer**: Every ghost, including the `SmartGhostAgent`, has a `ReleaseTick` attribute that defines at which simulation tick the agent is allowed to begin moving, either at game start or after Pac-Man loses a life.

- **Scatter Mode**: While in Scatter mode, the `SmartGhostAgent` has full autonomy over its movement. Unlike classic ghosts, it is not bound to predefined corner targets.

- **Frightened Mode**: When in the Frightened state, the ghost may move freely, but only on **every second tick**, mimicking the slowed behavior from the original game.

- **Eaten Mode**: Once eaten by Pac-Man, the ghost automatically returns to its home tile just like any regular `GhostAgent`.

- **Chase / Scatter Switching**: The `SmartGhostAgent` can switch between Chase and Scatter modes on its own, enabling more coordinated behavior with other ghosts.

- **Limited Vision**: For balancing purposes, the `SmartGhostAgent` has a **smaller visual range** than Pac-Man and can only detect Pac-Man **within its current field of view**. Global tracking of Pac-Man is **not** possible.

### Methods for Custom Logic

The following methods are available to the `SmartGhostAgent` for implementing custom strategies:

| Method | Description |
|--------|-------------|
| `PacManAgent? ExplorePacMan()` | Returns the visible `PacManAgent` instance, or `null` if Pac-Man is not in view. |
| `List<GhostAgent> ExploreTeam()` | Returns a list of all teammates. |
| `bool EnterChaseMode()` | Switches the ghost into Chase mode, if currently in Scatter mode. |
| `bool EnterScatterMode()` | Switches the ghost into Scatter mode, if currently in Chase mode. |
| `bool Frightened()` | Returns whether the ghost is currently in the Frightened state. |
| `bool MoveTowardsGoal(Position target)` | Moves the ghost one step toward a target. |
| `List<Position> ExploreOccupiablePositions()` | Returns all reachable positions within the ghost’s visual range. |
| `double GetDistance(Position target)` | Returns the Euclidean distance to a given position. |
| `Position GetRandomCell()` | Returns a random walkable cell from the map. |
| `List<Position> ExplorePelletPositions()` | Returns positions of all visible pellets. |
| `List<Position> ExplorePowerPelletPositions()` | Returns positions of all visible power pellets. |

For game balance reasons, the `SmartGhostAgent` has a smaller visual range than Pac-Man, since the ghosts act in groups of four.

## Game Logic

- Pac-Man gains:
  - **10 points** for a pellet,
  - **50 points** for a power pellet,
  - **200 points** for eating a ghost (only when powered up).
- Pac-Man has **3 lives**. A life is lost when a ghost catches him while not powered up.
- If Pac-Man loses a life:
  - He and all ghosts are reset to their **starting positions**.
- If all lives are lost, the simulation ends.
- The visibility of Pac-Man is defined by a **visual range** property.

To interact agents must be on the same position after their Tick() action.
This means that Pac-Man and a ghost may pass each other without interacting if they cross paths but do not end up on the same cell. Interactions only occur if they share the same grid cell at the end of the tick.

## Configuration

The simulation is controlled via the `config.json` file:

- `step`: Time steps of the simulation.
- `Visualization`: Set to `true` to enable the Python visualization client.
- `VisualizationTimeout`: Delay between ticks when visualization is active. A **lower number speeds up** the simulation visually.
- `agents` / `layers`: Defines which agents and layers are active and how many agents are spawned.

For more details on configuration, refer to the [MARS documentation](https://mars.haw-hamburg.de/articles/core/model-configuration/index.html).

## Simulation & Visualization Setup

### Prerequisites

- [.NET Core 8.0 or later](https://dotnet.microsoft.com/en-us/download)
- [Python 3.11](https://www.python.org/)
- A C# IDE (e.g. [JetBrains Rider](https://www.jetbrains.com/rider/) recommended)

### Run Instructions

1. Clone or download the repository.
2. Open the solution file (`.sln`) in your IDE.
3. Make sure `Visualization` is set to `true` in `config.json`.
4. Start the simulation (`Program.cs`). It will log:  
   `Waiting for live visualization to run.`
5. In a terminal, navigate to the `Visualization/` folder and run:

   ```bash
   pip install -r requirements.txt
   python main.py
Note for macOS users: You may need to use pip3 and python3 instead:
   ```bash
   pip3 install -r requirements.txt
   python3 main.py
