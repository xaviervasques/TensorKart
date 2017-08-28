TensorKart
==========

self-driving MarioKart with TensorFlow

Driving a new (untrained) section of the Royal Raceway:

![RoyalRaceway.gif](https://media.giphy.com/media/1435VvCosVezQY/giphy.gif)

Driving Luigi Raceway:

[![LuigiRacewayVideo](/screenshots/luigi_raceway.png?raw=true)](https://youtu.be/vrccd3yeXnc)

The model was trained with:
* 4 races on Luigi Raceway
* 2 races on Kalimari Desert
* 2 races on Mario Raceway

With even a small training set the model is sometimes able to generalize to a new track (Royal Raceway seen above).

On IBM Power Systems
--------------------

Install IBM PowerAI for IBM Power Systems:
	https://www.ibm.com/bs-en/marketplace/deep-learning-platform/details

PowerAI is a software distribution for machine learning running on the Enterprise Platform for AI: IBM Power Systems. IBM PowerAI includes most ML/DL frameworks built with optimized versions of leading frameworks including: Caffe-bvlc, Caffe-ibm, Caffe-nv, Chainer, DIGITS, Torch, Theano, and TensorFlow.

Hardware requirements
IBM Power Systems S822LC for High Performance Computing (8335-GTB) is the hardware platform built for PowerAI.
•	2 POWER8 CPUs
•	128GB of memory, or greater, recommended
•	NVIDIA® Tesla® P100 with NVLink GPUs strongly recommended
•	NVIDIA NVLink interface to Tesla GPUs strongly recommended

Software requirements
Ubuntu 16.04 LTS

Dependencies
------------
Install through apt-get install:

* mupen64plus 
* python-dev
* libandroid-properties1 
* gtk2.0 
* build-essential libgtk2.0-dev 
* gstreamer1.0 libblas-dev 
* liblapack-dev 
* gfortran 
* libhdf5-dev 
* build-essential libssl-dev 
* libffi-dev python-dev 
* libgtk2.0-dev 
* libgtk-3-dev 
* libjpeg-dev 
* libtiff-dev 
* libsdl1.2-dev 
* libgstreamer-plugins-base0.10-dev 
* libnotify-dev 
* freeglut3 
* freeglut3-dev 
* libsm-dev 
* libwebkitgtk-dev 
* libwebkitgtk-3.0-dev 
* libgstreamer-plugins-base1.0-dev 
* joystick 
* libfreetype6
* libfreetype6-dev
* libpng-dev
* zlib1g-dev
* g++
* nasm
* libjson-c2
* libjson-c-dev


Install through pip install:
* inputs 
* matplotlib 
* numpy 
* cython 
* scikit-image 
* termcolor 
* keras 
* h5py 
* pillow 
* pygame 
* wxPython

Install vglrun

The mupen64plus install: 

	#!/bin/bash
	mkdir mupen64plus-src && cd "$_" 
	git clone https://github.com/mupen64plus/mupen64plus-core
	git clone https://github.com/kevinhughes27/mupen64plus-input-bot
	cd mupen64plus-input-bot
	make all
	sudo make install

	cd gym-mupen64plus
	pip install -e .

For playstation controllers, add (before [Keyboard])in /usr/share/games/mupen64plus/InputAutoCfg.ini

	[Sony Computer Entertainment Wireless Controller]
	plugged = True
	plugin = 2
	mouse = False
	AnalogDeadzone = 4096,4096
	AnalogPeak = 32767,32767
	DPad R = hat(0 Right)
	DPad L = hat(0 Left)
	DPad D = hat(0 Down)
	DPad U = hat(0 Up)
	Start = button(9)
	Z Trig = button(6)
	B Button = button(0)
	A Button = button(1)
	C Button R = axis(2+)
	C Button L = axis(2-)
	C Button D = axis(5+)
	C Button U = axis(5-)
	R Trig = button(5)
	L Trig = button(4)
	Mempack switch =
	Rumblepak switch =
	X Axis = axis(0-, 0+)
	Y Axis = axis(1-, 1+)

If you use Xbox controller or others, modify in utils.py: 

	def _monitor_controller(self):
        	while True:
            	events = get_gamepad()
            	for event in events:
                	if event.code == 'ABS_Y':
                    		self.LeftJoystickY = ((event.state-125) / XboxController.MAX_JOY_VAL)*250 # normalize between -1 and 1
                	elif event.code == 'ABS_X':
                   		 self.LeftJoystickX = ((event.state-125) / XboxController.MAX_JOY_VAL)*250 # normalize between -1 and 1
                    
by 

	def _monitor_controller(self):
       		while True:
            	events = get_gamepad()
            	for event in events:
                	if event.code == 'ABS_Y':
                    		self.LeftJoystickY = (event.state / XboxController.MAX_JOY_VAL)*250 # normalize between -1 and 1
                	elif event.code == 'ABS_X':
                    		self.LeftJoystickX = (event.state / XboxController.MAX_JOY_VAL)*250 # normalize between -1 and 1

The goal is to normalize the inputs from record.py to -1 and 1

For TensorFlow:

Create a file ~/.matplotlib/matplotlibrc there and add the following code: 

backend: TkAgg


Recording Samples
-----------------
1. Start your emulator program (`mupen64plus`) and run Mario Kart 64
2. Make sure you have a joystick connected and that `mupen64plus` is using the sdl input plugin
3. Run `record.py`
4. Make sure the graph responds to joystick input.
5. Position the emulator window so that the image is captured by the program (top left corner)
6. Press record and play through a level. You can trim some images off the front and back of the data you collect afterwards (by removing lines in `data.csv`).

![record](/screenshots/record_setup.png?raw=true)

Notes
- the GUI will stop updating while recording to avoid any slow downs.
- double check the samples, sometimes the screenshot is the desktop instead. Remove the appropriate lines from the `data.csv` file


Viewing Samples
---------------
Run `python utils.py viewer samples/luigi_raceway` to view the samples


Preparing Training Data
-----------------------
Run `python utils.py prepare samples/*` with an array of sample directories to build an `X` and `y` matrix for training. (zsh will expand samples/* to all the directories. Passing a glob directly also works)

`X` is a 3-Dimensional array of images

`y` is the expected joystick ouput as an array:

```
  [0] joystick x axis
  [1] joystick y axis
  [2] button a
  [3] button b
  [4] button rb
```


Training
--------
The `train.py` program will train a model using Google's TensorFlow framework and cuDNN for GPU acceleration. Training can take a while (~1 hour) depending on how much data you are training with and your system specs. The program will save the model to disk when it is done.

You can play with the following variables in train.py to improve the model:

create_model(keep_prob = 0.8)

Training loop variables
    epochs = 100
    batch_size = 50
    
The validation split: validation_split=0.1 (increase the number of validation data set)



Play
----
The `play.py` program will use the [`gym-mupen64plus`](https://github.com/bzier/gym-mupen64plus) environment to execute the trained agent against the MarioKart environment. The environment will provide the screenshots of the emulator. These images will be sent to the model to acquire the joystick command to send. The AI joystick commands can be overridden by holding the 'LB' button on the controller.

You can also adjust the controller (x100 works fine for me):

	int(joystick[0]*100),
        int(joystick[1]*100),
        int(round(joystick[2])),
        int(round(joystick[3])),
        int(round(joystick[4])),

Settings VirtualBox
-------------------
In virtualbox settings, in display, check “Enable 2D Video Acceleration”

