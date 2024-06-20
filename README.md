# Counter

To start your Phoenix server:

  * Install dependencies with `mix deps.get`
  * Start Phoenix endpoint with `mix phx.server` or inside IEx with `iex -S mix phx.server`

Now you can visit [`localhost:4000`](http://localhost:4000) from your browser.

Ready to run in production? Please [check our deployment guides](https://hexdocs.pm/phoenix/deployment.html).

## Learn more

  * Official website: https://www.phoenixframework.org/
  * Guides: https://hexdocs.pm/phoenix/overview.html
  * Docs: https://hexdocs.pm/phoenix
  * Forum: https://elixirforum.com/c/phoenix-forum
  * Source: https://github.com/phoenixframework/phoenix


RF 
import math

def reward_function(params):
    # Read input parameters
    track_width = params['track_width']
    distance_from_center = params['distance_from_center']
    all_wheels_on_track = params['all_wheels_on_track']
    speed = params['speed']
    steering = abs(params['steering_angle'])
    waypoints = params['waypoints']
    closest_waypoints = params['closest_waypoints']
    heading = params['heading']

    # Initialize reward
    reward = 1.0

    # Penalize going off-track
    if not all_wheels_on_track:
        return 1e-3

    # Calculate markers for distance from the center
    marker_1 = 0.1 * track_width
    marker_2 = 0.25 * track_width
    marker_3 = 0.5 * track_width

    # Reward for staying close to the center line
    if distance_from_center <= marker_1:
        reward += 1.0
    elif distance_from_center <= marker_2:
        reward += 0.5
    elif distance_from_center <= marker_3:
        reward += 0.1
    else:
        reward = 1e-3

    # Determine if the next waypoint is in a straight or turn section
    next_waypoint = waypoints[closest_waypoints[1]]
    prev_waypoint = waypoints[closest_waypoints[0]]

    track_direction = math.atan2(next_waypoint[1] - prev_waypoint[1], next_waypoint[0] - prev_waypoint[0])
    track_direction = math.degrees(track_direction)
    direction_diff = abs(track_direction - heading)

    # Adjust direction difference to range [0, 180]
    if direction_diff > 180:
        direction_diff = 360 - direction_diff

    # Encourage higher speed on straight sections and appropriate speed in turns
    MAX_SPEED = 4.0
    if direction_diff < 10.0:  # Straight section
        reward += speed / MAX_SPEED
    else:  # Turn section
        # Encourage slower speed for smooth turning
        if speed > (MAX_SPEED / 2):
            reward *= 0.5
        else:
            reward += speed / (MAX_SPEED / 2)

    # Apply a penalty for excessive steering
    if steering > 15.0:
        reward *= 0.8

    return float(reward)
