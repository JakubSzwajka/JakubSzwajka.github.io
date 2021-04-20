---
layout: post
title: Make Rick Astley sing in your console 🎙
published: true
excerpt_separator: <!--more-->
---

This is kind of post that I'm still thinking why I did that 🤔. But remember that not every line of code should be serious. Keep it fun! 🎙


<!--more-->

### Step first — Video 

I assume that you already have some .mp4 file. If not, you can use ``pytube``. Check out this snippet: 

```python
import pytube

class YT_video():
    def __init__(self, url): 
        self.url = url
        youtube = pytube.YouTube(self.url)
        self.video = youtube.streams.get_highest_resolution()

    def download(self, dest_folder = 'downloads'):
        self.video.download(dest_folder)

```

### Read file and display it with OpenCV

```python 
import cv2 

video = cv2.VideoCapture(video_path)

while True:
    _, frame = video.read()
    cv2.imshow("video", frame)
    
    if cv2.waitKey(1) & 0xFF == ord("q"):
        break

video.release()
cv2.destroyAllWindows()
```

Ok now we have our core. We play video, but there is no sound, it is playing to fast and our goal is to have it in console. 

Let's solve the problem one by one. 

### Sound

You can use ``ffpyplayer`` package. I added to our code next part for handling sound. It's only music player that will start song in the background. 

```python 
import cv2 

video = cv2.VideoCapture(video_path)
music_player = MediaPlayer(video_path)

while True:
    _, frame = video.read()    
    cv2.imshow("video", frame)
    
    if cv2.waitKey(1) & 0xFF == ord("q"):
        break

video.release()
cv2.destroyAllWindows()
```

But wait, Rick is dancing twice as fast as sings. That is not spreading joy 🤔.

### Speed 

To know FPS (Frames Per Second) we will use OpenCV. First we calculate how long should one frame last. Then we will wait a bit before displaying next one. 

```python 
fps = video.get(cv2.CAP_PROP_FPS)
seconds_per_frame = 1 / fps
```

Now add small wait in the end of out While. 

```python 
import cv2 

video = cv2.VideoCapture(video_path)
music_player = MediaPlayer(video_path)

while True:
    frame_t_start = time.time()
    
    _, frame = video.read()    
    cv2.imshow("video", frame)
    
    if cv2.waitKey(1) & 0xFF == ord("q"):
        break
    
    while ( time.time() - frame_t_start ) < seconds_per_frame:
        pass

video.release()
cv2.destroyAllWindows()
```

### To Console!

Before we print it to console, we need to make our video grayscale. Use below to change frame. I've added some threshold to make it more 'binary' for console. You can play with values ``treshold`` and ``treshold_type``. Check [docs](https://docs.opencv.org/master/d7/d4d/tutorial_py_thresholding.html) of OpenCV to read more. 

I set my ``treshold_type`` for 3 and ``treshold`` around 120.   

```python

frame = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
_, frame = cv2.threshold(frame, treshold, 255, treshold_type )

```

Now our image is a 2D matrix of numbers. We will replace every number with some character. Assuming that our grayscale is from 0 to 255, find a char that suits perfect. 

Assume that our grayscale is such string: 
```python
GRAY_SCALE = "@$#*!=;:~-,. "
```

Method for mapping number to char will be: 
```python 

def grayScaleNumber(num):
    scale_size = len(GRAY_SCALE)
    index = (int((num / 255) * scale_size) - 1)
    return GRAY_SCALE[index]

```
Let's add simple print method instead of ``cv2.imshow("video",frame)``. One more thing here is to resize the video. OpenCV can handle it for us.

```python
def printFrameInConsole(frame, height, width):
    console_out_dim = ( int(width / SCALE),int(height / SCALE))
    frame = cv2.resize(frame, console_out_dim, interpolation = cv2.INTER_AREA)

    to_print = ''
    for row in frame: 
        to_print += ' '.join([ grayScaleNumber(num) for num in row.tolist()]) + "\n"

    sys.stdout.write(to_print)
    sys.stdout.flush()

```

### Enjoy 

Remember to play with it a bit. Check out some different thresholds and threshold types. Find the best Rick Astley for yourself. 

I will upload some video as soon as I can. Now admire console Rick 😅

![Rick_astley](https://github.com/JakubSzwajka/JakubSzwajka.github.io/blob/master/_posts/_images/Rick_Astley_1.png?raw=true)











