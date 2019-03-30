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
