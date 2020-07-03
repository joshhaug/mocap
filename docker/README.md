# Run VideoPose3D in a Docker Container

This Docker container allows you to run the VideoPose3D project without having to mess with any dependencies.  To get this project working, you will need a machine with an NVIDIA card. You don't need to have CUDA installed on your host machine, it will be installed at the container level!

My machine setup is:

- PopOS 20.04 LTS
- Graphics Card: GeForce GTX 1060 (3GB)

### Install Docker on Ubuntu

I'm going to install Docker first to prevent totally screwing up my existing system.

- Need to run the following commands to get GPU support:
  
  - sudo apt-get update
  
  - sudo apt-get install -y nvidia-container-toolkit

If you have a Windows machine, install [Docker Desktop](https://www.docker.com/products/docker-desktop)

### Building and Running the Docker Container

- `cd` to this directory (the one with a  `Dockerfile`)

- To build the docker image, run `sudo docker build --tag detectron .`
  
  - This will take several minutes to complete

- Once built, run: `sudo docker run --gpus all -it detectron bash`
  
  - It should drop you into a shell inside the container
  
  - To exit the container, use the `exit` command (or just press CTRL+D)
* To give the Docker container access to your local files (e.g. videos to run inference on) you'll need to include the `-v` flag in the call to `docker run`
  
  * On my host machine, I have a folder `/home/Mocap` that looks like this:

```
Mocap
├── output
│   └── (output files will go here)
├── input
│   └── Shannon_Game_Moves_50fps.mp4
```

- To allow the container access to this directory, run: `sudo docker run --gpus all -v /home/josh/Mocap:/Mocap -it detectron bash`
  
  - This maps the `Mocap` directory on my host machine to `/Mocap` in the container. Any changes made to this directory from within the container will  be made to the folder on the host machine, and vice versa. Docker documentation calls this "binding". [See here for more information.](https://docs.docker.com/storage/bind-mounts/)
  
  - The sytnax is: `-v HOST_DIRECTORY:TARGET_DIRECTORY`

### Running Inference on Videos

* Now that we are in the container, `cd` to the `detectron` directory and run the following to begin inference on the video in the `/Mocap/input` directory:

`python tools/infer_video.py --cfg configs/12_2017_baselines/e2e_keypoint_rcnn_R-101-FPN_s1x.yaml --output-dir /Mocap/output --image-ext mp4 --wts weights/model_final.pkl /Mocap/input`

Depending on your GPU memory, you may get an error that looks like this:

```
terminate called after throwing an instance of 'caffe2::EnforceNotMet'
  what():  [enforce fail at context_gpu.cu:343] error == cudaSuccess. 2 vs 0. Error at: /var/lib/jenkins/workspace/caffe2/core/context_gpu.cu:343: out of memory Error from operator: 
...
av_interleaved_write_frame(): Broken pipe
frame=    2 fps=0.0 q=-0.0 Lsize=    3835kB time=00:00:00.04 bitrate=785376.0kbits/s    
video:3835kB audio:0kB subtitle:0kB other streams:0kB global headers:0kB muxing overhead: 0.000000%
Conversion failed!
Aborted (core dumped)
```

* I was able to fix this error by running `vim /detectron/configs/12_2017_baselines/e2e_keypoint_rcnn_R-101-FPN_s1x.yaml` and changing SCALE in the TEST section from 800 to 300. This solution is based on [this issue](https://github.com/facebookresearch/Detectron/issues/812). I have also had success with 400. This parameter may need to be experimentally determined for your GPU setup.

If everything works, you should end up with an `.npz` file in the specified `output` directory. These are 2D keypoints. See [VideoPose3D's INFERENCE.md](https://github.com/facebookresearch/VideoPose3D/blob/master/INFERENCE.md#step-5-rendering-a-custom-video-and-exporting-coordinates) for more information.

To convert to 3D skeleton data, run the following commands:

* `cd /VideoPose3D/data` 
* `python3 prepare_data_2d_custom.py -i /Mocap/output -o myoutput`
* `cd /VideoPose3D`
* `python3 run.py -d custom -k myoutput -arc 3,3,3,3,3 -c checkpoint --evaluate pretrained_h36m_detectron_coco.bin --render --viz-subject Shannon_Game_Moves_50fps.mp4 --viz-action custom --viz-camera 0 --viz-video /Mocap/input/Shannon_Game_Moves_50fps.mp4 --viz-output /Mocap/output/output.mp4 --viz-size 6`

This will output a rendered video with a side-by-side comparison of keypoints and skeleton information.

To get the 3D data as a `.npy` file, use this call to `run.py` instead:

`python3 run.py -d custom -k myoutput -arc 3,3,3,3,3 -c checkpoint --evaluate pretrained_h36m_detectron_coco.bin --render --viz-subject Shannon_Game_Moves_50fps.mp4 --viz-action custom --viz-camera 0 --viz-video /Mocap/input/Shannon_Game_Moves_50fps.mp4 --viz-export /Mocap/output/output --viz-size 6`

The only difference is that this one uses the `--viz-export` argument in place of `--viz-output`.    



### Interpreting the Keypoint Motions

[Detectron/keypoints.py at 058768239b2d859c7350f633609fddb3a9094a91 · facebookresearch/Detectron · GitHub](https://github.com/facebookresearch/Detectron/blob/058768239b2d859c7350f633609fddb3a9094a91/detectron/utils/keypoints.py#L34-L51)  



Plotting this, the points are in the following order for each frame:

1. center of hips

2. right hip

3. right knee

4. right foot

5. left hip

6. left knee

7. left foot

8. spine center

9. spine top

10. nose

11. top of head

12. left shouder

13. left elbow

14. left hand

15. right shoulder

16. right elbow

17. right hand



Information on these `.npz` files is here:

https://github.com/facebookresearch/VideoPose3D/issues/2
