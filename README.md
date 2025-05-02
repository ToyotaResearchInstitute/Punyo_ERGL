# Punyo RL
A package for exploring reinforcement learning algorithms for Punyo using Isaac Gym.
This is a derivation from [Isaac Gym Benchmark Environemnts](https://github.com/NVIDIA-Omniverse/IsaacGymEnvs) and includes code, environemntes, and models relevant to Punyo.

## Set up

1. Install Anaconda by following the [instructions](https://docs.anaconda.com/free/anaconda/install/linux/).

   Tip: If you don’t want anaconda to modify your shell script, then choose “no” for step 8 (which is the default choice).
   If you do so, then in the future, when you want to use anaconda, you will need to:
   source path_to_anaconda/bin/activate (activate conda)
   conda activate your_environment_name

   For more conda related command, check out this [CONDA CHEAT SHEET](https://docs.conda.io/projects/conda/en/4.6.0/_downloads/52a95608c49671267e40c689e0bc00ca/conda-cheatsheet.pdf).

   Add channels to conda:
   ```
   conda config --add channels conda-forge
   ```

2. Fork this repo in Github and clone it.

   ```
   mkdir punyo_rl_isaac
   cd punyo_rl_isaac
   git clone {your_fork_of_punyo_rl}
   ```

3. Install IsaacGym

   Download [IsaacGym](https://developer.nvidia.com/isaac-gym) and extract the file in ```punyo_rl_isaac``` via ```tar -xvzf ~/Downloads/IsaacGym_Preview_4_Package.tar.gz```

   Run the following commands:

   ```
   conda env create -f rlgpu.yml
   conda activate rlgpu
   cd isaacgym/python
   pip install -e .
   export LD_LIBRARY_PATH=~/anaconda3/envs/rlgpu/lib:$LD_LIBRARY_PATH
   export ISAAC_GYM_PATH=~/IsaacGymEnvs/isaacgym 
   ```

   Try to run an example:

   ```
   cd python/examples
   python joint_monkey.py  # You should see a bunch of humanoids.
   ```

3. Install IsaacGymEnvs

   ```
   cd punyo_rl
   pip install -e .
   ```

   Try the example:

   ```
   cd isaacgymenvs
   python train.py task=PunyoV2AMP
   ```

## Run the code

To train a policy,

1. Set up the robot and object for your environment, e.g., in PunyoV2AMP.yaml:
   ```
   asset:
     assetFilePunyo: "urdf/punyo/punyo_v2.urdf"

   task:
   task_name:
      assetFileBox: "urdf/objects/jug_5_gallon_onshape.urdf"
   ```

2. Set up the demonstration file for your training, e.g., in PunyoV2AMP.yaml:
   ```
   asset:
     motion_file_path: 'data/paper/task_over_shoulder_lift_jug/teleop_eigen/original/'
   ```
The program will collect all the .pkl files in the specified directory to form the motion library. There can be only one or multiple .pkl files in the folder.

3. Set up the moving range for the manipuland's initial position, for example:
   ```
   object:
      xRange: 0.05
      yRange: 0.05
      yawRange: 0.05
   ```
The purpose of these parameters is for domain randomization.

4. Set up the target pose for the manipuland, for example:
   ```
   task:
      target_x: 0.13
      target_y: 0.3
      target_z: 0.64
      target_roll: 0.0
      target_pitch: -1.5708
      target_yaw: 0.0
   ```

5. Set up the observation for the discriminator and the policy, for example:
   ```
   env:
      ampObservation: [robot_dof]
      policyObservation: [robot_dof, box_pose, previous_actions]
      criticObservation: [robot_dof, box_pose, previous_actions]
   ```
   All the possible options are robot_dof (14), robot_vel (14), box_pose (7 position+quaternion), ee_pose (7\*2 left p+q, right p+q), ee_binary_contact (2 left, right), floatie_binary_contact (7\*2 left shoulder to hand, right shoulder to hand).

6. Set up the initialization for the robot and the box, for example:
   ```
   env:
     stateInit: "Default"
   ```
   All the possible options are:
   a. Default: Set the robot to the default state (if any), and set the box to the default state (if any) plus specified disturbance.
   b. Start: Set the robot and the box to the start state of the demonstration.
   c. Random: Set the robot and the box to a random state of the demonstration.
   d. Hybrid: A combination of Default and Random.

7. Start your training:
   ```
   python train.py task=PunyoV2AMP wandb_activate=True wandb_project=YOUR_PROJECT_NAME wandb_logcode_dir=ABSOLUTE_PUNYO_RL_PATH
   # The visualization can be toggled with a "v" key press.

   # The GPU to be used can be specified by adding the flags sim_device and rl_device
   # (e.g. sim_device=cuda:1 rl_device=cuda:1).
   ```
8. Test your policy:
   ```
   python play_policy.py --checkpoint_file data/task_over_shoulder_lift_jug.pth
   ```
