---
description: >-
  A lot of the numbers that you will come across in movement.py are selected
  through trial and error. To better appreciate why they hold the value they
  hold, feel free to make changes and observe.
---

# Movement guide

{% hint style="info" %}
Function names mostly self explanatory. Explanation is provided for those that may have less intuitive mechanism.
{% endhint %}

### move\_then\_rotate\(\)

Customize values for `speed` and `times` to see the different performance.

{% code-tabs %}
{% code-tabs-item title="movement.py" %}
```python
def move_then_rotate(self):
        for i in range(10):
            self.rovio.forward(speed=1)
        time.sleep(0.2)
        self.rotate_left(times=10)
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### rotate\_left\(\)

Rotate Rovio to left side, with particular angle and speed for a particular number of times. 

{% code-tabs %}
{% code-tabs-item title="movement.py" %}
```python
def rotate_left(self,times=1,speed_=4,angle=None):
        if not angle is None:
            times = int(angle/36)
        for i in range(times):
            self.movements.append(self.rovio.rotate_left.__name__)
            self.rovio.rotate_left(speed=speed_)
            time.sleep(self.delay)
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### rotate\_left\_90\(\)

Self explanatory.

{% code-tabs %}
{% code-tabs-item title="movement.py" %}
```python
def rotate_left_90(self):
        self.rotate_left(times=2)
        self.rotate_left(speed_=6)
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### rotate\_left\_less\(\)

{% code-tabs %}
{% code-tabs-item title="movement.py" %}
```python
def rotate_left_less(self):
        self.rotate_left(speed_=7)
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### rotate\_right\(\)

{% code-tabs %}
{% code-tabs-item title="movement.py" %}
```python
def rotate_right(self, times=1,speed_=4,angle=None):
        print(angle)
        if not angle is None:
            times = int(angle/36)
        print(times)
        for i in range(times):
            self.movements.append(self.rovio.rotate_right.__name__)
            self.rovio.rotate_right(speed=speed_)
            time.sleep(self.delay)
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### rotate\_right\_90\(\) 

{% code-tabs %}
{% code-tabs-item title="movement.py" %}
```python
def rotate_right_90(self):
        self.rotate_right(times=2)
        self.rotate_right(speed_=6)
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### rotate\_right\_less\(\)

{% code-tabs %}
{% code-tabs-item title="movement.py" %}
```python
def rotate_right_less(self):
        self.rotate_right(speed_=7)
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### move\_more\(\)

Long march forward. 

{% code-tabs %}
{% code-tabs-item title="movement.py" %}
```python
def move_more(self, movement_op, times=5, speed_=1):
        for i in range(times):
            self.movements.append(movement_op.__name__)
            movement_op(speed=speed_)
        time.sleep(0.3)
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### move\_forward\(\)

{% code-tabs %}
{% code-tabs-item title="movement.py" %}
```python
def move_forward(self):
        self.move_more(self.rovio.forward)
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### move\_forward\_less\(\)

{% code-tabs %}
{% code-tabs-item title="movement.py" %}
```python
def move_forward_less(self):
        self.move_more(self.rovio.forward, times=3)
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### move\_backward\(\)

{% code-tabs %}
{% code-tabs-item title="movement.py" %}
```python
def move_backward(self):
        self.move_more(self.rovio.backward)
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### move\_backward\_less\(\)

{% code-tabs %}
{% code-tabs-item title="movement.py" %}
```python
 def move_backward_less(self):
        self.move_more(self.rovio.backward, times=1)
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### move\_left\(\)

{% code-tabs %}
{% code-tabs-item title="movement.py" %}
```python
def move_left(self):
        self.move_more(self.rovio.left)
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### move\_left\_straight\(\)

{% code-tabs %}
{% code-tabs-item title="movement.py" %}
```python
def move_left_straight(self):
        self.move_more(self.rovio.left, times=10)
        self.rotate_left_less()
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### move\_right\(\)

{% code-tabs %}
{% code-tabs-item title="movement.py" %}
```python
 def move_right(self):
        self.move_more(self.rovio.right)
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### move\_right\_straight\(\)

{% code-tabs %}
{% code-tabs-item title="movement.py" %}
```python
def move_right_straight(self):
        self.move_more(self.rovio.right, times=10)
        self.rotate_right_less()

```
{% endcode-tabs-item %}
{% endcode-tabs %}

### stop\(\)

{% code-tabs %}
{% code-tabs-item title="movement.py" %}
```python
self.rovio.stop()
        print('Caught you!')
        time.sleep(3)

```
{% endcode-tabs-item %}
{% endcode-tabs %}

### move\_undo\(\)

{% code-tabs %}
{% code-tabs-item title="movement.py" %}
```python
def move_undo(self, times=1):
        if len(self.movements) > 0:
            movements = self.movements
            for i in range(times):
                movement = self.get_reverse_movement(movements[-1])
                print(movement.__name__, len(movements))
                movement()
                movements = movements[:-1]
            self.movements = movements
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### get\_reverse\_movement\(\)

This will reverse a movement, to allow Rovio to undo its previous step.

{% code-tabs %}
{% code-tabs-item title="movement.py" %}
```python
def get_reverse_movement(self, movement):
        return {
            'rotate_left': self.rotate_right,
            'rotate_right': self.rotate_left,
            'forward': self.move_backward,
            'backward': self.move_forward,
            'left': self.move_right,
            'right': self.move_left
        }.get(movement, None)
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### backtrack\(\)

{% code-tabs %}
{% code-tabs-item title="movement.py" %}
```python
def backtrack(self):
        self.move_undo(times=len(self.movements))
```
{% endcode-tabs-item %}
{% endcode-tabs %}



