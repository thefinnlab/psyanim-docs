# <ins>Playfight v2 FSM Design</ins>

## 1. Algorithm Overview

The `state machine` for the `Playfight v2` algorithm has `4 states` (and `6 possible transitions`) as shown on the diagram below:

<p align="center" style="font-size: 12px;">
    <img src="./imgs/playfight_v2_fsm.jpg"/>
    <br>Playfight v2 State Machine Diagram
</p>

In the diagram above, the `transitions` are numbered 1 through 6.  Each numbered `transition` is triggered according to the following rules / conditions:

1. If `fleeOrChargeWhenAttacked` is `true`, the `other agent` is in a `charge state` and within the `panic distance`, this transition occurs with a probability of `wanderFleeRate`.

2. If the `other agent` stops charging or this `agent` has been in state for a duration of `maxFleeDuration`.

3. If `fleeOrChargeWhenAttacked` is `true`, the `other agent` is in a `charge state` and within the `panic distance`, this transition occurs with a probability of `(1 - wanderFleeRate)`.

4. If this `agent` has been in state for a duration of `chargeDelay`.

5. If this `agent` has been in state for a duration of `maxChargeDuration` or this `agent` makes contact with the `other agent`.

6. If this `agent` has been in state for a duration of `breakDuration`.

When the PsyanimFSM `debug` flag is set to `true`, the agents have a color-coded debug visualization with the following rules:

- *Green*: Agent is in `Wander` state and no `charge` detected.
- *Yellow*: Agent is in `Wander` state and detected `other agent` charging, but not within `panic distance` yet
- *Blue*: Agent is in `Flee` state
- *Orange*: Agent is in `ChargeDelay` state, where it is wandering for a duration of `chargeDelay` before transitioning to `Charge` state
- *Red*: Agent is in `Charge` state

## 2. Algorithm Notes

The `parameters` in this algorithm have to be carefully `tuned` together.

If `fleeOrChargeWhenAttacked` is `true`, the agents' behavior will be very sensitive to the values chosen for `wanderFleeRate` and `maxChargeDuration`.

If the `maxChargeDuration` is very long, e.g. > 1 second, and the `wanderFleeRate` is fairly high, e.g. > 0.6, there is a decent probability of getting 2 to 3 back-to-back transitions to the `flee` state from an agent who is getting charged at.

The `maxChargeDuration` is also affected by parameters such as the `maxTargetDistanceForCharge`.  If the latter is larger, the `maxChargeDuration` needs to be larger or the agents will often not make contact after a charge.

That said, increasing the `maxChargeDuration` is best accompanied by a decrease of the `wanderFleeRate` for the reasons mentioned above.

These are only some of the things to watch out for when `tuning` these `algorithm parameters`, as there are many relationships between them that affect the behavior of the two agents.