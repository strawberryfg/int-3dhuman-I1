# int-3dhuman-I1
Hey there, this is the implementation of I1 (heatmap loss + integral loss) in this paper. Training is on H36M *exclusively* with 2-stage [coarse to fine](https://arxiv.org/pdf/1611.07828.pdf) Hourglass(https://arxiv.org/pdf/1603.06937.pdf). Simply jump to **Training procedure** below if you are not using C++.

----
## Source code & Installation
Please refer to the **"caffe_code/"** folder and **"Installation"** section in readme of [This repo](https://github.com/strawberryfg/c2f-3dhm-human-caffe)

----
## Configuration
This is for Caffe Ubuntu with 12 GB Titan card. Will release a windows version w/ batch size = 1 later.

----
## Data
Same as [**"data/"**](https://github.com/strawberryfg/c2f-3dhm-human-caffe/tree/master/data). Note this is different from commonly used data like [effective baseline](https://github.com/una-dinosauria/3d-pose-baseline). Major difference is my training(testing) set has 1559571(548999) images. 	

---
## Training procedure
*First off*, **THERE IS NO GURANTEE THAT OTHER TRAINING STRATEGY WOULD NOT YIELD SAME/SIMILAR OR BETTER RESULTS. THIS IS JUST FOR REFERENCE.**

The starting point is MPII pretrained model. See head of [training section](https://github.com/strawberryfg/c2f-3dhm-human-caffe) .

The following is sorted by d2 (*depth dimension of 3d heatmap*) in increaseing order.

**Note 1 [*heatmap2 init std*]:** The init of layer "heatmap2" (which reduces dimension to generate final 3d heatmap output) is gaussian with standard deviation of *a hyper param*.

**Note 2 [*loss*]:** 

1. **Adaptive H1** adaptively computes gradient magnitude of 2D/3D heatmap w.r.t neuron, and tries to **balance gradients flowing from 2D and 3D heatmap**. Search *AdaptiveWeightEucLoss* in [this pdf](https://github.com/strawberryfg/c2f-3dhm-human-caffe/blob/master/caffe_code/code.pdf) for expatiation.

  
- **d2 = 4**
  ```
  cd training/d2=4
  $CAFFE_ROOT/build/tools/caffe train --solver=solver_d4_ada.prototxt --weights=improved-hourglass_iter_640000.caffemodel
  ```
  
  | d2 | lr   |  "heatmap2" init std  | loss | Caffe Model  | Solver State |
  |:-:|:-:|:-:|:-:|:-:|:-:|
  | 4     | 2.5e-5 | 0.01   | Adaptive H1 | [net_iter_160533.caffemodel](https://drive.google.com/open?id=1LLTM2Ak4AHbdiZQZrsGhyXucDQOO313k) | [net_iter_160533.solverstate](https://drive.google.com/open?id=1Pyfocolp4gGg58ls-o6y30x9wX5Fembx) |

  - **d2 = 8**
  ```
  cd ../../training/d2=8
  $CAFFE_ROOT/build/tools/caffe train --solver=solver_d4_ada.prototxt --weights=improved-hourglass_iter_640000.caffemodel
  ```