---
description: >-
  How Rovio searches for another Rovio that is trying to hide behind available
  obstacles in an environment.
---

# Search algorithm

### get\_frame\(\)

This function will load the input from Rovio's front camera in a frame.

{% code-tabs %}
{% code-tabs-item title="search.py" %}
```python
def get_frame(self):
        if self.debug:
            print('=============================================')
        self.frame = None
        while self.frame is None:
            self.frame = self.rovio.camera.get_frame()
        if not self.rovio_json is None: self.rovio_json_ = self.rovio_json
        if not self.obs_json is None: self.obs_json_ = self.obs_json
        self.rovio_json = self.rovioDet(self.frame)
        self.rovio_found = not isinstance(self.rovio_json, str)
        self.obs_json = self.obsDet(self.frame)
        self.obs_found = not isinstance(self.obs_json, str)
        if not self.frame is None:
            self.frame = self.show_battery(self.frame)
            cv2.imshow('Obstacle', self.frame)
        if not self.rovioDet.processed_frame is None:
            # self.rovioDet.processed_frame = self.show_battery(self.rovioDet.processed_frame)
            cv2.imshow('Rovio', self.rovioDet.processed_frame)
        cv2.waitKey(1)
        if self.debug:
            print('rovio: ', self.rovio_json)
            print('obs: ', self.obs_json)
            print('=============================================')

```
{% endcode-tabs-item %}
{% endcode-tabs %}

### get\_ref\(\)

Returns the coordinates of the bottom edge an object \(Rovio or obstacle\) and the direction to which our bot should move, to reach that object.

{% code-tabs %}
{% code-tabs-item title="search.py" %}
```python
def get_ref(self, json, get_nearest=True):
        # if nothing found
        if isinstance(json, str):
            return json, -2, -2
        else:
            # if getting obstacle refer points
            if isinstance(json, list):
                ref_point = [x['ref_center'][1] for x in json]
                if get_nearest==True:
                    index = ref_point.index(max(ref_point))
                else:
                    index = ref_point.index(min(ref_point))
                ret_ref = json[index]['ref_center']
                direction = json[index]['direction']
                ret_json = json[index]
            else:   #getting rovio refer points
                ret_ref = json['ref_center']
                direction = json['direction']
                ret_json = json
            return ret_ref, direction, ret_json
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### avoidObs\(\)

In cases that Rovio is seeing obstacle AND the other Rovio in a frame, we use this function to make sure our Rovio will chase the hider \(the other Rovio\) without hitting the nearby obstacle.

`rovio_ref[1]` and `obs_ref[1]` return the Y coordinate \(from an \[X,Y\] array\). We compare these Y's \(the distance from top of the frame to the height of Rovio vs Obstacle\) to conclude which one of them is nearer.

{% code-tabs %}
{% code-tabs-item title="search.py" %}
```python
def avoidObs(self):
        self.get_frame()
        if self.rovio_found and self.obs_found:
            rovio_ref, rovio_dir, rovio_json = self.get_ref(self.rovio_json)
            obs_ref, obs_dir, obs_json = self.get_ref(self.obs_json)
            if rovio_ref[1] < obs_ref[1]:
                if obs_dir == 1:
                    self.move.move_left_straight()
                else: self.move.move_right_straight()
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### chase\(\)

Begin chasing the other Rovio.

