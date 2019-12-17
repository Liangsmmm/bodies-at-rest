# Bodies at Rest
## 3D Human Pose and Shape Estimation from a Pressure Image using Synthetic Data

<p align="center">
  <img width="98%" src="https://github.com/henryclever/bodies-at-rest/blob/master/docs/figures/intro_overview.JPG?raw=true" alt="None"/>
</p>

Paper:

Video: https://www.youtube.com/watch?v=UHBqw0BYWEw

Link to PressurePose dataset: XXXXXXX. Make a new folder on your desktop, `~/data_BR/`, and unzip it there.

<p align="center">
  <img width="110%" src="https://github.com/henryclever/bodies-at-rest/blob/master/docs/figures/git_break1.JPG?raw=true" alt="None"/>
</p>

## What code is in here?

This repository: 

* Allows you to visualize both synthetic and real data in the PressurePose dataset. This is to help you get started. The synthetic dataset includes 206,000 fully labeled pressure images, meaning that each pressure image has a corresponding SMPL human mesh parameterized by body shape (10 PCA parameters), pose (69 joint angles), posture (6 DOF global transform), gender, height, and weight. The real dataset includes 1051 pressure images with co-registered point cloud data, RGB data, gender, height, and weight. 
* Has the code for PressureNet. That gives you a jumping point if you are interested in considering other architectures. There are switches inside of the PressureNet code to modify how it is trained, e.g. to include height and weight during training or not. 

<p align="center">
  <img width="110%" src="https://github.com/henryclever/bodies-at-rest/blob/master/docs/figures/git_break2.JPG?raw=true" alt="None"/>
</p>

## PressurePose dataset visualization
For the synthetic data, when you run the following code, you will see something like the pictures below. There are flags in the code that allow you to segment based on the limbs and also to cut out mesh vertices that aren't facing the camera. The camera is positioned in the synthetic dataset at the same location as the real one, so cutting out the non-camera facing vertices will allow you to better compare the synthetic data to the real point cloud data.




For the real data, when you run the following code `python viz_real_cvpr_release.py` you will see two pop up boxes: one for 2D data that includes RGB, depth, and pressure; the other for a 3D point cloud and pressure image rendering in pyrender that you can flip around to inspect. Make sure you include flags to specify the participant number and the type of real pose dataset. For example, you might use `--p_idx 1 --pose_type 'prescribed'` to specify the first participant in the list and the set of 48 prescribed poses. You can use numbers 1 through 20 to specify the participant, because there are 20, and pose types of `'prescribed'` and `'p_select'`, with the latter used to specify participant selected poses. Here is what you should see when you run this code:


<p align="center">
  <img width="50%" src="https://github.com/henryclever/bodies-at-rest/blob/master/docs/figures/viz_real_2D.png?raw=true" alt="None"/>
  <img width="20%" src="https://github.com/henryclever/bodies-at-rest/blob/master/docs/figures/viz_real_3D_1.png?raw=true" alt="None"/>
  <img width="15%" src="https://github.com/henryclever/bodies-at-rest/blob/master/docs/figures/viz_real_3D_2.png?raw=true" alt="None"/>
  <img width="15%" src="https://github.com/henryclever/bodies-at-rest/blob/master/docs/figures/viz_real_3D_3.png?raw=true" alt="None"/>
</p>


<p align="center">
  <img width="110%" src="https://github.com/henryclever/bodies-at-rest/blob/master/docs/figures/git_break3.JPG?raw=true" alt="None"/>
</p>

## PressureNet training
There are 3 steps to train PressureNet as implemented in the paper.
* Step 1: Train network 1 for 100 epochs using loss function 1. Run the following: `python train_pressurenet.py --net 1`. You can also use the flags `--htwt`, `--calnoise`, and `--small` to include height/weight data in the network, calibration noise in the network, and train on a smaller dataset size (1/4th). It's important to visualize things to make sure your network is training OK. So if you use the `--viz` flag a set of pressure maps pops up with joint markers projected into 2D - there are 24 of them. Green - ground truth, yellow - estimated. The smaller top right images show the input channels, with the exception of height and weight. Note from the pressure image that this body is in a lateral posture, but it has just started training, so the yellow estimated joint positions are far from the ground truth.

<p align="left">
  <img width="50%" src="https://github.com/henryclever/bodies-at-rest/blob/master/docs/figures/net1.png?raw=true" alt="None"/>
</p>


* Step 2: Compute a new dataset that has spatial map reconstructions from the PMR output of network 1. Run the following: `python compute_network1_spatialmaps.py`. Make sure the flags on this match the flags you trained network 1 on.
* Step 3: Train network 2 for 100 epochs using loss function 2. Run the following: `python train_pressurenet.py --net 2 --pmr`. Make sure the flags on this match the flags you trained network 1 on (except `--viz`, that doesn't matter). If you do visualize, expect a box like the one below to pop up. For this example, while the ground truth is in a lateral posture, the network 1 estimate outputs a pose in a prone posture. The smaller top right images show the input channels. The bottom right channels show the output reconstructed spatial maps, as well as ground truth on the far right. Here, net 2 has just started training so the output Q_2 doesn't differ substantially from the input Q_1. 

<p align="left">
  <img width="80%" src="https://github.com/henryclever/bodies-at-rest/blob/master/docs/figures/net2_pmr.png?raw=true" alt="None"/>
</p>

The data can take a long time to load. Use the `--qt` flag to run a quick test on a small portion of the dataset to check for bugs. You can use an euler angle parameterization instead of the direction cosines in the SMPL model. Use the `--losstype 'anglesEU'` flag for that. You'll have to change some file directories so that the `train_pressurenet.py` knows where to find your data.


<p align="center">
  <img width="110%" src="https://github.com/henryclever/bodies-at-rest/blob/master/docs/figures/git_break4.JPG?raw=true" alt="None"/>
</p>

## What other packages do I need?
* SMPL: A Skinned Multi-Person Linear Model - https://smpl.is.tue.mpg.de/
* PyRender - https://github.com/mmatl/pyrender
* Trimesh - https://github.com/mikedh/trimesh
* PyTorch - https://pytorch.org/
* PyTorch HMR - https://github.com/MandyMo/pytorch_HMR
* LateX packages - `sudo apt-get install dvipng texlive-fonts-recommended texlive-fonts-extra`
* Matplotlib, PyGlet, some others .... 


<p align="center">
  <img width="110%" src="https://github.com/henryclever/bodies-at-rest/blob/master/docs/figures/git_break5.JPG?raw=true" alt="None"/>
</p>

## Computer requirements
To train all 184K images, you'll need at least 64GB of ram on your CPU with the present implementation. You can run smaller sizes (e.g. 32K images) on a machine much smaller. If you restructured some code or converted some images to a more efficient format (e.g. float to int) it might help. You'll also need at least 8GB of ram on your GPU to fit the present implementation of PressureNet. I'm sure there are other ways to make this code more efficient, so if you can do it, make a request for me and push it back up to this repository with a tagged python filename - just make sure you have documented the changes with good comments. 


<p align="center">
  <img width="110%" src="https://github.com/henryclever/bodies-at-rest/blob/master/docs/figures/git_break6.JPG?raw=true" alt="None"/>
</p>

## What code isn't here? 
The code for generating more synthetic data isn't here. I've got it spread across multiple repositories, so it would be challenging to make it decipherable. I'd also worry that users would have trouble getting it up and running because of its complexity; what I have now needs work to make it less challenging to install. 


<p align="center">
  <img width="110%" src="https://github.com/henryclever/bodies-at-rest/blob/master/docs/figures/git_break7.JPG?raw=true" alt="None"/>
</p>
