---
sidebar_position: 1
---

# ONNX Runtime

[ONNX Runtime](https://onnxruntime.ai/) is a cross-platform inference and training machine-learning accelerator. To get faster inference speeds for applications, we developed the SpaceMIT Execution Provider, which uses extended AI instructions for acceleration. Simply specify `SpaceMITExecutionProvider` when developing the application.

## Platform Support

| Platform & System     | Acceleration Support |
|-----------------------|----------------------|
| K1 Buildroot          | ✅ Supported         |
| K1 OpenHarmony 5.0    | ❌ Not supported     |
| K3 Bianbu LXQT/GNOME  | ✅ Supported         |

## Installation

```shell
sudo apt-get update
sudo apt-get install -y onnxruntime python3-spacemit-ort libopencv-dev
```

Verify the installation:

```shell
python3
>>> import onnxruntime as ort
>>> import spacemit_ort
>>> available_providers = ort.get_available_providers()
>>> print(available_providers)
['SpaceMITExecutionProvider', 'CPUExecutionProvider']
```

The final output `['SpaceMITExecutionProvider', 'CPUExecutionProvider']` indicates a successful installation.

## Using Python API

### Install dependencies

```shell
sudo apt-get install -y python3-onnx python3-pillow python3-matplotlib python3-opencv
```

### Prepare model, label and image

We provide quantized open-source models, which can be downloaded from [ModelZoo](https://archive.spacemit.com/spacemit-ai/ModelZoo/). In theory, models from the [ONNX Model Zoo] can also be used for inference, but their performance is slightly inferior to our quantized models.

```shell
wget https://archive.spacemit.com/spacemit-ai/ModelZoo/classification.tar.gz
tar -zxf classification.tar.gz
wget -P classification https://archive.spacemit.com/spacemit-ai/ModelZoo/classification/kitten.jpg
wget -P classification https://archive.spacemit.com/spacemit-ai/ModelZoo/classification/synset.txt
```

### Import dependencies

Verify that all dependencies are installed using the cell below. Continue if no errors encountered, warnings can be ignored.

```python
import onnx
import numpy as np
import onnxruntime as ort
import spacemit_ort
from PIL import Image
import cv2
import matplotlib.pyplot as plt
import time
```

### Read image

```python
def get_image(path, show=False):
    with Image.open(path) as img:
        img = np.array(img.convert('RGB'))
    if show:
        plt.imshow(img)
        plt.axis('off')
    return img
```

### Preprocess image

Preprocess inference image -> scale to 0~1, resize to 256x256, take center crop of 224x224, normalize image, transpose to NCHW format, cast to float32 and add a dimension to batchify the image.

```python
def preprocess(img):
    img = img / 255.
    img = cv2.resize(img, (256, 256))
    h, w = img.shape[0], img.shape[1]
    y0 = (h - 224) // 2
    x0 = (w - 224) // 2
    img = img[y0 : y0+224, x0 : x0+224, :]
    img = (img - [0.485, 0.456, 0.406]) / [0.229, 0.224, 0.225]
    img = np.transpose(img, axes=[2, 0, 1])
    img = img.astype(np.float32)
    img = np.expand_dims(img, axis=0)
    return img
```

### Predict

```python
def predict(path):
    img = get_image(path, show=True)
    img = preprocess(img)
    ort_inputs = {session.get_inputs()[0].name: img}
    start = time.time()
    preds = session.run(None, ort_inputs)[0]
    end = time.time()
    preds = np.squeeze(preds)
    a = np.argsort(preds)[::-1]
    print('time=%.2fms; class=%s; probability=%f' % (round((end-start) * 1000, 2), labels[a[0]], preds[a[0]]))
```

### Load model, labels, and run inference

```python
with open('classification/synset.txt', 'r') as f:
    labels = [l.rstrip() for l in f]

# Set the number of inference threads
session_options = ort.SessionOptions()
session_options.intra_op_num_threads = 2

# IMPORTANT: Specify the SpaceMITExecutionProvider
session = ort.InferenceSession('classification/resnet50/resnet50.q.onnx', sess_options=session_options, providers=["SpaceMITExecutionProvider"])

img_path = 'classification/kitten.jpg'
predict(img_path)
```

Output:

```shell
time=95.15ms; class=n02123045 tabby, tabby cat; probability=13.498201
```

### API Overview

For more APIs, please refer to the official [API Overview](https://onnxruntime.ai/docs/api/python/api_summary.html).
