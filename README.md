__Check this repository !!!
<https://github.com/ktk1501/tensorRT_Pytorch>__

YOLOv3
---------------

# yolov3 tensorrt video inference
## yolov3_to_onnx
I forked a great repo of tensorrt demos, and customized it only for yolov3 inference.
I used official sample in TensorRT 7.0.0.11, 'TensorRt-7.0.0.11/samples/python/yolov3_onnx/yolov3_to_onnx.py' because there were some issues on the forked files.
Convert yolov3 darknet based model to onnx.

## onnx_to_tensorrt
Same as above, I used 'onnx_to_tensorrt.py' in the same folder. 
convert onnx to trt model.

## testing environment
|GPU  | RTX 2080 Ti |
|---|:---:|
| CPU | Intel i7-8700 @ 3.20GHz |
| CUDA | 10.0 |
| CuDNN | 7.6.5.32|

## Step 1 : default setting

## Step 2 : increase max_workspace_size
I increased the 'max_workspace_size' from 28 to 33, which is 256mb to 8gb. (2^order) 

## Step 3 : FP16 precision
In 'onnx_to_tensorrt.py', you can choose precision mode by adding
```python
builder.fp16_mode = True
```
in 'build_engine()' function

## Step 4 : INT8 precision
Must have a custom calibrator. Need to be updated. 

## Results
|  | precision mode | max_workspace_size | FPS | Actually Allocated GPU Memory
|---|:---:|---:|---:| ---:|
| Step 1 | FP32 | 256mb | 24 | 1.29 GB |
| Step 2 | FP32 | 8GB | 24 | 1.31 GB |
| Step 3 | FP16 | 8GB | 30 | 0.95 GB
| Step 4 | INT8 | 8GB | - |



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

