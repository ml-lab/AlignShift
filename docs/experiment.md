# AlignShiftConv

AlignShift:Bridging the Gap of Imaging Thickness in 3D Anisotropic Volumes ([arXiv](https://arxiv.org/abs/?))


## Key contributions

* 
*
*

## Code structure

* ``convs``
  the core implementation of AlignShift convolution and TSM convolution, including the operators, models, and 2D-to-3D/AlignShift/TSM model converters. 
  * ``operators``: include AlignShiftConv, TSMConv.
  * ``converters``: include converters which convert 2D models to 3dConv/AlignShiftConv/TSMConv counterparts.
  * ``models``: Native AlignShift/TSM models. 
* ``experiments`` 
  the experiment code is base on [mmdetection](https://github.com/open-mmlab/mmdetection)
,this directory consists of compounents used in mmdetection.

## Convert a 2D model into 3D with a single line of code

```python
import Converter
import torchvision
from convs import AlignShiftConv
# m is a standard pytorch model
m = torchvision.models.resnet18(True)
alignshift_conv_cfg = dict(conv_type=AlignShiftConv, 
                          n_fold=8, 
                          alignshift=True, 
                          inplace=True,
                          ref_spacing=0.2, 
                          shift_padding_zero=True)
m = Converter(m, 
              alignshift_conv_cfg, 
              additional_forward_fts=['thickness'], 
              skip_first_conv=True, 
              first_conv_input_channles=1)
# after converted, m is using AlignShiftConv and capable of processing 3D volumes
x = torch.rand(batch_size, in_channels, D, H, W)
thickness = torch.rand(batch_size, 1)
out = m(x, thickness)
```

## Usage of AlignShiftConv/TSMConv operators

```python
from convs.operators import AlignShiftConv, TSMConv
x = torch.rand(batch_size, 3, D, H, W)
thickness = torch.rand(batch_size, 1)
# AlignShiftConv to process 3D volumnes
conv = AlignShiftConv(in_channels=3, out_channels=10, kernel_size=3, padding=1, n_fold=8, alignshift=True, ref_thickness=2.0)
out = conv(x, thickness)
# TSMConv to process 3D volumnes
conv = TSMConv(in_channels=3, out_channels=10, kernel_size=3, padding=1, n_fold=8, tsm=True)
out = conv(x)
```

## Usage of native  AlignShiftConv/TSMConv models

```python
from convs.models import DenseNetCustomTrunc3dAlign, DenseNetCustomTrunc3dTSM
net = DenseNetCustomTrunc3dAlign(num_classes=3)
B, C_in, D, H, W = (1, 3, 7, 256, 256)
input_3d = torch.rand(B, C_in, D, H, W)
thickness = torch.rand(batch_size, 1)
output_3d = net(input_3d, thickness)
```

## How to run the experiments

* Training
  ```bash
  ./experiments/dist_train.sh ${mmdetection script} ${dist training GPUS}
  ```

  * Train AlignShiftConv models 
  ```bash
  ./experiments/dist_train.sh ./experiments/mconfig/densenet_align.py 2
  ```

  * Train TSMConv models 
  ```bash
  ./experiments/dist_train.sh ./experiments/mconfig/densenet_tsm.py 2
  ```