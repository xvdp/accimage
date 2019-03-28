# accimage

[![Build status](https://travis-ci.org/pytorch/accimage.svg?branch=master)](https://travis-ci.org/pytorch/accimage)
[![Anaconda badge](https://anaconda.org/conda-forge/accimage/badges/version.svg)](https://anaconda.org/conda-forge/accimage)


An accelerated Image loader and preprocessor leveraging [Intel
IPP](https://software.intel.com/en-us/intel-ipp) and [jpeg-turbo](https://libjpeg-turbo.org/).

accimage mimics the PIL API and can be used as a backend for
[`torchvision`](https://github.com/pytorch/vision).

Operations implemented:

- `Image.resize((width, height))`
- `Image.crop((left, upper, right, lower))`
- `Image.transpose(PIL.Image.FLIP_LEFT_RIGHT)`

Modifications xvdp:
accimage does all operations in place,
- `Image.copy()`

Enable the torchvision accimage backend with:

```python
torchvision.set_image_backend('accimage')
```

## Installation

accimage is available on conda-forge, simply run the following to install

```
$ conda install -c conda-forge accimage
```
## Compilation
Dowload Intel IPP and jpeg-turbo; install. Ensure that include paths exist.
```
$ python setup.py install
```
