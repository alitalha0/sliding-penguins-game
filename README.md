# Sliding Penguins Puzzle Game

A turn-based puzzle game in Java, played in the terminal. Three penguins slide across a 10x10 icy grid and compete to collect the heaviest load of food. Hazards on the ice push them around, and the water around the edges can knock them out of the game.

Built as a team project for **CENG211 Programming Fundamentals** at İzmir Institute of Technology (Fall 2025). The project's focus is object-oriented design: interfaces, abstract classes, inheritance, polymorphism, enumerations and the Java Collections API.

## How the game works

- At the start, three penguins (**P1**, **P2**, **P3**) are placed on random edge squares. **15 hazards** and **20 food items** are scattered across the grid.
- The player controls one randomly assigned penguin. The other two are controlled by the computer.
- Each penguin gets **4 turns**. Turns go in the order P1 → P2 → P3.
- On each turn, a penguin picks a direction (Up, Down, Left, Right) and **slides** until something stops it:
  - a **food item**: the penguin stops and picks it up,
  - another **penguin**: the moving penguin stops, and the one it hit starts sliding instead,
  - a **hazard**: what happens depends on the hazard type (see below),
  - the **edge of the grid**: the penguin falls into the water and is out of the game.
- When the game ends, penguins are ranked by the total weight of the food they collected. Food collected by a penguin that fell out of the game still counts.

### Penguins

Each penguin type has a special ability it can use **once per game**.

| Type | Special ability |
|---|---|
| **King Penguin** | Can stop on the 5th square of its slide |
| **Emperor Penguin** | Can stop on the 3rd square of its slide |
| **Royal Penguin** | Can step to an adjacent square before sliding. A careless step can still drop it into the water. |
| **Rockhopper Penguin** | Can jump over one hazard in its path. The jump fails if the landing square is not empty. |

### Hazards

| Hazard | Behavior |
|---|---|
| **LightIceBlock** (LB) | Starts sliding when hit. The penguin that hit it is stunned and skips its next turn. |
| **HeavyIceBlock** (HB) | Doesn't move. The penguin that hits it loses its lightest food item. |
| **SeaLion** (SL) | Bounces the penguin back in the opposite direction and starts sliding itself |
| **HoleInIce** (HI) | Anything that slides into it falls in. A LightIceBlock or SeaLion that falls in plugs the hole (PH), and things can then slide over it. |

Hazards that slide can trigger **chain reactions**: they destroy food in their path, push other objects, or plug holes.

### Food

There are five food types, each with a random weight from 1 to 5 units: Krill (Kr), Crustacean (Cr), Anchovy (An), Squid (Sq) and Mackerel (Ma).

### Computer-controlled penguins

The AI chooses moves with simple priority rules:

1. Slide toward a direction that leads straight to food.
2. Otherwise, slide toward a hazard, but never into a HoleInIce.
3. Fall into the water only when there is no other option.

AI penguins have a 30% chance of using their special ability on each turn. Rockhoppers instead use their jump automatically the first time they slide toward a hazard.

## Example

```
*** Turn 1 P1:
P1 does NOT to use its special action.
P1 chooses to move to the LEFT.
P1 takes the Squid on the ground. (Weight 3 units)

New state of the grid:
---------------------------------------------------
|    |    | Cr |    | HI |    | Cr | Ma | P1 |    |
---------------------------------------------------
|    |    |    |    | LB |    | LB |    | Ma | LB |
---------------------------------------------------
| Ma | LB |    |    | HB |    | Sq | Kr |    | P2 |
---------------------------------------------------
...

***** GAME OVER *****
***** SCOREBOARD FOR THE PENGUINS *****
1st place: P3
|---> Food items: Sq (4 units), Kr (3 units), Sq (2 units)
|---> Total weight: 9 units
2nd place: P1
|---> Food items: Sq (3 units), Ma (4 units)
|---> Total weight: 7 units
3rd place: P2 (Your Penguin)
|---> Food items: Ma (3 units)
|---> Total weight: 3 units
```

## Project structure

```
src/penguingame/
├── main/        SlidingPuzzleApp: entry point
├── game/        GameController: game loop, turns, AI decisions, scoreboard
├── terrain/     IcyTerrain (grid, sliding and collision logic), CollisionResult
├── objects/     ITerrainObject, ISlidable, IHazard interfaces; abstract TerrainObject
│   ├── penguins/   Penguin (abstract) and the King, Emperor, Royal, Rockhopper subclasses
│   ├── hazards/    LightIceBlock, HeavyIceBlock, SeaLion, HoleInIce
│   └── food/       Food
├── enums/       Direction, FoodType, PenguinType
└── util/        Position
```

### Design notes

- **Interfaces describe capabilities.** Everything on the grid is an `ITerrainObject`. Objects that can move also implement `ISlidable`, and dangerous ones implement `IHazard`. A `SeaLion` is both a hazard and slidable; a `HoleInIce` is only a hazard.
- **Polymorphism handles special abilities.** `Penguin` is abstract, and each subclass overrides `useSpecialAction()`. The game loop doesn't need to know which penguin type it is working with.
- **The grid is a `List<List<List<ITerrainObject>>>`.** Each cell holds a list, because two objects can briefly share a square during a collision (for example, a penguin landing on food).

## Running

Requires Java 8 or newer. Run these commands from the folder that contains `src/`:

```bash
javac -d bin $(find src -name "*.java")
java -cp bin penguingame.main.SlidingPuzzleApp
```

Input is case-insensitive. Use `Y`/`N` for the special-action prompt and `U`/`D`/`L`/`R` for directions.
