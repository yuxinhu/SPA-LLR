# SPA-LLR
This is the implementation of our paper [spatial‐angular locally low‐rank regularization (SPA‐LLR)](https://onlinelibrary.wiley.com/doi/full/10.1002/mrm.28025) which joint reconstructs multiple directions of diffusion-weighted data.

### Introduction

In SPA-LLR, we use a non-linear model to jointly reconstruct multi-direction multi-shot diffusion-weighted data. In the non-linear model, magnitude and phase images are separately reconstructed, so that we could add regularization terms on the magnitude images. We use spatial-angular locally low-rank regularization to utilize the relationships between different diffusion-encoding directions. Please refer to our paper for detailed explanations. 

To solve the non-linear model, alternating minimization is used. Thank [Dr. Frank Ong](https://profiles.stanford.edu/210728) for sharing their code on the [phase cycyling work](https://onlinelibrary.wiley.com/doi/full/10.1002/mrm.27011). In each iteration, we apply a gradient descent step for updates of phase map of each shot and each direction; while for magnitude updates, we apply a gradient update and a proximal operator update. 

### Usage

The example data include a multi-direction multi-shot multi-channel k-space data, correpsonding sensitivity maps (the same for different directions here), the SENSE reconstruction results for initialization (we provide here for simplicit of the code, it is simply based on SENSE. Please check our paper for details). 

Starting from the raw data, there are several steps as below.

Step 1: Extract k-space data (demo_readP.m). Here we provide a modifed Orchestra function(in Matlab, SDK 1.7-1) which load k-space data and perform ramp sampling and EPI phase correction with the Pfile, vrgf and ref files as input. The main function named "EpiDiffusionRecon_yuxin" is based on the original function "EpiDiffusionRecon" in Orchestra. The strategy is pretty similar what we did for modifying Orchestra for online shot-LLR reconstruction: add code to save k-space data after some post-processing step, except this is in Matlab. Notice that we are saving each direction data as one individual file (k1.mat, k2.mat, k3.mat, ...). For different scanner version, it may start from k1.mat or k2.mat. You probably need to change the name to make sure the following reconstruction load the data approporiately. 

Step 2: SENSE initialization (demo_SENSEini.m). In this step, we are going to (1) estimate the sensitivity map from the non-diffusion-weighted data, (2) reconstruct the non-diffusion-weighted data (Homodyne for partial Fourier reconstruction), (3) apply SENSE reconstruction for each individual shot, direction, and slice as the initialization for the following step. Here, there is going to be one file for each direction and slice (tr1.mat, tr2.mat, ...). Each file contains the k-space data of some direction and slice, corresponding sensitivity map, and the header information. I choose this way because (1) it is easy to debug, and (2) each file can be used for shot-LLR reconstruction or deep learning reconstruction, though some disk space may be wasted since we may save sensitivity maps (not that big) multiple times. Another thing to be noticed is that I usually acquire one non-diffusion-weighted data for every fifteen directions as suggested by [Dr. Qiyuan Tian](https://www.nmr.mgh.harvard.edu/user/4287093) for later registration between different directions. So, for sensitivity map calculation for each direction, we have the choice to use the neighboor non-diffusion-weighted data instead of the one at the beginning of the acquisition. This is supposed to be helpful especially when the motion is severe. But I have not done a careful comparison.

Step 3: SPA-LLR reconstruction (demo_readP.m). In this step, we have to load the organize the huge amount of data from step 1, and then iteratively update the magnitude and phase images.

Step 4: Post-processing (demo_combine.m). This is about organizing the reconstruction results so that it can be analyzed by some fancy models like DTI.

### Results
Since the raw Pfile is so large, instead we selected 6 directions and provided the loaded data from Step 1 for you in folder "example_data". By running our scripts, you should be able to get the following results. Usually, more directions will be used and more significant improvements can be visualized.

### Notes
I have to say I got moved when I first made the whole pipeline worked and I got a lot of help. Even though the current pipeline is already very complex (at least to me), there might be still several things which may further help,

(1) Other regularization terms on the magnitude images.

(2) Regularization term on the phase images. We did try L1-wavelet and totol variation but did not change much in this case. 

(3) Other initialization method. This is very important for this non-linear model. SENSE is fast and not a bad option. Maybe consider using deep learning as initialization (while this may be somwhat similar to [Dr. Berkin Bilgic](https://www.nmr.mgh.harvard.edu/~berkin/index.html)'s work).