{% code-tabs %}
{% code-tabs-item title="search.py" %}
```python
def chase(self):
        self.get_frame()
        if self.rovio_json == 'no rovio':
            if self.rovio_json_['direction'] == -1:
                self.move.rotate_left()
            elif self.rovio_json_['direction'] == 1:
                self.move.rotate_right()
            else:
                self.rovio.backward()
        else:
            self.avoidObs()
            self.move_towards('rovio')
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### object\_center\(\)

Important function for making sure our Rovio is turning to face the object such that the object is in the center of our frame.  

{% code-tabs %}
{% code-tabs-item title="search.py" %}
```python
def object_center(self, object, get_nearest=True):
        while True:
            self.get_frame()
            if object == 'rovio':
                if not self.rovio_found:
                    break
                json = self.rovio_json
            else:
                if not self.obs_found:
                    break
                json = self.obs_json
            ret_ref, direction, ret_json = self.get_ref(json, get_nearest=get_nearest)
            obj_center = abs(ret_ref[0] - self.screen_width_h)
            if obj_center > 200:
                if direction==1: self.move.rotate_right(speed_=6)
                else: self.move.rotate_left(speed_=6)
            else:
                break
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### search\(\)

While Rovio isn't found, we want our seeker bot to keep looking for nearest obstacles, march toward them and search around them. 

{% code-tabs %}
{% code-tabs-item title="search.py" %}
```python
def search(self,partial=4,get_nearest=True):
        self.get_frame()
        obs_json = self.obs_json
        if self.rovio_found:
            # init chasing sequence
            self.chase()
            print('')
        else:
            # if not self.obs_found:
            if self.debug: print('no obstacle found')
            obs_json = self.find(get_nearest)
            if obs_json is False:
                if self.debug: print('still no obstacle found')
                # begin traceback operation
                return ''
            elif obs_json is True:
                # found rovio
                return ''
            ref_obs, ref_obs_direction,_ = self.get_ref(obs_json, get_nearest=get_nearest)
            if ref_obs_direction == 1:    # obstacle on the right
                self.move_around(object='obstacle',times=partial, get_nearest=get_nearest)
            elif ref_obs_direction == -1:   # obstacle on the left
                self.move_around(object='obstacle',times=partial, get_nearest=get_nearest)
            else:
                if self.debug: print('facing obstacle')
        self.obs_json_ = obs_json
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### find\(\)

Turning 360 degree. If found Rovio then stop immediately, else continue turning. Once done turning 360 degree, turn towards the nearest/furthest obstacle.

{% code-tabs %}
{% code-tabs-item title="search.py" %}
```python
def find(self,get_nearest=True):
        self.get_frame()
        angle = 0
        obs_found = []
        # rotate and look for obs or rovio
        while angle<360:
            if self.rovio_found:
                if self.debug: print('found rovio at angle ', angle)
                return True
            if self.obs_found:
                if self.debug: print('found obstacle at angle ', angle)
                for obs_ in self.obs_json:
                    obs_['angle'] = angle
                    obs_found.append(obs_)
            self.move.rotate_left()
            angle += 36
            self.get_frame()

        if not (self.rovio_found or len(obs_found) > 0):
            if self.debug: print('rotated 360 degree, cannot find anything')
            return False
        else:
            if self.debug: print('found obs: ',obs_found)
            _,_,obs_to = self.get_ref(obs_found, get_nearest=get_nearest)
            turn_angle = obs_to['angle']
            print(obs_to, turn_angle)
            if turn_angle<=180:
                self.move.rotate_left(angle=turn_angle)
            else:
                self.move.rotate_right(angle=360-turn_angle)
            return obs_to
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### calc\_dist\(\)

Calculate distance between our bot and an object \(obstacle or Rovio\). 

