---
description: Here's what you need for running the code on Rovio.
---

# Getting started

## Install Git

To install Git on your computer, follow this simple guide: [https://git-scm.com/book/en/v2/Getting-Started-Installing-Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)

#### Check if Git is installed

Open Terminal \(or Command Prompt\) and enter the following command:

```text
git --version
```

You should not get any errors.

## Fork the RovioSeeker repository

The code is available at: [https://github.com/noob-ja/SeekerRovio](https://github.com/noob-ja/SeekerRovio)

## Dependencies

* Python 3.x
* Numpy
* TensorFlow 1.x
* Keras 2.x
* OpenCV
* Beautiful Soup 4.x

## Connect to Rovio

{% hint style="warning" %}
If this is the first time you are connecting to Rovio, you may need to **reset** it before setting new configuration. 
{% endhint %}

### Reset Rovio

1. Turn on Rovio   .
2. Wait until the on button shows orange light   .
3. Turn off Rovio   .
4. Repeat step 1 - 3 _twice._
5. Stop when you see red light instead of orange   .
6. The power button LED should be blinking from red-orange-green repeatedly.
7. when green light then Rovio is reset successfully.

Once Rovio is switched on, you will be able to see its name under your computer's network connections list \(just like a WiFi network\). 

### Connect to Rovio using an Ad-Hoc connection on Windows

1. Turn on rovio
2. Open cmd    `> netsh wlan show network`
3. Look for **rovio\_wowwee** in the list

   {% hint style="info" %}
   If Rovio is not in the list, reset it.
   {% endhint %}

4. Open Network and Sharing Center
   1. Set up a new connection or network
   2. Manually connect to a wireless network
5. Type in the details
   1. Network name: ROVIO\_WOWWEE
   2. Security type: No authentication
   3. Uncheck "start connection automatically"
6. Open cmd    `> netsh wlan set profileparameter ROVIO_WOWWEE connectiontype=ibss`
7. Open cmd    `> netsh wlan connect ROVIO_WOWWEE`
8. Check if you are connected to ROVIO\_WOWWEE in your WiFi
9. Open web browser
10. Go to 192.168.10.18
11. yay!

## Run the code on Rovio

From the code directory, run `python main.py`

