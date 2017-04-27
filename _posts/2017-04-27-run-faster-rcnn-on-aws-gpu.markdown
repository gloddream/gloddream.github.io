---
layout: post
title:  "run faster rcnn on aws with gpu"
date:   2017-04-19 15:14:54
categories: jekyll
---

* content
{:toc}

[faster rcnn](https://github.com/rbgirshick/py-faster-rcnn) :towards real-time object detection with region proposal networks 


---

## Install dependencies

	

---

### Preparation
---


---

## Install Octave
	
	brew edit Octave
	brew install octave --build-from-source --with-portaudio --with-sndfile
	
---

### Test it

	octave --no-gui
---

## Some trouble shooting

**error1**
`bdc1394 error: Failed to initialize libdc1394`
	sudo ln /dev/null /dev/raw1394

**error2**
`Gdk-CRITICAL **: gdk_cursor_new_for_display: assertion 'GDK_IS_DISPLAY (display)' Failed:`
Edit: {caffe_root}/python/caffe/__init__.py
Add the following lines to the top:
	import matplotlib
	matplotlib.use('Agg')

---
