# Use Caffe2 image as parent image
FROM caffe2/caffe2:snapshot-py2-cuda9.0-cudnn7-ubuntu16.04

RUN mv /usr/local/caffe2 /usr/local/caffe2_build
ENV Caffe2_DIR /usr/local/caffe2_build

ENV PYTHONPATH /usr/local/caffe2_build:${PYTHONPATH}
ENV LD_LIBRARY_PATH /usr/local/caffe2_build/lib:${LD_LIBRARY_PATH}

# Clone the Detectron repository
RUN git clone https://github.com/facebookresearch/detectron /detectron

# Install Python dependencies
RUN pip install -r /detectron/requirements.txt

# Install the COCO API
RUN git clone https://github.com/cocodataset/cocoapi.git /cocoapi
WORKDIR /cocoapi/PythonAPI
RUN make install

# Go to the detectron directory
WORKDIR /detectron

# Set up Python modules
RUN make

# Download some dependencies from apt
RUN apt-get update
RUN apt-get install -y wget vim ffmpeg

WORKDIR /

# Clone the VideoPose3D project
RUN git clone https://github.com/facebookresearch/VideoPose3D/

# Move this script to the detetron/tools directory
RUN cp /VideoPose3D/inference/infer_video.py /detectron/tools/

# Download pretrained weights
RUN wget -P /detectron/checkpoint/ https://dl.fbaipublicfiles.com/video-pose-3d/pretrained_h36m_detectron_coco.bin 

# Avoid having to re-download weights each time
RUN mkdir /detectron/weights
RUN wget -P /detectron/weights/ https://dl.fbaipublicfiles.com/detectron/37698009/12_2017_baselines/e2e_keypoint_rcnn_R-101-FPN_s1x.yaml.08_45_57.YkrJgP6O/output/train/keypoints_coco_2014_train:keypoints_coco_2014_valminusminival/generalized_rcnn/model_final.pkl



# [Optional] Build custom ops
#RUN make ops
