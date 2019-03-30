# accimage

### Modifications xvdp
* added optional kwarg `in_place=[False]` to img operations `.crop()` `.transpose()` `.resize()`.
  
  New default behaviour matches PIL.Image, i.e. operations return a copy. 
  This is necessary for torchvision.transforms.FiveCrop().
  
* added deep copy
`Image.copy()`

## Description
An accelerated Image loader and preprocessor leveraging [Intel
IPP](https://software.intel.com/en-us/intel-ipp) and [jpeg-turbo](https://libjpeg-turbo.org/).

accimage mimics the PIL API and can be used as a backend for
[`torchvision`](https://github.com/pytorch/vision).

Operations implemented:

- `Image.resize((width, height))`
- `Image.crop((left, upper, right, lower))`
- `Image.transpose(PIL.Image.FLIP_LEFT_RIGHT)`

Enable the torchvision accimage backend with:

```python
torchvision.set_image_backend('accimage')
```

## Installation
Original accimage is available on conda-forge, simply run the following to install
```
$ conda install -c conda-forge accimage
```
As of yet, the original instalation does operations in place, therefore breaks torchvision transformations that require copies: FiveCrop, TenCrop.


## Compilation
Dowload Intel IPP and jpeg-turbo; install. Ensure that include paths exist.
```
$ python setup.py install
```

## Comparative Benchmarking
opening a (399,899,3) jpeg from disk on a home computer with fast SSD

```python
import PIL
import cv2
import accimage
import numpy as np
import torchvision.transforms as transforms
```
1/ Simply opening file, PIL is faster. Pillow only fetches the headers, so if you just want to get width, size, std, mean, use pillow.
```python
%timeit PIL.Image.open(img)
Out[*]: 41.6 µs ± 67.8 ns per loop (mean ± std. dev. of 7 runs, 10000 loops each)

%timeit accimage.Image(img)
Out[*]: 1.13 ms ± 1.02 µs per loop (mean ± std. dev. of 7 runs, 1000 loops each)
```
2/ Opening and returning numpy array, accimage is faster
``` python
%timeit np.array(PIL.Image.open(img))
Out[*]: 4.28 ms ± 22.3 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)

%timeit cv2.cvtColor(cv2.imread(img), cv2.COLOR_BGR2RGB)
Out[*]: 2.65 ms ± 50.9 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)

def opennp(img):
    pic = accimage.Image(img)
    nppic = np.zeros([pic.channels, pic.height, pic.width], dtype=np.uint8)
    pic.copyto(nppic)
    return nppic
%timeit opennp(img)
Out[*]: 1.22 ms ± 4.47 µs per loop (mean ± std. dev. of 7 runs, 1000 loops each)
```
3/ Opening and converting to tensor, accimage is faster
```python
%timeit transforms.ToTensor()(PIL.Image.open(img))
Out[*]: 6.28 ms ± 174 µs per loop (mean ± std. dev. of 7 runs, 100 loops each

%timeit transforms.ToTensor()(cv2.cvtColor(cv2.imread(img), cv2.COLOR_BGR2RGB))
Out[*]: 4.59 ms ± 56.7 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)

%timeit transforms.ToTensor()(accimage.Image)
Out[*]: 1.67 ms ± 43.5 µs per loop (mean ± std. dev. of 7 runs, 1000 loops each)
```
4/ Basic operations: Cropping, accimage is faster
```python
%timeit PIL.Image.open(img).crop((20, 20, 40, 40))
Out[*]: 3.35 ms ± 16.6 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)

%timeit cv2.cvtColor(cv2.imread(img), cv2.COLOR_BGR2RGB)[20:40, 20:40]
Out[*]: 2.18 ms ± 12.6 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)

%timeit accimage.Image(img).crop((20, 20, 40, 40))
Out[*]: 1.4 ms ± 6.47 µs per loop (mean ± std. dev. of 7 runs, 1000 loops each)
```
5/ Basic operations: transforming to tensor and then cropping: 
overhead of accimate to tensor is smaller than numpy to tensor
```python
%timeit transforms.ToTensor()(cv2.cvtColor(cv2.imread(img), cv2.COLOR_BGR2RGB))[20:40, 20:40]
Out[*]: 4.59 ms ± 178 µs per loop (mean ± std. dev. of 7 runs, 100 loops each)

%timeit transforms.ToTensor()(accimage.Image(img))[...,20:40, 20:40]
Out[*]: 1.63 ms ± 27.1 µs per loop (mean ± std. dev. of 7 runs, 1000 loops each)
```
