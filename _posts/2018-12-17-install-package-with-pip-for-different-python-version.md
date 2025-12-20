---
title: Install Package with pip for different Python version(Mac)
categories:
- Notes
tags:
- python
- pip
- Mac
- install
- package
description: installing packages for different versions of python with pip
date: 2018-12-17
author_profile: true
classes: wide
---

Sometime you will see errors saying packages are missing in a python version, the reason might be the package is not installed correctly using `pip install` or `pip3 install`, here is how to install the package for different python versions.

```
# use the system default python version
python -m pip install lxml

# specify which python to install the package
python3.6 -m pip install pandas
```

Now you can run either `python test.py` or `python3 test.py`.
