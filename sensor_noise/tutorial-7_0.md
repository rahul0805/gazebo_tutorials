# Introduction
Gazebo provides models of many common sensors.  In the real world, sensors exhibit noise, in that they do not observe the world perfectly.  By default, Gazebo's sensors will observe the world perfectly (though not the IMU; read more below).  To present a more realistic environment in which to try out perception code, we need to explicitly add noise to the data generated by Gazebo's sensors.

At the time of writing, Gazebo can add noise to the following types of sensors:

* Ray (e.g., lasers)
* Camera
* IMU

# Ray (laser) noise

For ray sensors, we add Gaussian noise to the range of each beam.  You can set the mean and the standard deviation of the Gaussian distribution from which noise values will be sampled.  A noise value is sampled independently for each beam.  After adding noise, the resulting range is clamped to lie between the sensor's minimum and maximum ranges (inclusive).

To test the ray noise model:

1. Create a model directory:

        mkdir -p ~/.gazebo/models/noisy_laser


1.  Create a model config file:

        gedit ~/.gazebo/models/noisy_laser/model.config

1.  Paste in the following contents:

    ~~~
    <?xml version="1.0"?>
    <model>
      <name>Noisy laser</name>
      <version>1.0</version>
      <sdf version='1.6'>model.sdf</sdf>

      <author>
       <name>My Name</name>
       <email>me@my.email</email>
      </author>

      <description>
        My noisy laser.
      </description>
    </model>
    ~~~

1.  Create a `~/.gazebo/models/noisy_laser/model.sdf` file.

    ~~~
    gedit ~/.gazebo/models/noisy_laser/model.sdf
    ~~~

1. Paste in the following, which is a copy of the standard Hokuyo model with the addition of noise:

    ~~~
    <?xml version="1.0" ?>
    <sdf version="1.6">
      <model name="hokuyo">
        <link name="link">
          <gravity>false</gravity>
          <inertial>
            <mass>0.1</mass>
          </inertial>
          <visual name="visual">
            <geometry>
              <mesh>
                <uri>model://hokuyo/meshes/hokuyo.dae</uri>
              </mesh>
            </geometry>
          </visual>
          <sensor name="laser" type="ray">
            <pose>0.01 0 0.03 0 -0 0</pose>
            <ray>
              <scan>
                <horizontal>
                  <samples>640</samples>
                  <resolution>1</resolution>
                  <min_angle>-2.26889</min_angle>
                  <max_angle>2.268899</max_angle>
                </horizontal>
              </scan>
              <range>
                <min>0.08</min>
                <max>10</max>
                <resolution>0.01</resolution>
              </range>
              <noise>
                <type>gaussian</type>
                <mean>0.0</mean>
                <stddev>0.01</stddev>
              </noise>
            </ray>
            <plugin name="laser" filename="libRayPlugin.so" />
            <always_on>1</always_on>
            <update_rate>30</update_rate>
            <visualize>true</visualize>
          </sensor>
        </link>
      </model>
    </sdf>
    ~~~

1. Start Gazebo:

        gazebo

1. Insert a noisy laser: in the left pane, select the `Insert` tab, then click on `Noisy laser`.  Drop your laser somewhere in the world and place a box in front of it.

    [[file:files/Noisy_laser_inserted.png|640px]]

1. Visualize the noisy laser: click on Window->Topic Visualization (or press Ctrl-T) to bring up the Topic Selector.

    [[file:files/Topic_visualizer.png|640px]]

1. Find the the topic with a name like `/gazebo/default/hokuyo/link/laser/scan` and click on it, then click `Okay`.  You'll get get a Laser View window that shows you the laser data.

    [[file:files/Noisy_laser_visualizer.png|640px]]

As you can see, the scan is noisy.  To adjust the noise, simply play with the mean and standard deviation values in the `model.sdf`, where the units are meters:

~~~
<noise>
  <type>gaussian</type>
  <mean>0.0</mean>
  <stddev>0.01</stddev>
</noise>
~~~

These are reasonable values for Hokuyo lasers.

