# General

## Variables
Variable | Description
:---     | :---
`n.mpc.tex` | current execution time horizon in (s)
`n.mpc.t`   | current time in (s)
`n.mpc.tp`  | current prediction time in (s) == `getvalue(n.tf)` if `n.s.finalTimeDV==false` (otherwise it is not applicable)

## Settings
The settings are defined using the `configureMPC!()` function where the following keys can be passed.

|Variable                 | Key | Possible Values    | Description |
|:---                      :---    | :---
|`n.mpc.s.mode`    |`:mode` | `:OCP`,  `:IP`, `:IPEP`, `:EP` | identifies the  `simulationMode`|
|`n.mpc.s.predictX0` | `:predictX0`| `true` or `false`| bool to indicate if `X0` will be predicted
|`n.mpc.s.fixedTex`   | `:fixedTex`| `true` or `false` | bool to indicate if `n.mpc.tex` is fixed
|`n.mpc.s.IPKnown`  |`:IPKnown` |`true` or `false` | bool to indicate if the ``InternalPlant`` is known
|`n.mpc.s.saveMode`  |`:saveMode` |`:all` or `:none` | indicates the mode that is to be utilized to save the results

As an example:
```julia
configureMPC!(n,(:mode=>:OCP))
```

## Flags
The value for all of the flags is `true` or `false`.

Variable | Initial Value | Description
:---     | :--- | :---
`n.mpc.flags.goalReached` | `false`| bool to indicate if the goal has been reached

# simulationModes
There are four different possible values for `simulationMode` that can be set by the `:mode` key as described above.

## OCP (`:OCP`)
In this case, the plant model is the set of differential equations defined within the OCP. The entire OCP is still defined entirely outside of the MPC_Module. For instance, ``n.numStates`` and ``n.numControls`` represent the number of states and controls for the OCP, respectively.

To keep track of all of the `n.X0`s that are passed to the OCP, we define an time stamped array is defined called ``n.mpc.X0ocp``. The first element in ``n.mpc.X0ocp`` is automatically set to ``n.X0`` after calling ``initializeMPC()``.


---
**NOTE**

For all modes, the initial state in optimization, ``n.X0``, is set using the ``define()`` function. If needed, it can be changed before the initial optimization using the ``updateX0!()`` function.

---

``n.mpc.X0ocp`` and ``n.U`` are passed to these differential equations to simulate the plant for a time given by ``n.mpc.tex``, the final state is stored in the next element in the ``n.mpc.X0ocp`` array. Then, ``n.X0`` is updated to ``n.mpc.X0ocp[end]``.

---
**NOTE**

Since the plant is known in this case, ``n.X0`` is updated using future knowledge of the state. So, the simulation is "cheating" in a way, by assuming perfect knowledge of where the vehicle will be after ``n.mpc.tex``.
```
                           Given
                             |
                             V
        OCP solving    n.mpc.X0ocp[end]
      -------------->
      x----------------------x----------------------x
  n.mpc.t0         (n.mpc.t0 + n.mpc.tex)
```
---

## InternalPlant (`:IP`)
In this case, the OCP is solved controls are sent to

### Variables
The states and controls in this model may not be the same as they are in the ``OCP`` and thus ``n.numStates`` and ``n.numControls`` may not represent the number of states and controls, respectively for the ``InternalPlant``.

Variable | Description
:--- | :---
`n.mpc.numControlsIP` | number of control variables for the `InternalPlant`
`n.mpc.numStatesIP` | number of state variables for the `InternalPlant`
`n.mpc.IPeMap` | mapping


