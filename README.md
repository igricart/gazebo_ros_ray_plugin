# Gazebo ROS Ray plugin
Implements a ray sensor for both GPU or CPU processing.
Developed by kev-the-dev to solve the following issues [1](https://github.com/ros-simulation/gazebo_ros_pkgs/pull/778) and [2](https://github.com/ros-simulation/gazebo_ros_pkgs/issues/772)

# Depends on Gazebo 9.4 at least
[How to update gazebo 9 to 9.4](https://bitbucket.org/DataspeedInc/velodyne_simulator/src/506664dd478984aa6645d8210802a4a7ddc40629/gazebo_upgrade.md)

## Usage
That is supposed to be the core of a LiDAR simulation using either intel real sense L515 or Livox Horizon using GPU

After building, one paste the following snippet in your `intel_L515_sensor.gazebo.xacro`

```
<robot name="intel_L515" xmlns:xacro="http://www.ros.org/wiki/xacro">
    <!-- Common Paramenter -->
    <xacro:property name="M_PI" value="3.1415926535897931" />

    <!-- Macro -->
    <xacro:macro name="intel_L515" params="*origin parent:=base_link name:=intel_L515 topic:=intel_L515_lidar robot_namespace:=/belt_loader">

        <joint name="${name}_base_mount_joint" type="fixed">
            <xacro:insert_block name="origin" />
            <parent link="${parent}" />
            <child link="${name}_case" />
        </joint>

        <link name="${name}_case">
            <visual>
                <geometry>
                    <cylinder length="0.0045" radius="0.03" />
                </geometry>
            </visual>
            <inertial>
                <mass value="0.010" />
                <origin xyz="0 0 0" />
                <inertia ixx="1.2e-6" ixy="0" ixz="0" iyy="1.5e-6" iyz="0" izz="1.2e-6" />
            </inertial>
            <collision>
                <geometry>
                    <cylinder length="0.0045" radius="0.03" />
                </geometry>
            </collision>
        </link>

        <joint name="${name}_case_lidar" type="fixed">
            <origin xyz="-0.001 0 0" rpy="0 ${M_PI/2} 0" />
            <parent link="${name}_case" />
            <child link="${name}_lidar" />
        </joint>

        <link name="${name}_lidar">

            <inertial>
                <mass value="0.010" />
                <origin xyz="0 0 0" />
                <inertia ixx="1.2e-6" ixy="0" ixz="0" iyy="1.5e-6" iyz="0" izz="1.2e-6" />
            </inertial>
        </link>

        <gazebo reference="${name}_lidar">
            <sensor type="gpu_ray" name="intel_{name}">
                <pose>0 0 0 0 0 0</pose>
                <visualize>false</visualize>
                <update_rate>30</update_rate>
                <ray>
                    <scan display="true">
                        <horizontal>
                            <samples>1024</samples>
                            <resolution>1</resolution>
                            <min_angle>-${M_PI * 35/180}</min_angle>
                            <max_angle>${M_PI * 35/180}</max_angle>
                        </horizontal>
                        <vertical>
                            <samples>768</samples>
                            <resolution>1</resolution>
                            <min_angle>-${M_PI * 27.5/180}</min_angle>
                            <max_angle>${M_PI * 27.5/180}</max_angle>
                        </vertical>
                    </scan>
                    <range>
                        <min>0.25</min>
                        <max>9.0</max>
                    </range>
                </ray>
                <plugin filename="libgazebo_ros_ray_sensor.so" name="gazebo_intel_L515_lidar_range">
                    <robotNamespace>${robot_namespace}</robotNamespace>
                    <!-- TODO: Add actual gaussian noise -->
                    <!-- <gaussianNoise>0.005</gaussianNoise> -->
                    <noise>
                        <type>gaussian</type>
                        <!-- Noise parameters based on published spec for Hokuyo laser
                             achieving "+-30mm" accuracy at range < 10m.  A mean of 0.0m and
                             stddev of 0.01m will put 99.7% of samples within 0.03m of the true
                             reading. -->
                        <mean>0.0</mean>
                        <stddev>0.01</stddev>
                    </noise>
                    <alwaysOn>true</alwaysOn>
                    <updateRate>30</updateRate>
                    <topicName>${topic}</topicName>
                    <frameName>${name}_lidar</frameName>
                    <outputType>sensor_msgs/PointCloud2</outputType>
                </plugin>
            </sensor>
        </gazebo>
    </xacro:macro>
</robot>
```

And adding to your project
```
	<!-- Import Intel L515 -->
	<xacro:include filename="$(find your_package_description)/urdf/sensors/intel_L515_sensor.gazebo.xacro" />
	<xacro:intel_L515 parent="base_link" name="intel_L515" topic="/intel_L515_test">
		<origin xyz="0 0 1.3" rpy="0 -${M_PI/2} 0" />
	</xacro:intel_L515>
```

Pay attention to the folder structure in `your_package_description`