# Camera noise
For camera sensors, we model [output amplifier noise](http://en.wikipedia.org/wiki/Image_noise#Amplifier_noise_.28Gaussian_noise.29), which adds a Gaussian-sampled disturbance independently to each pixel.  You can set the mean and the standard deviation of the Gaussian distribution from which noise values will be sampled.  A noise value is sampled independently for each pixel, then that noise value is added independently to each color channel for that pixel.  After adding noise, the resulting color channel value is clamped to lie between 0.0 and 1.0; this floating point color value will end up as an unsigned integer in the image, usually between 0 and 255 (using 8 bits per channel).

This noise model is implemented in a [GLSL](http://www.opengl.org/documentation/glsl/) shader and requires a GPU to run.

To test the camera noise model:

1. Create a model directory:

        mkdir -p ~/.gazebo/models/noisy_camera

1.  Create a model config file:

        gedit ~/.gazebo/models/noisy_camera/model.config

1.  Paste in the following contents:

    ~~~
    <?xml version="1.0"?>
    <model>
      <name>Noisy camera</name>
      <version>1.0</version>
      <sdf version='1.6'>model.sdf</sdf>

      <author>
       <name>My Name</name>
       <email>me@my.email</email>
      </author>

      <description>
        My noisy camera.
      </description>
    </model>
    ~~~

1.  Create a `~/.gazebo/models/noisy_camera/model.sdf` file.

    ~~~
    gedit ~/.gazebo/models/noisy_camera/model.sdf
    ~~~

1. Paste in the following, which is a copy of the standard camera model with the addition of noise:

    ~~~
    <?xml version="1.0" ?>
    <sdf version="1.6">
      <model name="camera">
        <link name="link">
          <gravity>false</gravity>
          <pose>0.05 0.05 0.05 0 0 0</pose>
          <inertial>
            <mass>0.1</mass>
          </inertial>
          <visual name="visual">
            <geometry>
              <box>
                <size>0.1 0.1 0.1</size>
              </box>
            </geometry>
          </visual>
          <sensor name="camera" type="camera">
            <camera>
              <horizontal_fov>1.047</horizontal_fov>
              <image>
                <width>1024</width>
                <height>1024</height>
              </image>
              <clip>
                <near>0.1</near>
                <far>100</far>
              </clip>
              <noise>
                <type>gaussian</type>
                <mean>0.0</mean>
                <stddev>0.07</stddev>
              </noise>
            </camera>
            <always_on>1</always_on>
            <update_rate>30</update_rate>
            <visualize>true</visualize>
          </sensor>
        </link>
      </model>
    </sdf>
    ~~~

1. Start Gazebo:

        gazebo

1. Insert a noisy camera: in the left pane, select the `Insert` tab, then click on `Noisy camera`.  Drop your camera somewhere in the world.

1. Visualize the noisy camera: click on Window->Topic Visualization (or press Ctrl-T) to bring up the Topic Selector.

1. Find the the topic with a name like `/gazebo/default/camera/link/camera/image` and click on it, then click `Okay`.  You'll get get a Image View window that shows you the image data.

    [[file:files/Noisy_camera_visualizer.png|640px]]

If you look closely, you can see that the image is noisy.  To adjust the noise, simply play with the mean and standard deviation values in the `model.sdf`.  These are unitless values; the noise will be added to each color channel within the range [0.0,1.0].

The example above has a very high `<stddev>`. Try reducing this value:

~~~
<noise>
  <type>gaussian</type>
  <mean>0.0</mean>
  <stddev>0.007</stddev>
</noise>
~~~

These are reasonable values for decent digital cameras.

# IMU noise
For IMU sensors, we model two kinds of disturbance to angular rates and linear accelerations: noise and bias.  Angular rates and linear accelerations are considered separately, leading to 4 sets of parameters for this model: rate noise, rate bias, accel noise, and accel bias.  No noise is applied to the IMU's orientation data, which is extracted as a perfect value in the world frame (this should change in the future).

Noise is additive, sampled from a Gaussian distribution.  You can set the mean and the standard deviation of the Gaussian distributions (one for rates and one for accels) from which noise values will be sampled.   A noise value is sampled independently for each component (X,Y,Z) of each sample, and added to that component.

Bias is also additive, but it is sampled once, at the start of simulation. You can set the mean and the standard deviation of the Gaussian distributions (one for rates and one for accels) from which bias values will be sampled.   Bias will be sampled according to the provided parameters, then with equal probability negated; the assumption is that the provided mean indicates the magnitude of the bias and that it's equal likely to be biased in either direction.  Thereafter, bias is a fixed value, added to each component (X,Y,Z) of each sample.

**Note:** depending on the system being simulated and the configuration of the physics engine, it can happen that the simulated IMU data is already quite noisy because the system is not being solved all the way to convergence.  So it may not be necessary to add noise, depending on your application.

To test the IMU noise model:

1. Create a model directory:

        mkdir -p ~/.gazebo/models/noisy_imu

1.  Create a model config file:

        gedit ~/.gazebo/models/noisy_imu/model.config

1.  Paste in the following contents:

    ~~~
    <?xml version="1.0"?>
    <model>
      <name>Noisy IMU</name>
      <version>1.0</version>
      <sdf version='1.6'>model.sdf</sdf>

      <author>
       <name>My Name</name>
       <email>me@my.email</email>
      </author>

      <description>
        My noisy IMU.
      </description>
    </model>
    ~~~

1.  Create a `~/.gazebo/models/noisy_imu/model.sdf` file.

        gedit ~/.gazebo/models/noisy_imu/model.sdf

1. Paste in the following:

    ~~~
    <?xml version="1.0" ?>
    <sdf version="1.6">
      <model name="imu">
        <link name="link">
          <inertial>
            <mass>0.1</mass>
          </inertial>
          <visual name="visual">
            <geometry>
              <box>
                <size>0.1 0.1 0.1</size>
              </box>
            </geometry>
          </visual>
          <collision name="collision">
            <geometry>
              <box>
                <size>0.1 0.1 0.1</size>
              </box>
            </geometry>
          </collision>
          <sensor name="imu" type="imu">
            <imu>
              <angular_velocity>
                <x>
                  <noise type="gaussian">
                    <mean>0.0</mean>
                    <stddev>2e-4</stddev>
                    <bias_mean>0.0000075</bias_mean>
                    <bias_stddev>0.0000008</bias_stddev>
                  </noise>
                </x>
                <y>
                  <noise type="gaussian">
                    <mean>0.0</mean>
                    <stddev>2e-4</stddev>
                    <bias_mean>0.0000075</bias_mean>
                    <bias_stddev>0.0000008</bias_stddev>
                  </noise>
                </y>
                <z>
                  <noise type="gaussian">
                    <mean>0.0</mean>
                    <stddev>2e-4</stddev>
                    <bias_mean>0.0000075</bias_mean>
                    <bias_stddev>0.0000008</bias_stddev>
                  </noise>
                </z>
              </angular_velocity>
              <linear_acceleration>
                <x>
                  <noise type="gaussian">
                    <mean>0.0</mean>
                    <stddev>1.7e-2</stddev>
                    <bias_mean>0.1</bias_mean>
                    <bias_stddev>0.001</bias_stddev>
                  </noise>
                </x>
                <y>
                  <noise type="gaussian">
                    <mean>0.0</mean>
                    <stddev>1.7e-2</stddev>
                    <bias_mean>0.1</bias_mean>
                    <bias_stddev>0.001</bias_stddev>
                  </noise>
                </y>
                <z>
                  <noise type="gaussian">
                    <mean>0.0</mean>
                    <stddev>1.7e-2</stddev>
                    <bias_mean>0.1</bias_mean>
                    <bias_stddev>0.001</bias_stddev>
                  </noise>
                </z>
              </linear_acceleration>
            </imu>
            <always_on>1</always_on>
            <update_rate>1000</update_rate>
          </sensor>
        </link>
      </model>
    </sdf>
    ~~~

1. Start Gazebo:

        gazebo

1. Insert a noisy IMU: in the left pane, select the `Insert` tab, then click on `Noisy IMU`.  Drop your IMU somewhere in the world.

1. Visualize the noisy IMU: click on Window->Topic Visualization (or press Ctrl-T) to bring up the Topic Selector.

1. Find the the topic with a name like `/gazebo/default/imu/link/imu/imu` and click on it, then click `Okay`.  You'll get get a Text View window that shows you the IMU data.

It can be difficult to appreciate noise on a high-rate sensor like an IMU, especially in a complex system.  You should be able to see the effect of large non-zero means in the noise and/or bias parameters.

To adjust the noise, simply play with the mean and standard deviation values in the `model.sdf`.  Units for rate noise and rate bias are rad/s, for accel noise and accel bias are m/s^2.

~~~
<angular_velocity>
  <x>
    <noise type="gaussian">
      <mean>0.0</mean>
      <stddev>2e-4</stddev>
      <bias_mean>0.0000075</bias_mean>
      <bias_stddev>0.0000008</bias_stddev>
    </noise>
  </x>
  <y>
    <noise type="gaussian">
      <mean>0.0</mean>
      <stddev>2e-4</stddev>
      <bias_mean>0.0000075</bias_mean>
      <bias_stddev>0.0000008</bias_stddev>
    </noise>
  </y>
  <z>
    <noise type="gaussian">
      <mean>0.0</mean>
      <stddev>2e-4</stddev>
      <bias_mean>0.0000075</bias_mean>
      <bias_stddev>0.0000008</bias_stddev>
    </noise>
  </z>
</angular_velocity>
<linear_acceleration>
  <x>
    <noise type="gaussian">
      <mean>0.0</mean>
      <stddev>1.7e-2</stddev>
      <bias_mean>0.1</bias_mean>
      <bias_stddev>0.001</bias_stddev>
    </noise>
  </x>
  <y>
    <noise type="gaussian">
      <mean>0.0</mean>
      <stddev>1.7e-2</stddev>
      <bias_mean>0.1</bias_mean>
      <bias_stddev>0.001</bias_stddev>
    </noise>
  </y>
  <z>
    <noise type="gaussian">
      <mean>0.0</mean>
      <stddev>1.7e-2</stddev>
      <bias_mean>0.1</bias_mean>
      <bias_stddev>0.001</bias_stddev>
    </noise>
  </z>
</linear_acceleration>
~~~

These are reasonable values for a high-quality IMU.
