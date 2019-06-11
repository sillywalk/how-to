# Preparing a development Environement on Kubrick

In this tutorial, I walk through how to setup the Kubrick cluster to start prototyping and deploying your tensorflow application.

# Contents
[1. Why use anaconda?](#1-why-use-anaconda)

[2. Setting up Anaconda](#2-Setting-up-Anaconda)

[3. Running a Remote Jupyter Notebook on host](#3-running-a-remote-jupyter-notebook-on-host)

## 1. Why use anaconda?
I ran into a number of problems installing and running tensorflow in the preinstalled python environment on Kubrick. 

To name a few:

+ Just installing `tensorflow` with `pip install tensorflow` was straight forward. But this version did not 
make use of GPUs on Kubrick.

+ So, I tried `tensorflow-gpu` with `pip install tensorflow-gpu`. But, this is where the problems started. 
This installed tensorflow-gpu, however, I ran into a number of issues with linking `libcublas`, `cudatoolkit`, and `cudnn`. 
This is what I got when doing `import tensorflow`:

```sh
% /usr/bin/python
Python 2.7.15rc1 (default, Nov 12 2018, 14:31:15)
[GCC 7.3.0] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import tensorflow
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/home/user/.local/lib/python2.7/site-packages/tensorflow/__init__.py", line 24, in <module>
    from tensorflow.python import pywrap_tensorflow  # pylint: disable=unused-import
  File "/home/user/.local/lib/python2.7/site-packages/tensorflow/python/__init__.py", line 49, in <module>
    from tensorflow.python import pywrap_tensorflow
  File "/home/user/.local/lib/python2.7/site-packages/tensorflow/python/pywrap_tensorflow.py", line 74, in <module>
    raise ImportError(msg)
ImportError: Traceback (most recent call last):
  File "/home/user/.local/lib/python2.7/site-packages/tensorflow/python/pywrap_tensorflow.py", line 58, in <module>
    from tensorflow.python.pywrap_tensorflow_internal import *
  File "/home/user/.local/lib/python2.7/site-packages/tensorflow/python/pywrap_tensorflow_internal.py", line 28, in <module>
    _pywrap_tensorflow_internal = swig_import_helper()
  File "/home/user/.local/lib/python2.7/site-packages/tensorflow/python/pywrap_tensorflow_internal.py", line 24, in swig_import_helper
    _mod = imp.load_module('_pywrap_tensorflow_internal', fp, pathname, description)
ImportError: libcublas.so.10.0: cannot open shared object file: No such file or directory


Failed to load the native TensorFlow runtime.

See https://www.tensorflow.org/install/errors

for some common reasons and solutions.  Include the entire stack trace
above this error message when asking for help.
```
+ So, I tried to manually install `cudatoolkit` and `cudnn`. But, it turns out (a) they are not in pip, (b) If we compile from source, the versions don't match those required by `tensorflow`. Specifically, we get `cudatoolkit 10.1.168-0` but it turns out we need `10.0.130-0` and we get `cudnn 7.6.0-cuda10.1_0` and we need `7.6.0-cuda10.0_0`
+ I must admit, I may be going about this wrongly, but google searches didn't produce anything better. And, this seems to be a commonly occuring [issue](https://github.com/tensorflow/tensorflow/issues/26182)

## 2. Setting up Anaconda

Well, just let's just use miniconda. It just works. Here's what we do:

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

## 3. Running a Remote Jupyter Notebook on host

This is a nice to have. Say you want to remotely connect to Kubrick and use jupyter notebook to develop on your local machine, there's a simple way to do that.

1. **[Both On Kubrick & Local]** First, install `jupyter` on *both* Kubrick and your local machine
```sh
[user@kubrick:~] conda install jupyter
```

2. **[On Kubrick]** You'll need to initialize jupyter configurations file. For this we need `jupyter notebook --generate-config`. This command will create the Jupyter folder (at `/home/user/.jupyter`) if necessary, and create notebook configuration file, `jupyter_notebook_config.py`, in this directory.
```sh
[user@kubrick:~/tf-notebooks] jupyter notebook --generate-config
```

3. **[On Kubrick]** The first time you log-in, the notebook server may ask you for a password. I recommend setting this up. Use `jupyter notebook password`. You'll be prompted for a username and password. Entering it will save a hashed password in your `/home/user/.jupyter/jupyter_notebook_config.json`.
```sh
[user@kubrick:~] jupyter notebook password
Enter password:  ****
Verify password: ****
[NotebookPasswordApp] Wrote hashed password to /Users/you/.jupyter/jupyter_notebook_config.json
```

4. **[On Kubrick]** Next, *on kubrick* navigate to the directory where you park all your notebooks. Then, run the following command. *Note: This terminal must be kept open.*
```sh
[user@kubrick:~] mkdir tf-notebooks && cd tf-notebooks
[user@kubrick:~/tf-notebooks] jupyter notebook --no-browser --port=8889
```

5. **[On Local machine]** Forward that port to a local port on *your machine*. For this, *on your local machine*, do the following
```sh
[localmachine: ~] ssh -N -f -L localhost:8888:localhost:8889 username@kubrick.cs.columbia.edu
```

6. **[On Local machine]** That's it. You're all set. Open up a browser and go to `localhost:8888`. You'll be asked for the password you setup, and then you'll land in the folder on *kubrick* with all the notebooks. Code away....


