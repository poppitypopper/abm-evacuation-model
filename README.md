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

The explanation of the workings of the algorithm is not provided. If required, can be found [here](https://www.cs.us.es/~fsancho/Blog/posts/General_A_Solver_NetLogo.md.html).
#### Netlogo Implementation

  ```Netlogo
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
  ask patches with [pxcor = 0 and pycor > -8 and pycor < 8] [
    set pcolor black]
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
![](https://github.com/poppitypopper/abm-evacuation-simulation/blob/main/demo-files/a*%20interactive.gif)

### Involving agents
Now that we can handle barricading, we can use our code to define barricades and spawn turtles randomly, give them attributes that describe their behaviour and simulate different barricading methods. 

Increase panic based on neighbors' average speed
```NetLogo
    let neighbors turtles in-radius 3
    if any? neighbors [
      set panic mean [speed] of neighbors
    ]

    set speed 0.5 + panic
```
Avoid other turtles within distance-preference
```Netlogo
    let too-close turtles in-radius distance-preference
    if any? too-close [
      let closest min-one-of too-close [distance myself]
      face closest
      rt 90 
    ]
```

 







