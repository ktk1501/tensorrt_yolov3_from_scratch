# tensorrt_demos

Examples demonstrating how to optimize caffe/tensorflow/darknet models with TensorRT and run inferencing on NVIDIA Jetson or x86_64 PC platforms.  Highlights:  (The FPS numbers in this README are test results against JetPack 4.3, i.e. TensorRT 6, on Jetson Nano.)

* Run an optimized "GoogLeNet" image classifier at ~60 FPS on Jetson Nano.
* Run a very accurate optimized "MTCNN" face detector at 6~11 FPS on Jetson Nano.
* Run an optimized "ssd_mobilenet_v1_coco" object detector ("trt_ssd_async.py") at 27~28 FPS on Jetson Nano.
* Run an optimized "yolov3-416" object detector at ~3 FPS on Jetson Nano.
* Run an optimized **"yolov4-416"** object detector at ~3 FPS on Jetson Nano.
* All demos work on Jetson TX2, AGX Xavier, Xavier NX ([link](https://github.com/jkjung-avt/tensorrt_demos/issues/19#issue-517897927) and [link](https://github.com/jkjung-avt/tensorrt_demos/issues/30)), and run much faster!
* Furthermore, all demos should work on x86_64 PC with NVIDIA GPU(s) as well.  Some minor tweaks would be needed.  Please refer to [README_x86.md](https://github.com/jkjung-avt/tensorrt_demos/blob/master/README_x86.md) for more information.

Demo: YOLOv3
---------------

Along the same line as Demo #3, this demo showcases how to convert pre-trained YOLOv3 models through ONNX to TensorRT engines.  This demo also requires TensorRT "Python API" and has been verified working against TensorRT 5.x+.

Assuming this repository has been cloned at "${HOME}/project/tensorrt_demos", follow these steps:

1. Install "pycuda" in case you have not done so in Demo #3.  Note that installation script resides in the "ssd" folder.

   ```shell
   $ cd ${HOME}/project/tensorrt_demos/ssd
   $ ./install_pycuda.sh
   ```

2. Install version "1.4.1" (**not the latest version**) of python3 "onnx" module.  Reference: [information provided by NVIDIA](https://devtalk.nvidia.com/default/topic/1052153/jetson-nano/tensorrt-backend-for-onnx-on-jetson-nano/post/5347666/#5347666).

   ```shell
   $ sudo pip3 install onnx==1.4.1
   ```

3. Download the pre-trained YOLOv3 COCO models and convert the targeted model to ONNX and then to TensorRT engine.  This demo supports 5 models: "yolov3-tiny-288", "yolov3-tiny-416", "yolov3-288", "yolov3-416", and "yolov3-608".  In addition, the code should also work for custom YOLOv3 models, or even models with different width and height for the input, e.g. yolov3-416x256.  Refer to my [TensorRT YOLOv3 For Custom Trained Models](https://jkjung-avt.github.io/trt-yolov3-custom/) blog post for more information.

   I use "yolov3-416" as example below.

   ```shell
   $ cd ${HOME}/project/tensorrt_demos/yolo
   $ ./download_yolov3.sh
   $ python3 yolo_to_onnx.py --model yolov3-416
   $ python3 onnx_to_tensorrt.py --model yolov3-416
   ```

   The last step ("onnx_to_tensorrt.py") takes a little bit more than half an hour to complete on my Jetson Nano DevKit.  When that is done, the optimized TensorRT engine would be saved as "yolov3-416.trt".

4. Test the YOLOv3 TensorRT engine with the "dog.jpg" image.

   ```shell
   $ wget https://raw.githubusercontent.com/pjreddie/darknet/master/data/dog.jpg -O ${HOME}/Pictures/dog.jpg
   $ python3 trt_yolo.py --model yolov3-416 \
                         --image --filename ${HOME}/Pictures/dog.jpg
   ```

   This was tested against JetPack-4.3, i.e. TensorRT 6.

   ![YOLOv3-416 detection result on dog.jpg](https://raw.githubusercontent.com/jkjung-avt/tensorrt_demos/master/doc/dog_trt_yolov3.png)

5. The "trt_yolo.py" demo program could also take various image inputs.  Refer to step 5 in Demo #1 again.

6. Similar to step 5 of Demo #3, I also created "eval_yolo.py" for evaluating mAP of the optimized YOLOv3 engines.

   ```shell
   $ python3 eval_yolo.py --model yolov3-tiny-288
   $ python3 eval_yolo.py --model yolov3-tiny-416
   $ python3 eval_yolo.py --model yolov3-288
   $ python3 eval_yolo.py --model yolov3-416
   $ python3 eval_yolo.py --model yolov3-608
   ```

   I evaluated all of yolov3-tiny-288, yolov3-tiny-416, yolov3-288, yolov3-416 and yolov3-608 TensorRT engines with COCO "val2017" data and got the following results.  The FPS (frames per second) numbers were measured using "trt_yolov3.py" on my Jetson Nano DevKit with JetPack-4.3.

   | TensorRT engine        | mAP @<br>IoU=0.5:0.95 |  mAP @<br>IoU=0.5  | FPS on Nano |
   |:-----------------------|:---------------------:|:------------------:|:-----------:|
   | yolov3-tiny-288 (FP16) |          0.077        |        0.158       |     20.9    |
   | yolov3-tiny-416 (FP16) |          0.096        |        0.202       |     14.2    |
   | yolov3-288 (FP16)      |          0.331        |        0.600       |     5.42    |
   | yolov3-416 (FP16)      |          0.373        |        0.664       |     3.07    |
   | yolov3-608 (FP16)      |          0.376        |        0.665       |     1.53    |
   | yolov3-608 (FP32)      |          0.376        |        0.665       |      --     |

7. Check out my blog post for implementation details:

   * [TensorRT ONNX YOLOv3](https://jkjung-avt.github.io/tensorrt-yolov3/)
   * [Verifying mAP of TensorRT Optimized SSD and YOLOv3 Models](https://jkjung-avt.github.io/trt-detection-map/)
   * For adapting the code to your own custom trained YOLOv3 models: [TensorRT YOLOv3 For Custom Trained Models](https://jkjung-avt.github.io/trt-yolov3-custom/)

