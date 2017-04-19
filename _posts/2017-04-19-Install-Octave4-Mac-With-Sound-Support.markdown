---
layout: post
title:  "Install Octave4 On Mac with Sound Support"
date:   2017-04-19 15:14:54
categories: jekyll
---

* content
{:toc}

[Octave](https://www.gnu.org/software/octave/)是machine learning大神N.g推荐学习工具,本文主要记录如何在Mac OS上安装支持[音频(libsndfile,soundport)](https://github.com/erikd/libsndfile)读写处理的Octave版本.


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


---