{% code-tabs %}
{% code-tabs-item title="search.py" %}
```python
def calc_dist(self, json, object='rovio'):
        dist_btm = abs(self.screen_height - json['ref_center'][1])
        if object=='rovio':
            dist_center = abs(self.screen_height_q - json['ref_center_true'][1])
        else:
            dist_center = abs(self.screen_height_h - json['ref_center_true'][1])
        return dist_btm, dist_center
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### move\_towards\(\)

Move towards the object with a variable speed; faster at a distance, slower when nearer to avoid bumping into the object.

{% code-tabs %}
{% code-tabs-item title="search.py" %}
```python
def move_towards(self, object, get_nearest=True):
        if self.debug:
            print('________________________')
            print('Moving towards: ',object)
        move_stop = 100
        move_near = 150
        while True:
            self.object_center(object=object, get_nearest=get_nearest)
            self.get_frame()
            json = self.rovio_json if object=='rovio' else self.obs_json
            _,_, obj_json = self.get_ref(json,get_nearest=get_nearest)
            if isinstance(obj_json, int):
                if self.debug: print('lost target')
                self.lost = True
                # self.move.move_undo()
                return False
            obj_dist_btm, obj_dist_center = self.calc_dist(json=obj_json, object=object)
            print(object,'_dist_btm: ', obj_dist_btm)
            print(object,'_dist_center: ', obj_dist_center)
            if obj_dist_btm < move_stop and obj_dist_center < move_stop:
                return True
            elif obj_dist_btm < move_near or obj_dist_center < move_near:
                self.move.move_forward_less()
            else:
                self.move.move_forward()
        if self.debug:
            print('Moving towards done ')
            print('________________________')

```
{% endcode-tabs-item %}
{% endcode-tabs %}

### move\_around\(\)

Move around an obstacle. Default define around as moving pass all four sides of the obstacle.

{% code-tabs %}
{% code-tabs-item title="search.py" %}
```python
def move_around(self, object='obstacle',times=4, get_nearest=True):
        if self.debug:
            print('@@@@@@@@@@@@@@@@@@@@@@@@@@@@')
            print('Moving around: ',object)
        if object=='obstacle':
            if not self.move_towards(object=object,get_nearest=get_nearest):
                return ''
            for i in range(times):
                if self.move_pass()==1:
                    break    # move pass rovio
                # attempt to check for rovio
                self.move.move_backward()
                self.get_frame()
                if self.rovio_found: break
                else: self.move.move_forward()

                if i == times-1:
                    break

                if self.moving_direction: self.move.rotate_right_90()
                else: self.move.rotate_left_90()
                self.move.move_backward_less()
            self.moving_direction = not self.moving_direction

        if self.debug:
            print('Moving around done ')
            print('@@@@@@@@@@@@@@@@@@@@@@@@@@@@')
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### move\_pass\(\)

Move past an object: have to see it then if it can't see it, we conclude we've moved past it.

{% code-tabs %}
{% code-tabs-item title="search.py" %}
```python
def move_pass(self, object='obstacle',direction='left'):
        if object=='obstacle':
            found = False
            moved = 0
            while moved < 3:
                ignore = False
                self.get_frame()
                if self.rovio_found: return 1  # found rovio
                obs_found = self.obs_found
                if obs_found:
                    _,_,json = self.get_ref(self.obs_json)
                    obs_dist_btm, obs_dist_center = self.calc_dist(json=json, object=object)
                    print(obs_dist_btm, obs_dist_center)
                    if abs(json['ref_right'][0]-json['ref_left'][0]) < self.screen_width/10:
                        obs_found = False
                    elif obs_dist_center < 50:
                        found = True
                    else:
                        ignore = True
                if found and (not obs_found or ignore):    break
                # move left once, right once
                if self.moving_direction: self.move.move_left_straight()
                else: self.move.move_right_straight()
                self.move.move_backward_less()
                moved += 1
            return 0
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### moving\(\)

Moving around the game area, on DIRECTION \( left or right \), one time left, one time right.

{% code-tabs %}
{% code-tabs-item title="search.py" %}
```python
def moving(self):
        self.moving_direction = True    # True = left, False = right
        get_nearest = True
        while True:
            self.lost = False
            self.search(partial=3, get_nearest=get_nearest)
            if self.rovio_found and not self.lost:
                print('Aww yeah')
                self.chase()
                if self.lost:
                    continue
                print("WAITING FOR INPUT~~~~~")
                a = input()
            get_nearest = False
```
{% endcode-tabs-item %}
{% endcode-tabs %}



