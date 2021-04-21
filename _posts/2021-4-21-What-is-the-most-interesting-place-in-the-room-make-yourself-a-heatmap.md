---
layout: post
title: What is the most interesting place in the room. Make yourself a heatmap.
published: false
excerpt_separator: <!--more-->
---

### Data 

Obviously if we talk about heatmaps, we talk about displaying some data. Today it is about displaying heatmap on some image. In this case it will be dog position in the garden. You can read how to obtain such [here](). Connect the dots, and you will have some cool system in your garden 😎.

First, I have to admit that I don't have a dog. Data will be random X and Y positions. 

```python 
import numpy as np

# 100 random pairs of x and y in range from 0 to 20
data = np.random.randint(0,20,(100,2))
```

### Image to put heatmap on it 

Here I will use my talent a bit 😎. 

![garden](https://github.com/JakubSzwajka/JakubSzwajka.github.io/blob/master/_posts/_images/garden.png?raw=true)
