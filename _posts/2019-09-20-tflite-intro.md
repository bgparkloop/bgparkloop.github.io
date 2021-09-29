---
title: "[Study] TfLite Conversion"
categories:
  - TfLite
tags:
  - TfLite
  - Mobile Deep Learning
comments: true
toc: true
toc_sticky: true
toc_label: Title
toc_color: green
---

### Version 기준

- Tensorflow-gpu 2.3.0 기준

```bash
import tensorflow as tf
from tensorflow import keras

model = keras.applications.mobilenet_v2.MobileNetV2( {wanted options} )
modle.compile( {wanted} )

## After Training ##

# In code(script)
converter = tf.lite.TFLiteConverter.from_keras_model(model) # defined model
converter.optimizations = [tf.lite.Optimize.DEFAULT] # optimizer option
converter.target_spec.supported_types = [tf.float16] # quantization option
tflite_model = converter.convert()

with open(MODEL_OUTPUT_PATH, "wb") as f:
    f.write(tflite_model)

# 변환 확인
interpreter = tf.lite.Interpreter(model_content=tflite_model)
print()
print('==>interpreter.get_input_details()')
print(interpreter.get_input_details())
print()
print('==>interpreter.get_output_details()')
print(interpreter.get_output_details())
print()
interpreter.allocate_tensors()
```

Model Save Error

```bash
File "/home/bgpark/.local/lib/python3.6/site-packages/h5py/_hl/group.py", line 373, in __setitem__
    h5o.link(obj.id, self.id, name, lcpl=lcpl, lapl=self._lapl)
  File "h5py/_objects.pyx", line 54, in h5py._objects.with_phil.wrapper
  File "h5py/_objects.pyx", line 55, in h5py._objects.with_phil.wrapper
  File "h5py/h5o.pyx", line 202, in h5py.h5o.link
RuntimeError: Unable to create link (name already exists)

tensorflow/core/kernels/data/generator_dataset_op.cc:103] Error occurred when finalizing GeneratorDataset iterator: Failed precondition: Python interpreter state is not initialized. The process may be terminated.
	 [[{{node PyFunc}}]]
```

변환된 그래프의 input, output 이름 확인법

```bash
bazel build tensorflow/tools/graph_transforms:summarize_graph 
bazel-bin/tensorflow/tools/graph_transforms/summarize_graph --in_graph=../tflite_model/tflite_graph.pb

# 꼭 ,이후에 띄어쓰기가 없어야 함. 안그러면 에러뜸
# output_arrays='test1','test2',...

```

예시)

![image](/assets/imgs/tflite/00.png)