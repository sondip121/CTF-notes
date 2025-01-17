---
title: Image Manipulation for Machine Learning
description: We can update each pixel value to change an image.
tags: 
    - Computer Vision
    - Machine Learning
refs:
date: 2023-08-20
draft: false
---

## Swapping Pixels

Reference: [https://www.kaggle.com/code/jonbown/ai-ctf-submissions?scriptVersionId=105606691&cellId=102](https://www.kaggle.com/code/jonbown/ai-ctf-submissions?scriptVersionId=105606691&cellId=102)

This example updates pixel values at specified positions.

```python
import numpy as np
from PIL import Image

img = Image.open("example.png")

# Reshape image data to desired size for easy processing
pixels = np.array(img.getdata())
pixels = np.reshape(pixels, (28, 28))

# Update each pixel with desired value for changing image
for i in range(img.size[0]):
	for j in range(img.size[1]):
		# change pixel value at position (8, 19)
		if i == 8 and j == 19:
			pixels[i, j] = 255
		# change pixel value at position 25th row, 20th column onwards
		if i > 25 and j > 20:
			pixels[i, j] = np.random.randint(0, 50)

# Convert numpy array to image
img_updated = Image.fromarray(pixels.astype(np.uint8))
```
