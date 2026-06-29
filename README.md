# Sensorless Transparent Wam Teleop

This repository contains the implementation accompanying the IROS 2026 paper:

> **Sensorless Four-Channel Control Architecture Using Inverse Dynamics Modeling for Human-Scale Bilateral Teleoperation**  
> **Amir Noohian**, Dylan Miller, Justin Valentine, Alan Lynch, and Martin Jagersand  
> *IEEE/RSJ International Conference on Intelligent Robots and Systems (IROS), 2026.*

The repository implements a real-time bilateral teleoperation framework for Barrett WAM robots. It supports teleoperation between a 4-DOF leader equipped with a haptic wrist and a 7-DOF follower using UDP-based communication. The framework includes both conventional and model-based teleoperation controllers, including inverse dynamics compensation and sensorless force estimation. ROS is used only for robot state publishing and data collection and is not part of the real-time control loop.

---

## Dynamic Parameter Identification

The model-based teleoperation controllers require identified dynamic parameters for the leader and follower WAM robots.

Dynamic parameter identification should be performed using the companion repository [here](https://github.com/amir-noohian/feasible-dynamics-estimation).

After estimating the parameters, update the corresponding parameter files used by this package before running the model-based teleoperation controllers.

---

## Build Instructions

Place this package in `<catkin_ws>/src/`.

By default, both the leader and follower executables are built.

To build only the follower:

```bash
catkin_make --cmake-args -DBUILD_LEADER=OFF
```

or set

```cmake
option(BUILD_LEADER "Build leader executable" OFF)
```

in `CMakeLists.txt`.

To build the leader, the `haptic_wrist` library is required. Installation instructions are available [here](https://github.com/dmiller12/libhaptic_wrist).

---

## Run Instructions

The `config/` directory contains the Barrett configuration files for the leader and follower.

Configure the appropriate Barrett configuration by running

```bash
source scripts/setup_leader.sh
```

or

```bash
source scripts/setup_follower.sh
```

These scripts set the `BARRETT_CONFIG_FILE` environment variable for the current terminal session.

Depending on your CAN interface, you may also need to modify the bus port in:

- `config/leader.conf`
- `config/follower.conf`

Each node supports the following command-line arguments:

```bash
rosrun wam_teleop leader [remoteHost] [recPort] [sendPort]
rosrun wam_teleop follower [remoteHost] [recPort] [sendPort]
```

Use `-h` or `--help` to see all available options.

---

## Example

Source your workspace:

```bash
source devel/setup.bash
```

Initialize the CAN interface.

For PCI CAN:

```bash
source scripts/pci_can_init.sh
```

For USB CAN:

```bash
source scripts/usb_can_init.sh
```

> **Note:** You may need to adjust the CAN interface number depending on the order in which the devices were connected.

Start the ROS master:

```bash
roscore
```

Start the leader:

```bash
source scripts/setup_leader.sh
rosrun wam_teleop leader 127.0.0.1 5555 5554
```

Start the follower:

```bash
source scripts/setup_follower.sh
rosrun wam_teleop follower 127.0.0.1 5554 5555
```

The receive and send ports must match between the leader and follower:

| Robot | Receive Port | Send Port |
|--------|--------------|-----------|
| Leader | 5555 | 5554 |
| Follower | 5554 | 5555 |

Once both nodes have started:

1. Press `l` on the leader to move to the synchronization position.
2. Press `l` on the follower to move to the synchronization position.
3. Wait until both robots reach the synchronization position.
4. Press **Enter** on the leader to establish the connection.
5. Press **Enter** on the follower.

The system is now ready for bilateral teleoperation.

---

## Shutdown

To ensure proper thread and socket cleanup:

1. Return both WAMs to the home position.
2. On the leader, press `x` to exit the control loop.
3. Shift-idle the leader.
4. On the follower, press `x` to exit the control loop.
5. Shift-idle the follower.

---

## Citation

If you use this repository in your research, please cite:

```bibtex
@inproceedings{noohian2026sensorless,
  title     = {Sensorless Four-Channel Control Architecture Using Inverse Dynamics Modeling for Human-Scale Bilateral Teleoperation},
  author    = {Noohian, Amir and Miller, Dylan and Valentine, Justin and Lynch, Alan and Jagersand, Martin},
  booktitle = {Proceedings of the IEEE/RSJ International Conference on Intelligent Robots and Systems (IROS)},
  year      = {2026}
}
```
