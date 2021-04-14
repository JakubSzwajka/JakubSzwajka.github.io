---
layout: post
title: Maping coordinates from video frame to technical projection 🤓
published: false
excerpt_separator: <!--more-->
---


Many google search results led me here. Hope one of yours will lead you here and this post will shorten your suffering.

<!--more-->

### But first... quick background! 

This is me in one of the frames in video (left picture). Assume that camera position in stable, something like surveillance camera. Our goal is to make system witch will tell us where we are in flat space on something like technical projection (right picture). And as always we will use Python. 

![homography_idea](https://github.com/JakubSzwajka/JakubSzwajka.github.io/blob/master/_posts/_images/homography_1.png?raw=true) 

### Just use homography 🤷‍♀️

As I said, I tried lots of different approaches which didn't work for me and then in some [YT](https://www.youtube.com/watch?v=fVJeJMWZcq8) lecture, I found ⭐homography⭐.

I will try to explain it to you in very simple words. I mention our problem is to find corelation between two flat spaces. Basicaly the screen and technical projection on space. That corelation is called homography 🤯. 

If you are searching for detailed stuff go to [wiki page](https://en.wikipedia.org/wiki/Homography_(computer_vision)), I will cover using it with python in my case. 

### What we need? 

Choose four points on your video, if you have more than four it is even better. Now you need to measure where in 2d space of technical projection those points are. Yes, I see it as a disadvantage of this solution too... you have to be able to be where your camera is pointing to. Another way is to assume more or less distances between those points.

So now we have our input. In my project I've made something like camera settings. For each camera there is a JSON file with input values to calculate homography.  

```json
{
    "name" : "camera_1",
    "src" : [[501, 1013],[1289, 1065], [849, 363], [1058, 524]],
    "dst" : [[20, 400],[210,400],[60,80],[150,200]]
}
```

## Here comes OpenCV

Finally, let's do something! You need two methods. 

`` cv2.getPerspectiveTransform( here_goes_list_of_src_points, here_goes_list_of_dst_points )``

This will return calculated homography between two flat spaces we were talking about earlier.  

`` cv2.perspectiveTransform( points_we_want_to_map, our_homography )``

This will map other points. 

```Python 
    import numpy as np

    pts_src = np.float32(cameraConfig['src'])
    pts_dst = np.float32(cameraConfig['dst'])

    homography = cv2.getPerspectiveTransform(pts_src, pts_dst)

    coords_to_map = np.float32([[ frame['x'], frame['y'] ]]).reshape(-1,1,2)
    mapped_coords = cv2.perspectiveTransform( coords_to_map, homography )

```

Basicaly that's it. Now go and build neighbor tracker around your house 😉.  