As an example, assume that in the OCP, the [KinematicBicycle](https://github.com/JuliaMPC/VehicleModels.jl/blob/master/src/KinematicBicycle/KinematicBicycle.jl) is used. The state and controls should be defined as:
```
states!(n,[:x,:y,:psi,:ux])
controls!(n,[:sa,:ax])
```
Then assume that the [ThreeDOFv1](https://github.com/JuliaMPC/VehicleModels.jl/blob/master/src/Three_DOF/Three_DOF.jl) is used for the `InternalPlant`. The states and controls would be defined as:
```
statesIP!(n,[:x,:y,:v,:r,:psi,:sa,:ux,:ax])
controlsIP!(n,[:sr,:jx])
```

To calculate the error array, each state variable in the `OCP` is compared with each state and control variable in the `IP`. The result is stored in a map called `n.mpc.mIP`. For the aforementioned example, that map look like:

```julia

````

### State Equations
For this mode, the plant model is defined by `plantEquations` within NLOptControl.

This is simply done as
```julia
 n.mpc.plantEquations = KinematicBicycle
 ```
 where `KinematicBicycle` is a function that solves a set of ODEs given a control, an initial state, and a simulation time. For an example see [VehicleModels.jl](https://github.com/JuliaMPC/VehicleModels.jl/blob/master/src/KinematicBicycle/KinematicBicycle.jl#L24).


## InternalExternalPlant (`:IPEP`)
In this mode, there is an `IP` that can be used to help predict X0 (X0p) for an `EP`.


This option can be useful when the OCP needs to be solved quickly and a more complicated model (`IP`) may give better `X0p`. Also, in developing functionality to determine the error in `X0p`. That is without having to deal with an external simulation can methods be developed to improve `X0p`.

## ExternalPlant (`:EP`)
A set of `n.X` and `n.U` makes up `UEX` and is fed directly to an `EP`.

# Synchronization
Synchronizing MPC systems is critical for performance and safety.

## Fixed Execution Horizon

## Variable Execution Horizon
Currently, there is no functionality for this. But, this may be useful and it would augment a prediction of the time as well as `X0`. So, to account for this possible expansion, a predicted time (very simply the current time plus `n.mpc.tex` for the fixed execution horizon case) is added to `X0p`.

This is the case, where the OCP is being solved as quickly as possible. In this case predicting `n.r.t_solve`( roughly equal to `n.mpc.tex`) is a challenging problem because there is no guarantee that the OCP will be solved in a particular amount of time. A simple way to predict ``n.r.t_solve`` is to average several of the previous `n.r.t_solve` values.  


# Error
Evaluating the error of the prediction of `X0` is important. Additionally, evaluating the tracking error (or following error) for each state is also important. Fortunately, there is built in functionality to calculate and save these errors.  

## OCP
Currently there is no need to quantify error in this case.

## InternalPlant
In this case, the errors are calculated

## ExternalPlant

# InternalExternalPlant
This is the most complicated mode and there can be errors

# Results and Variables
The following tables describe the results and are organized by mode.

Concern is that there may be too much data to save.

## OCP
Variable | Description
:--- | :---
`n.X0`          | current initial state
`n.r.X`         | current solution for states
`n.r.U`         | current solution for controls
`n.r.t_st`      | corresponding time for states (and controls minus the last entry)
`n.mpc.r.dfsX0` | DataFrame of all `n.X0` arrays used in optimization each appended with `n.mpc.t`

## InternalPlant
Variable | Description
:--- | :---
`n.mpc.r.UIP` | array of latest matrix of controls for the `InternalPlant`
`n.mpc.r.dfsUIP`| DataFrame of all matrices of `n.mpc.r.UIP`
`n.mpc.r.X0pIP` | array of latest prediction of `n.mpc.r.X0aIP`
`n.mpc.r.dfsX0pIP` | DataFrame of all `n.mpc.r.X0pIP` arrays
`n.mpc.r.X0aIP` | array of latest actual initial state for the `InternalPlant`
`n.mpc.r.dfsX0aIP` | DataFrame of all `n.mpc.r.X0aIP` arrays
`n.mpc.r.X0pIPe` | array of latest error in between `n.mpc.r.X0pIP` and `n.mpc.r.X0aIP`
`n.mpc.r.dfsX0pIPe` | DataFrame of all `n.mpc.r.X0pIPe` arrays

# need a mapping between states and controls for different models to calculate error

## ExternalPlant
Variable | Description
:--- | :---
`n.mpc.r.UIP` | latest matrix of controls for the `InternalPlant`
`n.mpc.r.dfsUIP` | DataFrame of all matrices of `n.mpc.r.UIP`

## Error
Variable | Description
:--- | :---
`n.mpc.r.X0IPE` |
`n.mpc.r.dfsX0IPE` | DataFrame

errorX0
errorTraj

predicted initial states