# Setting up Anaconda (or Miniconda) Locally

## But Why?
I run into a number of problems installing and running tensorflow in the preinstalled python environment on Kubrick. 
To name a few:
+ Just installing `tensorflow` with `pip install tensorflow` was straight forward. But this version did not 
make use of GPUs on Kubrick.
+ So, I tried `tensorflow-gpu` with `pip install tensorflow-gpu`. But, this is where the problems started. 
This installed tensorflow-gpu, however, I ran into a number of issues with linking `libcublas`, `cudatoolkit`, and `cudnn`. 
This is what I got when doing `import tensorflow`

```sh
---------------------------------------------------------------------------
ImportError                               Traceback (most recent call last)
~/miniconda3/lib/python3.7/site-packages/tensorflow/python/pywrap_tensorflow.py in <module>
     57
---> 58   from tensorflow.python.pywrap_tensorflow_internal import *
     59   from tensorflow.python.pywrap_tensorflow_internal import __version__

~/miniconda3/lib/python3.7/site-packages/tensorflow/python/pywrap_tensorflow_internal.py in <module>
     27             return _mod
---> 28     _pywrap_tensorflow_internal = swig_import_helper()
     29     del swig_import_helper

~/miniconda3/lib/python3.7/site-packages/tensorflow/python/pywrap_tensorflow_internal.py in swig_import_helper()
     23             try:
---> 24                 _mod = imp.load_module('_pywrap_tensorflow_internal', fp, pathname, description)
     25             finally:

~/miniconda3/lib/python3.7/imp.py in load_module(name, file, filename, details)
    241         else:
--> 242             return load_dynamic(name, filename, file)
    243     elif type_ == PKG_DIRECTORY:

~/miniconda3/lib/python3.7/imp.py in load_dynamic(name, path, file)
    341             name=name, loader=loader, origin=path)
--> 342         return _load(spec)
    343

ImportError: libcublas.so.10.0: cannot open shared object file: No such file or directory

During handling of the above exception, another exception occurred:

ImportError                               Traceback (most recent call last)
<ipython-input-1-64156d691fe5> in <module>
----> 1 import tensorflow as tf

~/miniconda3/lib/python3.7/site-packages/tensorflow/__init__.py in <module>
     22
     23 # pylint: disable=g-bad-import-order
---> 24 from tensorflow.python import pywrap_tensorflow  # pylint: disable=unused-import
     25
     26 from tensorflow._api.v1 import app

~/miniconda3/lib/python3.7/site-packages/tensorflow/python/__init__.py in <module>
     47 import numpy as np
     48
---> 49 from tensorflow.python import pywrap_tensorflow
     50
     51 # Protocol buffers

~/miniconda3/lib/python3.7/site-packages/tensorflow/python/pywrap_tensorflow.py in <module>
     72 for some common reasons and solutions.  Include the entire stack trace
     73 above this error message when asking for help.""" % traceback.format_exc()
---> 74   raise ImportError(msg)
     75
     76 # pylint: enable=wildcard-import,g-import-not-at-top,unused-import,line-too-long

ImportError: Traceback (most recent call last):
  File "/home/user/miniconda3/lib/python3.7/site-packages/tensorflow/python/pywrap_tensorflow.py", line 58, in <module>
    from tensorflow.python.pywrap_tensorflow_internal import *
  File "/home/user/miniconda3/lib/python3.7/site-packages/tensorflow/python/pywrap_tensorflow_internal.py", line 28, in <module>
    _pywrap_tensorflow_internal = swig_import_helper()
  File "/home/user/miniconda3/lib/python3.7/site-packages/tensorflow/python/pywrap_tensorflow_internal.py", line 24, in swig_import_helper
    _mod = imp.load_module('_pywrap_tensorflow_internal', fp, pathname, description)
  File "/home/user/miniconda3/lib/python3.7/imp.py", line 242, in load_module
    return load_dynamic(name, filename, file)
  File "/home/user/miniconda3/lib/python3.7/imp.py", line 342, in load_dynamic
    return _load(spec)
ImportError: libcublas.so.10.0: cannot open shared object file: No such file or directory


Failed to load the native TensorFlow runtime.

See https://www.tensorflow.org/install/errors

for some common reasons and solutions.  Include the entire stack trace
above this error message when asking for help.
```
+ So, I tried to manually install `cudatoolkit` and `cudnn`. But, it turns out the versions don't match those required by `tensorflow`. Specifically, we get `cudatoolkit 10.1.168-0` but we need `10.0.130-0` and `cudnn 7.6.0-cuda10.1_0` and we need `7.6.0-cuda10.0_0`
+ I must admit, I may be going about wrongly with this, but google searches didn't produce anything better. But, this seems to be a common [issue](https://github.com/tensorflow/tensorflow/issues/26182)

## Alternative setup

Well, just use miniconda. 

1. From the root directory, make a folder for downloads 
```sh
[user@kubrick:~] mkdir Downloads && cd Downloads
```

2. Download the latest version of Miniconda from [here](https://docs.conda.io/en/latest/miniconda.html)
```sh
[user@kubrick:~/Downloads] wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
```

3. Make it executable
```sh
[user@kubrick:~/Downloads] chmod +x Miniconda3-latest-Linux-x86_64.sh
```

4. Install Miniconda
```sh
[user@kubrick:~/Downloads] ./Miniconda3-latest-Linux-x86_64.sh
...
[user@kubrick:~/Downloads] which python 
/home/user/miniconda3/bin/python
[user@kubrick:~/Downloads] cd ~
```

5. Install Tensorflow, Keras, ipython, and jupyter all using conda
```sh
[user@kubrick:~] conda install tensorflow-gpu
[user@kubrick:~] conda install keras
[user@kubrick:~] conda install ipython
[user@kubrick:~] conda install jupyter
```

6. Check to see if Keras and Tensorflow works
```sh
[user@kubrick:~] python -c "import keras as kr; print(kr.__version__)"
Using TensorFlow backend.
2.2.4
[user@kubrick:~] python -c "import tensorflow as tf; print(tf.__version__)"
1.13.1
```
