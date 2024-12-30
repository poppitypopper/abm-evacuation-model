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

### Barricading and moving the exit points
This section we move the exit to be all the pixels on the left edge of the screen by
```NetLogo
to set-exits
  ask patches with [pxcor = min-pxcor] [  ; Leftmost column patches
    set pcolor red
  ]
end
```





