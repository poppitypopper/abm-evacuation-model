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
<Details> 
  <Summary> Expand Code (119 Lines) </Summary>

  ```Netlogo
dfjgdf
dsf
```









