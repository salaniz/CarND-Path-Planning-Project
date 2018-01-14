## Reflection

The path planning model follows 3 steps:
1. Predict other vehicles on the road.
2. Calculate trajectories for all possible options
3. Evaluate trajectories for collisions and reach
4. Possibly perform lane changing action and update final trajectory

### Predict other vehicles (lines 405 - 426)
The future trajectory for every vehicle from the sensor data is calculated to be able to detect possible collisions in the future and decide if lane changes are safe.
Therefore, an evaluation horizon is defined (here: 1000 steps of 50ms) and the cars position is linearly projected in Frenet space with their current speeds.
Specifically, the cars' lane is assumed not to change (constant d value) and the position along the road (s value) is linearly increased.

### Caculate possible trajectories (lines 429 - 467)
The vehicle has up to three options: stay in lane, change to the left lane or change to the right lane. For each option a trajectory is planned using waypoints and fitting a spline. The same number of future steps are calculated (here: 1000 steps) to get the position of the car for each of the subsequent time steps.

### Evaluate trajectories (lines 469 - 502)
After the trajectories of all possible options as well as the trajectories of all other vehicles have been calculated, it is now evaluated which action would lead the furthest without any collision.
To do this, the distance between the action trajectories and all other vehicles is calculated. If the distance in s and d coordinates (Frenet system) falls below a defined threshold, a collision is registered.
The result of this evaluation might look like this:  
* changing lane to the left: 34 steps without collision
* staying on the same lane: 411 steps without collision
* changing lane to the right: 1000 steps without collision

From this result, one might conclude that changing the lane to the right is best, because no collision can be detected during the time horizon which is not true for the other actions. Since the data and predictions can sometimes be a bit inaccurate, changing lanes is only performed when the corresponding trajectory seems to be better in several consecutive steps (here chosen to be 20).

It is also worth noting that even though trajectories are checked for collisions, these collisions would not occur most of the time, because these trajectories assume constant speed of the car and do not incorporate slowing down when coming to close to a vehicle. However, these calculations are a very good proxy to whether or not a path is safe and how far ahead no other vehicle will block the path.

### Perform action and update enacted trajectory (lines 528 - 593)
Finally, the decision from the evaluation step is carried out and the car's trajectories points are updated with the newest information. Additionally to managing the car's current state ('performing lane change', 'staying in line'), this part also manages speed changes in case all paths are currently blocked, or if the road ahead of the vehicle is free for the car to speed up again.
