# drone_src
ROS2 source code for drone workspace

---

## Steps to Connect Cube Flight Controller with ROS 2 Humble

For ROS 2, the most common way to connect a Cube flight controller (running ArduPilot) is through **MAVROS** (ROS 1) or **MAVROS2** (ROS 2 version). MAVROS is the bridge between ROS and the MAVLink protocol that ArduPilot uses.

### 1. Plug in the Cube

* Use USB from the carrier board → Ubuntu laptop.
* Run:

  ```bash
  ls /dev/ttyACM*
  ```

  You should see something like `/dev/ttyACM0`.
  That’s the Cube’s serial port.

---

### 2. Install MAVROS for ROS 2 Humble

MAVROS has a ROS 2 port, usually packaged as `ros-humble-mavros` and `ros-humble-mavros-extras`.

```bash
sudo apt update
sudo apt install ros-humble-mavros ros-humble-mavros-extras
```

---

### 3. Install GeographicLib Datasets

MAVROS needs GeographicLib for GPS and geodesy support:

```bash
wget https://raw.githubusercontent.com/mavlink/mavros/master/mavros/scripts/install_geographiclib_datasets.sh
chmod +x install_geographiclib_datasets.sh
sudo ./install_geographiclib_datasets.sh
```

---

### 4. `Permission denied` on `/dev/ttyACM0`

Add your user to the `dialout` group (serial device group):

```bash
sudo usermod -aG dialout $USER
```

Then **log out and log back in** (or reboot) so the group membership takes effect.
After that, check permissions:

```bash
ls -l /dev/ttyACM0
```

You should see something like:

```
crw-rw---- 1 root dialout 166, 0 Sep  3 19:30 /dev/ttyACM0
```

And your user will now belong to `dialout`.


### 5. Run MAVROS Node

Run the **`mavros_node`** with parameters on the command line to see if your CubeBlue is talking to ROS 2.

```bash
ros2 run mavros mavros_node --ros-args -p fcu_url:=serial:///dev/ttyACM0:115200
```

* `ros2 run mavros mavros_node` → runs the MAVROS bridge.
* `--ros-args -p fcu_url:=...` → passes the serial port parameter.
* Replace `/dev/ttyACM0` with the actual port if it’s different (`ls /dev/ttyACM*` to check).

---

### 6. Check if It Works

Open another terminal and run:

```bash
ros2 topic list
```

You should see MAVROS topics like:

```
/mavros/state
/mavros/imu/data
/mavros/battery
/mavros/local_position/pose
```

Then try:

```bash
ros2 topic echo /mavros/state
```

You should get MAVLink heartbeat info, like:

```
header:
  stamp:
    sec: 123
    nanosec: 456
connected: True
armed: False
guided: False
mode: "STABILIZE"
system_status: 3
```

---

If this works, then we know the Cube, MAVROS2, and ROS 2 Humble connection is solid. At this point, ROS 2 nodes can subscribe to telemetry and publish commands to the Cube.

