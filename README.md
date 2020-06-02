# Requirements
  
  1. Python2.7
  ``` shell
  $ conda create -n darknet2caffe python=2.7 # Create virutal env has python version of 2.7
  $ conda activate darknet2caffe # Activate virtual env
  $ conda install cython scikit-image ipython h5py nose pandas protobuf pyyaml # Install Deps
  $ cd $HOME/caffe/python
  $ for req in $(cat requirements.txt); do pip install $req; done  # Install Requirements
  ```

  2. Caffe
  * Modify Makefile.config and make sure the `CONDA_VENV_HOME` is set to your anaconda directory
  * Modify source code as [Add Caffe Layers](##add-caffe-layers)
  * Compile Caffe
  ``` shell
  $ cd $HOME && git clone https://github.com/BVLC/caffe.git
  $ cd $HOME && git clone https://github.com/es6rc/darknet2caffe.git
  $ cd darknet2caffe
  $ cp ./Makefile.config $HOME/caffe # Modify in Makefile.config 
  $ cd $HOME/caffe
  $ make all
  $ make pycaffe
  $ echo 'export PYTHONPATH=$HOME/caffe/python:$PYTHONPATH' >> $HOME/.bashrc # Add Python Path for Caffe
  ```

  3. Pytorch >= 0.40
  ``` shell
  $ conda activate darknet2caffe # Activate virtual env
  $ pip install torch
  $ pip install future # Install dependencies
  ```


## Add Caffe Layers

1. Copy `caffe_layers/mish_layer/mish_layer.hpp,caffe_layers/upsample_layer/upsample_layer.hpp` into `include/caffe/layers/`.
```
$ cd $HOME/darknet2caffe
$ cp ./caffe_layers/mish_layer/mish_layer.hpp $HOME/caffe/include/caffe/layers/
$ cp ./caffe_layers/upsample_layer/upsample_layer.hpp $HOME/caffe/include/caffe/layers/
```

2. Copy `caffe_layers/mish_layer/mish_layer.cpp mish_layer.cu,caffe_layers/upsample_layer/upsample_layer.cpp upsample_layer.cu` into `src/caffe/layers/`.
```
$ cd $HOME/darknet2caffe
$ cp caffe_layers/mish_layer/mish_layer.c* $HOME/caffe/src/caffe/layers/
$ cp caffe_layers/upsample_layer/upsample_layer.c* $HOME/caffe/src/caffe/layers/
```

3. Copy `caffe_layers/pooling_layer/pooling_layer.cpp` into `src/caffe/layers/`.Note:only work for yolov3-tiny,use with caution.
```
$ cd $HOME/darknet2caffe
$ cp caffe_layers/pooling_layer/pooling_layer.cpp $HOME/caffe/src/caffe/layers/
```

4. Add below code into `src/caffe/proto/caffe.proto`.
``` c++
// LayerParameter next available layer-specific ID: 147 (last added: recurrent_param)
message LayerParameter {
  optional TileParameter tile_param = 138;
  optional VideoDataParameter video_data_param = 207;
  optional WindowDataParameter window_data_param = 129;
++optional UpsampleParameter upsample_param = 149; //added by chen for Yolov3, make sure this id 149 not the same as before.
++optional MishParameter mish_param = 150; //added by chen for yolov4,make sure this id 150 not the same as before.
}
++message VideoDataParameter{
++  enum VideoType {
++    WEBCAM = 0;
++    VIDEO = 1;
++  }
++  optional VideoType video_type = 1 [default = WEBCAM];
++  optional int32 device_id = 2 [default = 0];
++  optional string video_file = 3;
++  // Number of frames to be skipped before processing a frame.
++  optional uint32 skip_frames = 4 [default = 0];
++}
// added by chen for YoloV3
++message UpsampleParameter{
++  optional int32 scale = 1 [default = 1];
++}

// Message that stores parameters used by MishLayer
++message MishParameter {
++  enum Engine {
++    DEFAULT = 0;
++    CAFFE = 1;
++    CUDNN = 2;
++  }
++  optional Engine engine = 2 [default = DEFAULT];
++}
```
5.(re)make caffe.

# Demo
``` shell
  $ python darknet2caffe.py cfg[in] weights[in] prototxt[out] caffemodel[out]
``` 

  Example
``` shell
$ mkdir caffemodel # o/w throw core dump error
$ python darknet2caffe.py ./cfg/ods_yolov3.cfg ./weights/ods_yolov3_final.weights ./prototxt/ods_yolov3.prototxt ./caffemodel/ods_yolov3.caffemodel
```
  partial log as below.
```
I0522 10:19:19.015708 25251 net.cpp:228] layer1-act does not need backward computation.
I0522 10:19:19.015712 25251 net.cpp:228] layer1-scale does not need backward computation.
I0522 10:19:19.015714 25251 net.cpp:228] layer1-bn does not need backward computation.
I0522 10:19:19.015718 25251 net.cpp:228] layer1-conv does not need backward computation.
I0522 10:19:19.015722 25251 net.cpp:228] input does not need backward computation.
I0522 10:19:19.015725 25251 net.cpp:270] This network produces output layer139-conv
I0522 10:19:19.015731 25251 net.cpp:270] This network produces output layer150-conv
I0522 10:19:19.015736 25251 net.cpp:270] This network produces output layer161-conv
I0522 10:19:19.015911 25251 net.cpp:283] Network initialization done.
unknow layer type yolo 
unknow layer type yolo 
save prototxt to prototxt/yolov4.prototxt
save caffemodel to caffemodel/yolov4.caffemodel
```
