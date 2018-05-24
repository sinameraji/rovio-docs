---
description: Guide on Rovio and obstacle recognition algorithms
---

# Image recognition

**Check out the Readme on our Github repo first:** [**https://github.com/noob-ja/SeekerRovio/tree/master/imgProcessing**](https://github.com/noob-ja/SeekerRovio/tree/master/imgProcessing)



{% hint style="info" %}
Original library used in recognizing can be found here: [https://github.com/pierluigiferrari/ssd\_keras](https://github.com/pierluigiferrari/ssd_keras)
{% endhint %}

### Dependencies

* Python 3.x \(Python 2.7 seems to work, but isn't actively supported\)
* Numpy
* TensorFlow 1.x
* Keras 2.x
* OpenCV
* Beautiful Soup 4.x



## Obstacle detection

There is a great guide for this on OpenCV website, check it out: [https://docs.opencv.org/3.0-beta/doc/py\_tutorials/py\_imgproc/py\_colorspaces/py\_colorspaces.html](https://docs.opencv.org/3.0-beta/doc/py_tutorials/py_imgproc/py_colorspaces/py_colorspaces.html)

Basically, we detect obstacles based on color. 

![Before ](.gitbook/assets/image%20%281%29.png)

![After](.gitbook/assets/image%20%282%29.png)



{% code-tabs %}
{% code-tabs-item title="imgProcessing/process.py" %}
```python
class ObstacleDetect(object):
    def __init__(self, bound=20, screen_height=480, screen_width=640):
        self.bound = bound
        self.screen_height = screen_height
        self.screen_width = screen_width
        self.screen_height_h = self.screen_height/2
        self.screen_width_h = self.screen_width/2

    def preprocess_image(self,frame):
        # frame = cv2.imread(frame)
        pink = np.uint8([[[147,20,255]]])
        hsv_pink = cv2.cvtColor(pink,cv2.COLOR_BGR2HSV)

        hsv = cv2.cvtColor(frame, cv2.COLOR_BGR2HSV)

        lower_bound = np.array([hsv_pink[0][0][0]-self.bound,147,147])
        upper_bound = np.array([hsv_pink[0][0][0]+self.bound,255,255])

        mask = cv2.inRange(hsv, lower_bound, upper_bound)
        res = cv2.bitwise_and(frame,frame, mask=mask)

        # morphology
        kernel = np.ones((5, 5), np.uint8)
        opening = cv2.morphologyEx(res, cv2.MORPH_OPEN, kernel)
        closing = cv2.morphologyEx(opening, cv2.MORPH_CLOSE, kernel)
        final_result = closing.copy()
        final_result = cv2.cvtColor(final_result, cv2.COLOR_RGB2GRAY)

        return final_result
```
{% endcode-tabs-item %}
{% endcode-tabs %}

