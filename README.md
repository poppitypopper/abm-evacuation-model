# Modelling crowd evacuation using ABM
Crowd management is an important but often overlooked avenue of event management. Events handling heavy volumes of participants need effective and optimised evacuation strategies in cases of malfunction or disaster. Here we model crowd behaviour using the concept of agent-based-modelling in Netlogo with several parameters.

## Defining the scope of the project
1) Setting up the base simulation(s) of exit points and crowd behaviour (speed and distance from another agent)
2) Comparing evacuation times with heuristic barricading
3) Include more complex agent behaviour like panic and hysteria

## Explaining the code
### Outlining a skeletal agent model
Lets initialise an agent (a turtle) with the properties : 
1) Always head towards the `exit point` in steps of `1`
2) Move away from any turtle who is in proximity less than `prox`
3) Once an agent hits an `exit point`, they are removed from the simulation

```NetLogo
to go
  ask turtles [
    move-towards-exit
    if pcolor = red [ die ]
  ]
  tick
end

to move-towards-exit
  let exit one-of patches with [pcolor = red] ; Find the exit
  face exit
  fd 1
  if any? turtles in-radius 1 [
    rt random 90  ; Avoid others
  ]
```

Running the skeletal simulation with an exit at (10,10) colored red.

![](https://github.com/poppitypopper/abm-evacuation-model/blob/main/demo-files/skeletal-model-100.gif)

### Handling barricading
To model the behaviour of turtles when faced with a barrier, the A<sup>*</sup> pathfinding algorithm is used here. For this algorithm to work we define 
1) A start point
2) An end point
3) A barrier in between

The algorithm highlights the shortest path between the start and end points using a sum of absolute and heuristic information. We assume that the agents know exactly "how far" the exit is, since we feed the "euclidean distance" as the heuristic function and the absolute distance as the determinate function.

