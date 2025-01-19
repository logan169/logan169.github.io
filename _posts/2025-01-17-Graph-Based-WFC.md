---
title: 'Graph Based Wave Function Collapse (WFC)' 
tags: Algorithm Procedural Houdini
---
In this article, we will delve into implementing a Graph-Based Wave Function Collapse (GBWFC) algorithm, explaining its core principles, highlighting its advantages over traditional methods, and showcasing its application through practical examples.

The GBWFC is a variation of the Wave Function (WFC) Collapse algorithm developed by Maxim Gumin, WFC has been used in various applications, including image synthesis, level design, and texture generation. Inspired by quantum mechanics, the algorithm operates by collapsing a "wave function" of possible configurations into a single, consistent result, much like how the quantum state of particles is observed in physics. The core idea of WFC is to generate patterns, textures, or levels that adhere to local constraints (like adjacency rules) while maintaining global consistency across the entire structure as shown below from a gif picked from the [WFC original repository](https://github.com/mxgmn/WaveFunctionCollapse/tree/master). It is a fascinating mix of randomness and determinism, allowing for the creation of complex structures that appear both organic and highly structured.

<div class="grid">
  <div class="cell cell--2"></div>
  <div class="cell cell--auto">
    <img src="https://github.com/mxgmn/WaveFunctionCollapse/blob/master/images/wfc.gif?raw=true" alt="city generator">
  </div>
  <div class="cell cell--2"></div>
</div>

# How It Works

The original WFC implementation process starts with a grid or a defined space where each cell is in a superposition of all possible states (or patterns). The algorithm progresses by collapsing these possibilities, one cell at a time, based on predefined constraints that dictate how patterns can adjacently align. This process continues until the entire grid is resolved into a coherent output that adheres to the specified rules. Since its introduction, alternative implementations of the WFC algorithm have emerged, enabling its use in generating procedural 3D voxel-based worlds and even in crafting poetry.

# Limitations
While the Wave Function Collapse (WFC) algorithm is powerful, it has notable limitations. A primary challenge is ensuring global coherence in the generated output. The algorithm prioritizes local constraints, which can occasionally result in inconsistencies at a larger scale. For example, when generating expansive structures such as cities, the local arrangements might be logical and consistent, but the overall layout could lack cohesion. Additionally, if the constraints are too loose, some visible tile redundancy may occur, reducing the uniqueness of the output.

![city generator](https://camo.githubusercontent.com/8016d071f236aa7d1f00572740069b57360b5100f55598a517b766f67526b2a8/68747470733a2f2f692e696d6775722e636f6d2f764c3830697a762e6a7067) [source](https://github.com/marian42/wavefunctioncollapse)

Another significant limitation of WFC is its computational cost. The algorithm involves frequent recalculation of possible states for each cell in the grid, particularly after every collapse, to ensure the constraints are maintained. As the size of the grid and the number of potential states increase, the computational demand grows exponentially. This makes WFC especially resource-intensive for larger grids or complex constraint sets, potentially slowing down generation times or requiring optimizations to remain practical for real-time applications. 

# Graph Based WFC

Graph-based Wave Function Collapse (GBWFC), first introduced in scientific literature in 2019, has been gaining attention for its ability to address the limitations of traditional grid-based WFC. 

By leveraging graph structures, GBWFC improves constraint management, scalability, and computational efficiency. It minimizes redundancy and enhances global coherence, making it more effective for larger, more complex systems. 

In this article, we'll demonstrate how to implement GBWFC, and apply it to an example which feature non-neighborhood constraints that the original WFC method struggles to handle.

# WFC In Houdini

In Houdini, an already existing [SideFX Labs WFC implementation](https://www.sidefx.com/tutorials/wfc-dungeon-generator/) relies on Houdini node-based approach to procedurally generate environments like dungeons. By using Wang tiles, it ensures that adjacency constraints are satisfied, providing a flexible and powerful solution to create complex game level design.

If this tool aligns with your needs, it may be all you require, and I highly recommend giving it a try, as it is a well-designed solution. However, if computational costs become a concern, you may want to consider an alternative implementation that will also supports backtracking (which isn't the case of this tool from my last read of its source code).

Additionally, if your constraints involve non square tiles (e.g., Voronoi or hexagonal tiles) or tiles that aren’t adjacent on the grid, the SideFX Labs implementation will not be suitable for your purposes, and you might need to develop your own WFC solution.

# Motivations

Since I work in the cinema industry with large, complex 3D environments, we are often pushing the boundaries of what the current Wang tiles method can offer with the SideFX Labs approach. However, I still dream of using this technology to create vast, immersive environments.

For example, a Nested Recursive Multi-Level GBWFC approach could be particularly effective for generating complex structures like cities. The process begins by running a WFC algorithm to assign neighborhood types to each grid cell. Once neighborhoods are defined, each cell is treated as a subgrid, and a new WFC process is initiated to assign house or block types to the cells within each subgrid. This recursive process could continue at multiple levels—populating building interiors, adding furniture, and placing objects on furniture—while preserving the constraints at both the higher neighborhood level and the local room or object level. This multi-level approach ensures the generated city remains coherent, with local details aligning with the broader global structure, offering a scalable and flexible framework for procedural content generation.

# GBWFC Implementation

If you've made it this far, it's time to dive into implementing our GBWFC. The objective was to design the core components to be as generic as possible, so they could be easily extended with additional attributes specific to the needs of different generators (e.g., generating Sudoku/voronoi grids). This approach ensures flexibility and adaptability for a range of applications while maintaining the core functionality of the algorithm.

## Imports

Import base dependencies
```python

import copy
import math
import random

```

## Generic WFC Components

At this stage, we can begin implementing the core components of the WFC algorithm:

- State: Holds all possible configurations that can be assigned to tiles.
- Tile: Represents the individual grid tiles, store edges and states information.
- Edge: Enables us to abstract the input shape, allowing support for any shape, not just grid-based inputs. It also facilitates the creation of more complex pattern relationships.

### State

```python
class State(object):
    """
    State class represents a container for storing the state data and associated information.
    It can hold an ID, a state value, and additional optional attributes.
    """

    def __init__(self, id, state, **kwargs):
        """
        Initializes a State object with an ID, a state value, and optional additional attributes.

        Parameters:
        id (int): The unique identifier for this state object.
        state (str): The state value represented as a string.
        **kwargs: Additional keyword arguments to dynamically set extra attributes on the object.
        """
        self.id = id  # Assign the unique identifier to the object
        self.state = str(state)  # Ensure the state is stored as a string

        # Dynamically set additional attributes from kwargs
        for key, value in kwargs.items():
            setattr(self, key, value)

    def __repr__(self):
        """
        Returns a string representation of the State object, showing its state.

        Returns:
        str: The state value as a string.
        """
        return self.state

```

### Tile

```python
class Tile:
    """
    The Tile class represents a tile in the grid. It holds the states a tile can have, 
    the edges connecting it to other tiles, and allows for operations related to its state and connections.
    """

    def __init__(self, id, states, edges, **kwargs):
        """
        Initializes a Tile object with an ID, possible states, edges, and optional additional attributes.

        Parameters:
        id (int): A unique identifier for the tile.
        states (set): A set of all possible State objects that can be assigned to the tile.
        edges (list): A list of Edge objects that represent the connections to other tiles.
        **kwargs: Optional keyword arguments representing additional attributes to attach to the tile object.
        """
        self.id = id  # Unique identifier for the tile.
        self.repr_str = f"Tile{self.id}"  # String representation of the tile.
        self.states = states  # Set of possible states for the tile.
        self.edges = edges  # List of edges connecting this tile to others.

        # Dynamically set additional attributes from kwargs
        for key, value in kwargs.items():
            setattr(self, key, value)

    @property
    def has_converged(self):
        """
        Property to check if the tile has converged, i.e., if it has only one possible state left.

        Returns:
        bool: True if the tile has converged (i.e., only one possible state remains), 
              False otherwise.
        """
        return len(self.states) < 2

    def get_connected_tile(self, edge_name):
        """
        Finds and returns the tile connected through a specific edge by its name.

        Parameters:
        edge_name (str): The name of the edge to look for.

        Returns:
        Tile: The tile connected by the specified edge.

        Raises:
        ValueError: If no edge with the given name is found in the tile's edges.
        """
        # Clean the edge name to match the expected format
        clean_edge_name = "__".join([self.repr_str, edge_name])

        # Iterate through the edges to find the corresponding connection
        for edge in self.edges:
            if clean_edge_name in edge.names:
                return edge.get_connected_tile(self)

        # Raise an error if the edge name is not found
        raise ValueError(f"Unable to find an existing edge with the name: {edge_name}")

    def __repr__(self):
        """
        Provides a string representation of the Tile object, typically used for debugging.

        Returns:
        str: A string representing the tile (e.g., 'Tile1', 'Tile2', etc.).
        """
        return self.repr_str
```
### Edge

```python
class Edge:
    """
    A data container object representing an edge between two tiles in the WFC model.
    
    Each edge contains an ID, a list of names representing the connection between tiles,
    and references to the two connected tiles. Additional attributes can be passed as
    keyword arguments and dynamically added to the Edge object.
    """

    def __init__(self, id, names, tiles, **kwargs):
        """
        Initializes an Edge object.

        Args:
            id (int): A unique identifier for the edge.
            names (list): A list of strings representing the edge's name based on the connected tiles.
            tiles (list): A list containing exactly two Tile objects that the edge connects.
            **kwargs: Additional optional attributes to be added dynamically to the Edge object.
        """
        self.id = id
        self.names = names
        self.tiles = tiles

        # Dynamically add any additional attributes
        for key, value in kwargs.items():
            setattr(self, key, value)

    def get_connected_tile(self, input_tile):
        """
        Retrieves the tile connected to the given input tile.

        Args:
            input_tile (Tile): The tile for which the connected tile is to be found.

        Returns:
            Tile: The tile connected to the input tile.

        Raises:
            IndexError: If the input tile is not part of this edge.
        """
        return [tile for tile in self.tiles if tile != input_tile][0]

    def __repr__(self):
        """
        Returns a string representation of the edge, showing its names.

        Returns:
            str: A string representation of the edge.
        """
        return ' | '.join(self.names)
```

## GBWFC Logic Components


Now that we have implemented the core WFC components, we need to create a few additional classes:

- WFCTileSelector: Contains methods for selecting the next tile to collapse.
- WFCStateResolver: Includes methods that resolve a tile to a specific state.
- WFCNeighborhoodCollapser: Manages the updating of connected tile states based on custom rules.
- WFCLogger: Provides methods for logging custom information during each epoch of the WFC process.

By designing our code this way, we can easily add new methods in the future to each class to provide extra flexibility and handle different scenarios, all while maintaining a consistent structure.

At the beggining of each epochs, we use one of the WFCTileSelector class methods to select the next tile to collapse. We then, call one of the WFCStateResolver class methods to select a state and assign it to the current evaluated tile. Since all tiles are connected together by signal with the Observer design pattern, updating one tile'state will trigger a revaluation of neighboorhood 

### GBWFC TileSelector

In the WFCTileSelector class, we will implement a base method responsible for selecting the next tile to process. By default, this method will choose the tile with the lowest entropy in the model. In other words, at the start of a new WFC epoch or cycle, it identifies the tile with the fewest potential states it can collapse into. This approach is crucial to minimizing the likelihood of reaching an unsolvable model state. 

```python
class WFCTileSelector:
    """
    This class contains methods to select the next tile to process in the Wave Function Collapse (WFC) algorithm.
    The class provides logic for choosing the tile with the smallest entropy (the least number of possible states).
    If no tile can be found, the function returns None.
    """

    @staticmethod
    def get_min_entropy_tile(wfc):
        """
        Selects and returns the tile with the smallest number of possible states (entropy) that has not yet converged.
        If all tiles have converged, the function will return None.

        Parameters:
        wfc (WFC): The WFC object that contains the grid model and tiles to be processed.

        Returns:
        Tile or None: The tile with the smallest entropy that has not yet converged, or None if all tiles have converged.
        """
        
        # Gather the states of all tiles in the model
        tile_states = [tile.states for tile in wfc.model.tiles]

        try:
            # Find the smallest entropy (tile with the least number of states remaining)
            min_val = min(len(states) for states in tile_states if len(states) > 1)
        except ValueError:
            # If no tile has more than one state, the WFC has converged
            wfc.model.has_converged = True
            return None

        # Find the index of the tile with the smallest entropy
        smallest_entropy_tile_index = [len(states) for states in tile_states].index(min_val)
        
        # Retrieve the tile with the smallest entropy
        smallest_entropy_tile = wfc.model.tiles[smallest_entropy_tile_index]

        return smallest_entropy_tile

```

### GBWFC StateResolver

```python
import random

class WFCStateResolver:
    """
    Provides utility methods to determine the next state of a tile in a
    Wave Function Collapse (WFC) algorithm.

    This class contains methods for selecting and returning a valid next state
    for a given tile, based on the current state of the WFC object.
    """

    @staticmethod
    def get_random_tile_state(wfc, next_tile_to_collapse):
        """
        Selects and returns a random state for the given tile to collapse.

        Args:
            wfc: The Wave Function Collapse (WFC) object containing the overall state.
            next_tile_to_collapse: An object representing the tile to collapse.
                                   It must have a `states` attribute which is a list of possible states.

        Returns:
            list: A list containing a single randomly chosen state for the tile.

        Raises:
            ValueError: If the `states` attribute of the tile is empty.
        """
        if not next_tile_to_collapse.states:
            raise ValueError("The tile to collapse has no available states.")

        # Select one random state from the tile's available states.
        rnd_state = [random.choice(next_tile_to_collapse.states)]

        return rnd_state

```
### GBWFC NeighborhoodCollapser

```python
class WFCNeighborhoodCollapser:
    """
    Provides utility methods for determining the states of a destination tile
    in a Wave Function Collapse (WFC) algorithm, based on the relationship between
    a source tile, an edge, and a destination tile.

    This class supports extracting states for the destination tile that are
    unique compared to the states of the source tile.
    """

    @staticmethod
    def get_unique_destination_tile_states(wfc, source_tile, edge, destination_tile):
        """
        Returns a list of states for the destination tile that are not present
        in the source tile's states.

        Args:
            wfc: The Wave Function Collapse (WFC) object containing the overall state.
            source_tile: The tile serving as the reference for comparison.
                         It must have a `states` attribute which is a list of states.
            edge: An object representing the relationship or connection between
                  the source and destination tiles.
            destination_tile: The tile whose states are to be determined.
                              It must have a `states` attribute which is a list of states.

        Returns:
            list: A list of states in the destination tile that are not present
                  in the source tile's states.

        Raises:
            ValueError: If the `states` attribute of the source or destination tile is empty.
        """
        if not source_tile.states:
            raise ValueError("The source tile has no available states.")

        if not destination_tile.states:
            raise ValueError("The destination tile has no available states.")

        # Filter destination tile states to exclude any that are present in source tile states.
        new_destination_tile_states = [
            state for state in destination_tile.states if state not in source_tile.states
        ]

        return new_destination_tile_states
```

### GBWF CLogger

```python

class WFCLogger:
    """
    A utility class for logging and displaying information about the state of
    the Wave Function Collapse (WFC) algorithm.
    
    This class provides methods to visualize the current state of the WFC grid.
    """

    @staticmethod
    def print_wfc_matrix_state(wfc):
        """
        Displays the current state of the WFC matrix in a readable format.

        Args:
            wfc: The Wave Function Collapse (WFC) object containing the model and tiles.

        Raises:
            ValueError: If the WFC model does not contain any tiles.
        """
        tiles = wfc.model.tiles
        if not tiles:
            raise ValueError("The WFC model contains no tiles.")

        # Determine the width of the grid (assumes a square grid).
        grid_width = int(math.sqrt(len(tiles)))

        # Reshape the flat list of tiles into a 2D grid.
        matrix = [
            tiles[x : x + grid_width]
            for x in range(0, len(tiles), grid_width)
        ]

        # Helper to format tile states.
        def format_tile_states(states):
            return states[0] if len(states) == 1 else states

        # Print each row of the matrix.
        for row in matrix:
            formatted_row = [format_tile_states(tile.states) for tile in row]
            print(formatted_row)
```

## GBWFC Core Component

To integrate all components, we will now implement the following classes:
- WFCModel: Responsible for managing the runtime state of the WFC algorithm. It provides mechanisms to save and restore the model's state, facilitating backtracking when tiles fail to converge.
- WFC: The core engine of the WFC algorithm, controlling its initialization and execution.

### GBWFC Model

```python

class WFCModel:
    """
    Represents the core model for the Wave Function Collapse (WFC) algorithm.

    This class manages the states, tiles, and edges of the WFC grid, along with
    functionality for saving, restoring, and resetting states.
    """

    def __init__(self, states, tiles, edges):
        """
        Initialize the WFCModel with states, tiles, and edges.

        Args:
            states: A list of all possible states.
            tiles: A list of tile objects in the model.
            edges: A list of edge objects connecting the tiles.
        """
        self.states = states
        self.tiles = tiles
        self.edges = edges

        self.has_converged = False
        self.history = []
        self.steps = 0

    def save_model_state(self):
        """
        Save the current model state into the history for potential backtracking.

        The state is deep-copied to ensure the history is not affected by future changes.
        """
        self.history.append(
            copy.deepcopy((
                self.steps,
                self.states,
                self.tiles,
                self.edges
            ))
        )

    def restore_state(self, old_state):
        """
        Restore the model to a previously saved state.

        Args:
            old_state: A tuple containing the previous steps, states, tiles, and edges.
        """
        self.steps, self.states, self.tiles, self.edges = old_state
        self.has_converged = False

    def load_state_from_history(self, history_index):
        """
        Restore the model to a specific state in the history.

        Args:
            history_index: The index of the state in the history to restore.

        Raises:
            ValueError: If the provided history index is out of bounds.
        """
        if history_index >= len(self.history):
            raise ValueError(
                f"Provided history index {history_index} exceeds maximum index {len(self.history) - 1}."
            )

        self.restore_state(self.history[history_index])
        self.history = self.history[:history_index + 1]

        print(f"Restored tiles' states from history to step #{self.steps}.")

    def reset_tiles_states(self, tiles, recursivity_deepness=1):
        """
        Reset the states of the provided tiles to their initial states.

        Args:
            tiles: A list of tile objects to reset.
            recursivity_deepness: The depth of recursive resetting for neighboring tiles.
        """
        if recursivity_deepness == 0:
            return

        for tile in tiles:

            # Gather all neighboring tiles' states
            neighborhood_states_union = []
            neighborhood_tiles = [edge.get_connected_tile(tile) for edge in tile.edges]
            for neighborhood_tile in neighborhood_tiles:
                neighborhood_states_union.extend(neighborhood_tile.states)

            # Assign the difference between all possible states and neighboring states to the tile
            tile.states = list(set(self.states) - set(neighborhood_states_union))

            # Recursively reset the states of neighboring tiles
            self.reset_tiles_states(
                tiles=neighborhood_tiles,
                recursivity_deepness=recursivity_deepness - 1
            )

        self.has_converged = False
```

### GBWFC
```python
class WFC:
    """
    Core class for the Wave Function Collapse (WFC) algorithm.

    This class manages the state of the WFC model, coordinates the collapse process,
    and ensures the propagation of constraints through the tile neighborhood.
    """

    def __init__(
        self,
        wfc_model,
        wfc_tile_selector_function,
        wfc_state_resolver_function,
        wfc_neighborhood_collapse_function,
        wfc_log_function=None,
        autosave_wfc_model_every_n_steps=50,
        **kwargs
    ):
        """
        Initialize the WFC object.

        Args:
            wfc_model: The WFC model object containing the grid and tiles.
            wfc_tile_selector_function: Function to select the next tile to collapse.
            wfc_state_resolver_function: Function to determine the next state of a tile.
            wfc_neighborhood_collapse_function: Function to update the states of neighboring tiles.
            wfc_log_function: Optional logging function to record WFC progress.
            autosave_wfc_model_every_n_steps: Frequency (in steps) to autosave the model state.
            **kwargs: Additional custom attributes to add to the WFC instance.
        """
        self.model = wfc_model
        self.wfc_tile_selector_function = wfc_tile_selector_function
        self.wfc_state_resolver_function = wfc_state_resolver_function
        self.wfc_neighborhood_collapse_function = wfc_neighborhood_collapse_function
        self.wfc_log_function = wfc_log_function

        # WFC configuration
        self.autosave_wfc_model_every_n_steps = autosave_wfc_model_every_n_steps

        # Add any additional custom attributes
        for key, value in kwargs.items():
            setattr(self, key, value)

    def collapse_neighborhood_tile_states(self, source_tile):
        """
        Collapse the states of tiles in the neighborhood of the source tile.

        Args:
            source_tile: The tile whose neighbors need to be updated.
        """
        for edge in source_tile.edges:
            destination_tile = edge.get_connected_tile(source_tile)

            # Default to the current states of the destination tile
            new_states = destination_tile.states

            # Update states only if the source tile has converged
            if source_tile.has_converged:
                new_states = self.wfc_neighborhood_collapse_function(
                    self, source_tile, edge, destination_tile
                )

            # Trigger an update if the states have changed
            if new_states != destination_tile.states:
                print(f"Updating {destination_tile} states from {destination_tile.states} --> {new_states}")
                destination_tile.states = new_states
                # Recursively collapse the neighborhood of the updated tile
                self.collapse_neighborhood_tile_states(source_tile=destination_tile)

    def run(self):
        """
        Execute the WFC algorithm until convergence.

        Raises:
            ValueError: If the model has already converged.
        """
        if self.model.has_converged:
            raise ValueError("This WFC has already converged. Update the model state and retry.")

        print("Running WFC!!\n")

        while not self.model.has_converged:
            print("\n######################")
            print(f"WFC STEP {self.model.steps}")
            print("######################\n")

            # Autosave the current model state at regular intervals
            if self.model.steps % self.autosave_wfc_model_every_n_steps == 0:
                self.model.save_model_state()

            # Log progress if a log function is provided
            if self.wfc_log_function:
                self.wfc_log_function(self)

            # Select the next tile to collapse using the selector function
            next_tile_to_collapse = self.wfc_tile_selector_function(wfc=self)

            # If no tile can be collapsed, mark the model as converged
            if next_tile_to_collapse is None:
                self.model.has_converged = True
                break

            # Collapse the selected tile's states
            new_tile_state = self.wfc_state_resolver_function(
                wfc=self, next_tile_to_collapse=next_tile_to_collapse
            )
            next_tile_to_collapse.states = new_tile_state

            # Update the neighborhood of the collapsed tile
            self.collapse_neighborhood_tile_states(next_tile_to_collapse)

            # Increment the step counter
            self.model.steps += 1

        print("Done converging WFC!\n")

        # Final autosave of the model state
        self.model.save_model_state()

        # Log final state if a log function is provided
        if self.wfc_log_function:
            self.wfc_log_function(self)

```
At this stage, you should now have all the foundational components necessary to implement your GBWFC algorithm effectively.

Next, we’ll focus on crafting specialized methods tailored to generate Sudoku grids. Sudoku serves as an excellent test case due to its unique constraints involving non-adjacent tiles: each cell’s value must remain distinct across its row, column, and subgrid. This makes it a compelling example to demonstrate the capabilities and flexibility of GBWFC in handling complex, non-local constraints.

## Sudoku GBWFC Parts:

To specialize our WFC for generating Sudoku grids, we need to implement the following functions that will provide the necessary components:

- generate_sudoku_states: Returns all possible Sudoku states (values from 1 to 9).
- generate_sudoku_tiles: Returns all Sudoku tiles instances (81 tiles for default grid's 9x9 format).
- generate_sudoku_edges: Returns all tile's edges instances that link them with same row, column, and subgrid tiles.

### generate_sudoku_states

```python
def generate_sudoku_states():
    """
    Generates a list of all possible Sudoku states (values 1 through 9).

    This function creates a list of State objects, where each object represents 
    a possible value (from 1 to 9) for a Sudoku cell. The State object is initialized 
    with an id corresponding to the value and a state as a string representation of that value.

    Returns:
        list: A list of State objects, each representing a possible Sudoku value.
    """
    min_val = 1
    max_val = 9
    
    # Initialize an empty list to store the possible states
    all_possible_states = []

    # Loop through each number from min_val to max_val (inclusive)
    for i in range(min_val, max_val + 1):
        # Create a new State object for each number and append it to the list
        all_possible_states.append(State(id=i, state=str(i)))

    # Return the list containing all possible Sudoku states
    return all_possible_states

```

### generate_sudoku_tiles
```python
def generate_sudoku_tiles(sudoku_subgrid_width_count, states):
    """
    Generates a list of Sudoku tile instances.

    This function creates instances of Tile objects for a Sudoku grid, with each
    tile being associated with a set of possible states and positioned within the grid. 
    It calculates the Sudoku subgrid index based on the tile's position within the grid.

    Args:
        sudoku_subgrid_width_count (int): The number of subgrids in one dimension of the Sudoku grid.
        states (list): A list of possible states for each tile (e.g., values 1 through 9).

    Returns:
        list: A list of Tile instances representing the Sudoku grid.
    """
    tiles = []
    grid_cells_width = sudoku_subgrid_width_count * 3  # Width of the Sudoku grid, considering 3x3 subgrids

    # Loop through each grid position to create tile instances
    for x in range(grid_cells_width):
        for y in range(grid_cells_width):
            
            # Generate unique tile ID
            tile_id = len(tiles)

            # Append the new Tile instance to the list
            tiles.append(
                Tile(
                    id=tile_id,
                    states=states,
                    edges=[],  # Placeholder for future edge data (if applicable)
                    position=(x, y),  # Position of the tile in the grid (x, y)
                    sudoku_subgrid_index=(
                        (tile_id % grid_cells_width) // 3 +  # Horizontal subgrid index
                        3 * (tile_id // (grid_cells_width * 3))  # Vertical subgrid index
                    ),
                )
            )

    return tiles
```

### generate_sudoku_edges

```python
def generate_sudoku_edges(tiles):
    """
    Generates edges between tiles that are in the same row, column, or Sudoku subgrid.

    This function creates edges between tiles that share a row, column, or Sudoku subgrid.
    It ensures that no edge is duplicated between the same pair of tiles.

    Args:
        tiles (list): A list of Tile objects representing the Sudoku grid.

    Returns:
        list: A list of Edge objects representing the connections between tiles.
    """
    edges = []
    edge_id = 1  # Initialize edge ID counter

    # Loop through each tile as the source tile
    for source_tile in tiles:
        # Retrieve the position of the source tile
        source_tile_x, source_tile_y = source_tile.position

        # Loop through each tile as the destination tile
        for destination_tile in tiles:
            # Skip processing the same tile over itself
            if source_tile is destination_tile:
                continue

            # Retrieve the position of the destination tile
            destination_tile_x, destination_tile_y = destination_tile.position

            # Check if the destination tile shares the same row, column, or Sudoku subgrid
            if source_tile_x == destination_tile_x or \
               source_tile_y == destination_tile_y or \
               source_tile.sudoku_subgrid_index == destination_tile.sudoku_subgrid_index:

                # Check if an edge already exists between the source and destination tile
                existing_edge = [edge for edge in source_tile.edges if destination_tile in edge.tiles]
                if existing_edge:
                    continue

                # Create a new edge between the source and destination tile
                edge = Edge(
                    id=edge_id,
                    names=[
                        '{}__{}'.format(str(source_tile.position), str(len(source_tile.edges) + 1)),
                        '{}__{}'.format(str(destination_tile.position), len(destination_tile.edges) + 1)
                    ],
                    tiles=[source_tile, destination_tile]
                )

                # Add the edge to both the source and destination tiles
                source_tile.edges.append(edge)
                destination_tile.edges.append(edge)

                # Increment the edge ID counter for the next edge
                edge_id += 1

    # Return the list of all created edges
    return edges
```

Design a custom exception to handle scenarios where tiles in a GBWFC implementation fail to resolve to a valid state paired with an autofix_empty_states_tiles method, which detects and resolves such tiles by assigning fallback or calculated states, ensuring the algorithm progresses despite convergence issues. 

This method helps maintain stability in procedural generation while providing a mechanism to gracefully handle errors without restarting the entire process.

```python

class UnresolvableWfcException(Exception):
    """
    Exception raised when the WFC model encounters a situation that cannot be resolved.
    """
    def __init__(self, *args: object) -> None:
        super().__init__(*args)

def autofix_empty_states_tiles(wfc):
    """
    Attempts to resolve empty tile states by recursively rolling back the WFC model 
    and applying the Memento pattern. This ensures that the WFC outputs remain viable 
    even when encountering empty tile states.

    Args:
        wfc: The WFC instance that is attempting to resolve the empty states.

    Raises:
        UnresolvableWfcException: If the WFC model cannot be resolved after rolling 
                                   back to all previous states.
    """
    empty_states_tiles = lambda wfc: [tile for tile in wfc.model.tiles if not tile.states]
    history_index = 1

    while len(empty_states_tiles(wfc)) > 0:
        # If history_index exceeds the available history, raise an exception
        if history_index >= len(wfc.model.history):
            raise UnresolvableWfcException("Unable to find any resolvable states of the WFC")

        # Reload previous state from the history
        wfc.model.load_state_from_history(len(wfc.model.history) - history_index)
        history_index += 1

        # If there are still empty states, continue to the next rollback
        if len(empty_states_tiles(wfc)) > 0:
            continue

        # Try resolving with the rolled-back version
        wfc.run()
```

## Testing Everything

Develop a primary generate_sudoku_grid method that encapsulates all the necessary logic to create a Sudoku grid. This method should return the completed grid as a string of characters, effectively representing the Sudoku puzzle in a compact and accessible format.
```python

def generate_sudoku_grid():
    """
    Generates a complete Sudoku grid using the WFC (Wave Function Collapse) algorithm.

    This function initializes the Sudoku grid with states, tiles, and edges, and then
    attempts to resolve the grid using the WFC model. If the grid contains any unconverged
    cells, it will attempt to fix them by rolling back the WFC model and rerunning the process.

    Returns:
        list: A list of solved Sudoku tile states.
    """
    # Generate the initial Sudoku states, tiles, and edges
    sudoku_init_states = generate_sudoku_states()
    sudoku_tiles = generate_sudoku_tiles(sudoku_subgrid_width_count=3, states=sudoku_init_states)
    sudoku_edges = generate_sudoku_edges(sudoku_tiles)

    # Create the Sudoku WFC model
    sudoku_wfc = WFC(
        wfc_model=WFCModel(
            states=sudoku_init_states,
            tiles=sudoku_tiles,
            edges=sudoku_edges
        ),
        wfc_tile_selector_function=WFCTileSelector.get_min_entropy_tile,
        wfc_state_resolver_function=WFCStateResolver.get_random_tile_state,
        wfc_neighborhood_collapse_function=WFCNeighborhoodCollapser.get_unique_destination_tile_states,
        wfc_log_function=WFCLogger.print_wfc_matrice_state,
        autosave_wfc_model_every_n_steps=10
    )

    # Run the WFC model to solve the Sudoku grid
    sudoku_wfc.run()

    # Check if there are any unconverged cells (empty tile states)
    unconverged_cells = [len(tile.states) == 0 for tile in sudoku_wfc.model.tiles]

    # If there are unconverged cells, try to fix them
    if any(unconverged_cells):
        try:
            autofix_empty_states_tiles(sudoku_wfc)
        except UnresolvableWfcException:
            return generate_sudoku_grid()  # Retry the generation if it remains unsolvable

    # Return the solved Sudoku grid as a string sequence so we can turn it into
    # a detail attribute in houdini later on
    sudoku_cells = [tile.states[0].state for tile in sudoku_wfc.model.tiles]
    
    return ''.join(sudoku_cells)

generate_sudoku_grid()
```

If you were to run this, you should be ending with something that looks like this in the shell.
![wfc output](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/wfc/wfc5.png?raw=true)

# GBWFC In Houdini

Now that the GBWFC implementation is functional, I've created a Houdini setup that allows the Sudoku grid to be populated from our implementation procedurally. 

This setup makes the process more visual, providing a real-time view of the grid in the viewport. It not only enhances the user experience but also helps in identifying issues or bugs more easily by allowing for immediate feedback and observation of how the grid evolves.

To start, we create a grid mesh and calculate the centroid of each cell. This mesh represents the Sudoku layout, where each cell corresponds to a position in the puzzle. The centroids serve as spatial anchors that will be use later to place numbers.
![wfc setup create grid](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/wfc/wfc1.png?raw=true)

Next, we use a font node in conjunction with a switch node. The switch node dynamically selects the corresponding number from the Sudoku grid string at the current iteration index. This number is then converted into a visual representation through the font node. 

![wfc setup create tiles](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/wfc/wfc3.png?raw=true)

The output is subsequently copied to the centroid of each associated cell point in the grid mesh, effectively assigning and displaying the Sudoku values in their respective positions on the grid. 
![wfc setup populate integer](https://github.com/logan169/logan169.github.io/blob/master/assets/images/posts_images/wfc/wfc.png?raw=true)

As seen in the previous screenshot, our GBWFC implementation successfully enforces all constraints for rows, columns, and subgrids, ensuring the Sudoku grid remains valid and adheres to the rules at every step of the generation process.

# Final Thoughts

In this article, we demonstrated the implementation of a Graph-Based Wave Function Collapse (GBWFC) backbone algorithm designed for flexibility. This framework can be seamlessly adapted to address diverse problem-specific constraints while maintaining the ability to generate consistent and coherent procedural content.

Our implementation allows flexibility to modify the core GBWFC behavior during convergence. Depending on the specific requirements of a problem, you can extend logic components in the following classes:

- WFCTileSelector: Customize how the next tile is chosen for collapse.
- WFCStateResolver: Define how a tile's state is selected during collapse.
- WFCNeighborhoodCollapser: Adjust how waves propagate to update neighboring tile states.

This modularity ensures the algorithm's adaptability to diverse procedural generation tasks. For instance, when creating a kitchen layout, instead of purely random tile selection, the GBWFC could prioritize placing essential, larger assets like a table with chairs, an oven, or a fridge early in the process. Once these key items are positioned, the algorithm could revert to its default random tile-picking method. This strategy ensures that critical elements are properly integrated into the layout, preventing conflicts and maintaining functional and spatial coherence.

Additionally, custom edge types could be implemented to address various constraint types effectively. For instance, in a Nested Recursive Multi-Level GBWFC approach (discussed in the motivation section), each tile in a subgrid could be linked to its parent converged tile. Metadata from the parent tile—such as biome, neighborhood type, house type, or room type—would guide the selection of assets eligible for instantiation within the subgrid as a new GBWFC runs at that level. This structure ensures context-sensitive asset placement and enforces coherence across hierarchical levels of generation.

Finally, instead of randomly choosing a state to collapse a tile, a Markov chain could guide this decision based on probabilistic data derived from the parent cell's type. For example, in a GBWFC subgrid designed for vegetation placement, the biome metadata of the resolved parent tile could inform a vegetation type distribution. This method allows the algorithm to produce outputs that more accurately reflect realistic environments, leveraging patterns and correlations to generate contextually appropriate and natural results.

I hope you found this post helpful and that it inspires those in the cinema industry to explore integrating this powerful algorithm into their workflows. If you create something using this approach, feel free to share a picture—I’d love to see the results of your creativity!