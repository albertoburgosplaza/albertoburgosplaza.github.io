---
layout: post
title:  "Developing a motion detector"
date:   2022-11-27 10:00:00 +0100
categories: python computer-vision
---
Using a Deep Learning model is a high resource consuming task. Sometimes, we need to run a model on a device with limited resources or on a device that is powered by a battery. In these cases, we need to find a way to reduce the resource consumption of the model. Also, some Deep Learning tasks, like detecting people in front of a camera, don't need to be running the whole time. We can save resources by running the model only when there is a change in the scene. In this post, we will see how to develop a motion detector using OpenCV and Python that can be used to trigger a Deep Learning model.

First, we need to import the necessary libraries:

```python
import cv2
import pandas as pd
import matplotlib.pyplot as plt

print(cv2.__version__)
```

```python
4.6.0
```

I'm going to use [frames 1 and 10 from video 2 of MOT20 dataset](https://motchallenge.net/vis/MOT20-02) to show you how to check the difference between two images. To keep it simple, I will convert the images to grayscale in order to reduce the number of channels.

```python
frame_1 = cv2.cvtColor(cv2.imread("frame-1.png"), cv2.COLOR_BGR2GRAY)
frame_10 = cv2.cvtColor(cv2.imread("frame-10.png"), cv2.COLOR_BGR2GRAY)

fig, ax = plt.subplots(1, 2, figsize=(10, 5))
ax[0].imshow(frame_1, cmap="gray")
ax[1].imshow(frame_10, cmap="gray")
plt.show()
```

![image info](/assets/images/frame-1_frame-10_MOT2002.png)

Using OpenCV's [absdiff](https://docs.opencv.org/4.6.0/d2/de8/group__core__array.html#ga6fef31bc8c4071cbc114a758a2b79c14) function we can see the per-element absolute difference between both frames ranging from 0 (no difference) to 255 (maximum difference).

```python
diff_frame = cv2.absdiff(frame_1, frame_10)
diff_frame
```

```python
array([[ 3,  3,  2, ..., 14, 11, 10],
       [ 4,  3,  1, ...,  7,  5,  5],
       [ 3,  3,  1, ...,  4,  4,  4],
       ...,
       [15,  3,  3, ...,  3,  6,  3],
       [10,  4, 10, ...,  3,  4,  4],
       [ 4, 15, 19, ...,  1,  6,  9]], dtype=uint8)
```

```python
plt.imshow(diff_frame, cmap="gray")
plt.show()
```

![image info](/assets/images/frame-1_frame-10_MOT2002_differences.png)


Then we can use a threshold to check if there is a difference between the two images. If the difference is greater than the threshold, we can say that there is a change in the scene. The lower the threshold, the more sensitive the detector will be. The higher the threshold, the less sensitive the detector will be.

We can see the histogram of differences in order to select an appropriate threshold.

```python
pd.Series(diff_frame.flatten()).hist(bins=100)
plt.show()
```

![image info](/assets/images/frame-1_frame-10_MOT2002_differences_histogram.png)

```python
pd.Series(diff_frame.flatten()).describe()
```

```python
count    518400.000000
mean         15.612299
std          26.149980
min           0.000000
25%           1.000000
50%           5.000000
75%          17.000000
max         234.000000
dtype: float64
```

In this case, I will use a threshold of 100 to detect changes in the scene.

```python
(diff_frame > 100).any()
```

```python
True
```

Now, we can rearrange the code to check the difference between the sequential frames that we can get from a camera and print the date and time when there is a change in the scene. You can check the full code in [this GitHub Gist](https://gist.github.com/albertoburgosplaza/003f782c2c6bef14869262510b6861c6).