The explanation of the workings of the algorithm is not provided. If required, can be found [here](https://www.cs.us.es/~fsancho/Blog/posts/General_A_Solver_NetLogo.md.html). Python and Netlogo implementations have been discussed.


#### Netlogo Implementation
```NetLogo
patches-own [
  f-score  ; total estimated cost
  g-score  ; cost from start to current
  parent   ; parent patch for reconstructing path
  visited? ; whether patch has been visited
]

globals [
  start-patch  ; starting point
  end-patch    ; destination
  open-set     ; patches to be evaluated
  path-found?  ; whether path was found
]

to setup
  clear-all
  reset-ticks
  
  ; Initialize patches
  ask patches [
    set f-score 99999
    set g-score 99999
    set visited? false
    set parent nobody
    set pcolor white
  ]
  
  ; Set start and end points
  set start-patch patch -10 0
  set end-patch patch 10 0
  ask start-patch [ set pcolor green ]
  ask end-patch [ set pcolor red ]
  
  ; Create some barriers (can be modified)
  ask patches with [abs pxcor < 8 and abs pycor < 2] [
    if random 100 < 30 [ set pcolor black ]  ; 30% chance of barrier
  ]
end

to find-path
  ; Create pathfinding turtle
  create-turtles 1 [
    set color blue
    set size 1
    move-to start-patch
  ]
  
  ; Initialize pathfinding variables
  set open-set (list start-patch)
  ask start-patch [
    set g-score 0
    set f-score [distance end-patch] of self
  ]
  
  set path-found? false
  
  ; Main pathfinding loop
  while [not empty? open-set and not path-found?] [
    let current first open-set
    set open-set but-first open-set
    
    ask current [
      set visited? true
      
      ; Check if we reached the goal
      if self = end-patch [
        set path-found? true
        stop
      ]
      
      ; Check all neighboring patches
      ask neighbors4 [
        ; Skip if barrier or already visited
        if pcolor != black and not visited? [
          let tentative-g [g-score] of myself + 1
          
          ; If this path is better than previous
          if tentative-g < g-score [
            set parent myself
            set g-score tentative-g
            set f-score tentative-g + distance end-patch
            
            ; Add to open set if not already there
            if not member? self open-set [
              set open-set (fput self open-set)
              ; Sort open-set by f-score
              set open-set sort-by [[f-score] of ?1 < [f-score] of ?2] open-set
            ]
          ]
        ]
      ]
    ]
  ]
  
  ; Trace path if found
  if path-found? [
    let current end-patch
    while [current != start-patch] [
      ask current [
        set pcolor yellow
        set current parent
      ]
    ]
  ]
end

to go
  setup
  find-path
end
```

#### Python Implementation
```Python3
import numpy as np
from dataclasses import dataclass
from typing import List, Tuple, Optional
import heapq

@dataclass
class Node:
    position: Tuple[int, int]
    g_score: float = float('inf')
    f_score: float = float('inf')
    parent: Optional['Node'] = None
    
    def __lt__(self, other):
        return self.f_score < other.f_score

class PathFinder:
    def __init__(self, width: int, height: int):
        self.width = width
        self.height = height
        self.grid = np.zeros((height, width), dtype=int)  # 0: empty, 1: barrier
        self.start = None
        self.end = None
        
    def set_start(self, x: int, y: int):
        self.start = (x, y)
        
    def set_end(self, x: int, y: int):
        self.end = (x, y)
        
    def add_barrier(self, x: int, y: int):
        self.grid[y, x] = 1
        
    def get_neighbors(self, node: Node) -> List[Tuple[int, int]]:
        x, y = node.position
        neighbors = []
        
        # Check all 4 adjacent positions (up, right, down, left)
        for dx, dy in [(0, 1), (1, 0), (0, -1), (-1, 0)]:
            new_x, new_y = x + dx, y + dy
            
            # Check if position is within grid bounds and not a barrier
            if (0 <= new_x < self.width and 
                0 <= new_y < self.height and 
                self.grid[new_y, new_x] == 0):
                neighbors.append((new_x, new_y))
                
        return neighbors
    
    def manhattan_distance(self, pos1: Tuple[int, int], pos2: Tuple[int, int]) -> int:
        return abs(pos1[0] - pos2[0]) + abs(pos1[1] - pos2[1])
    
    def find_path(self) -> List[Tuple[int, int]]:
        if not self.start or not self.end:
            raise ValueError("Start and end points must be set")
            
        # Initialize start node
        start_node = Node(self.start)
        start_node.g_score = 0
        start_node.f_score = self.manhattan_distance(self.start, self.end)
        
        # Initialize open and closed sets
        open_set = [start_node]
        closed_set = set()
        
        # Dictionary to keep track of nodes by position
        node_dict = {self.start: start_node}
        
        while open_set:
            current = heapq.heappop(open_set)
            
            # If we reached the end
            if current.position == self.end:
                path = []
                while current:
                    path.append(current.position)
                    current = current.parent
                return path[::-1]  # Reverse path to get start→end
            
            closed_set.add(current.position)
            
            # Check all neighbors
            for neighbor_pos in self.get_neighbors(current):
                if neighbor_pos in closed_set:
                    continue
                    
                # Calculate tentative g_score
                tentative_g = current.g_score + 1
                
                # Get or create neighbor node
                if neighbor_pos not in node_dict:
                    neighbor = Node(neighbor_pos)
                    node_dict[neighbor_pos] = neighbor
                else:
                    neighbor = node_dict[neighbor_pos]
                
                # If we've found a better path
                if tentative_g < neighbor.g_score:
                    neighbor.parent = current
                    neighbor.g_score = tentative_g
                    neighbor.f_score = tentative_g + self.manhattan_distance(neighbor_pos, self.end)
                    
                    if neighbor_pos not in [n.position for n in open_set]:
                        heapq.heappush(open_set, neighbor)
        
        return []  # No path found

# Example usage
def main():
    # Create a 20x20 grid
    finder = PathFinder(20, 20)
    
    # Set start and end points
    finder.set_start(2, 2)
    finder.set_end(15, 15)
    
    # Add some random barriers
    np.random.seed(42)  # For reproducibility
    for _ in range(100):
        x = np.random.randint(0, 20)
        y = np.random.randint(0, 20)
        if (x, y) != finder.start and (x, y) != finder.end:
            finder.add_barrier(x, y)
    
    # Find path
    path = finder.find_path()
    
    # Visualize the result
    grid_viz = np.full((20, 20), '.', dtype=str)
    grid_viz[finder.grid == 1] = '#'  # Barriers
    
    if path:
        for x, y in path:
            grid_viz[y, x] = '*'  # Path
            
    # Mark start and end
    sx, sy = finder.start
    ex, ey = finder.end
    grid_viz[sy, sx] = 'S'
    grid_viz[ey, ex] = 'E'
    
    # Print the grid
    for row in grid_viz:
        print(' '.join(row))
        
if __name__ == "__main__":
    main()
```









