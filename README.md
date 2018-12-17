# int-3dhuman-I1
Hey there, this is the implementation of **I1** (heatmap loss + integral loss) in [this paper](https://arxiv.org/pdf/1711.08229.pdf). Training is on H36M *exclusively* with 2-stage [*coarse to fine*](https://arxiv.org/pdf/1611.07828.pdf) [Hourglass](https://arxiv.org/pdf/1603.06937.pdf). Kindly jump to **Training procedure** below if you are not using C++.

----
## Source code & Installation
Please refer to the **"caffe_code/"** folder and **"Installation"** section in readme of [this repo](https://github.com/strawberryfg/c2f-3dhm-human-caffe)

----
## Configuration
This is for **Caffe** Ubuntu with 12 GB Titan card. Will release a windows version w/ batch size = 1 later.

----
## Data
Same as [**"data/"**](https://github.com/strawberryfg/c2f-3dhm-human-caffe/tree/master/data). Note this is different from commonly used data like [effective baseline](https://github.com/una-dinosauria/3d-pose-baseline). Major difference is my training(testing) set has 1559571(548999) images. 	

---
## Training procedure
*First off*, **THERE IS NO GURANTEE THAT OTHER TRAINING STRATEGY WOULD NOT YIELD SAME/SIMILAR OR BETTER RESULTS. THIS IS JUST FOR REFERENCE.**

*Having said that*, **I WOULD BE MORE THAN HAPPY TO KNOW IF ANYONE HAS A MORE ELEGANT WAY TO TRAIN THIS *WACKY* STUFF. MUCH APPRECIATED. THANKS!** 

*Back to our topic*,

The starting point is MPII pretrained model. See head of [training section](https://github.com/strawberryfg/c2f-3dhm-human-caffe) .

**Note 1 [*heatmap2 init std*]:** The init of layer "heatmap2" (which reduces dimension to generate final 3d heatmap output) is gaussian with standard deviation of *a hyper param*.

**Note 2 [*loss*]:** Adaptive weight balancing is employed. Found [*this paper*](https://arxiv.org/pdf/1707.04822.pdf) several weeks after writing the [*AdaptiveWeightEucLossLayer*](https://github.com/strawberryfg/c2f-3dhm-human-caffe/blob/master/caffe_code/src/caffe/layers/Operations/adaptive_weight_euc_loss_layer.cpp)

**Note 3 [*heatmap2_flat_scale*]:** Before softmax normalization, hereinafter *a scale* **30.0** is multiplied by 3d heatmap output for numerical reason. The semantic of heatmap no longer preserves any more, as alluded in [*this issue*](https://github.com/JimmySuen/integral-human-pose/issues/8).

1. **Adaptive H1** adaptively computes gradient magnitude of 2D/3D heatmap w.r.t neuron, and tries to **balance gradients flowing from 2D and 3D heatmap**. Search *AdaptiveWeightEucLoss* in [this pdf](https://github.com/strawberryfg/c2f-3dhm-human-caffe/blob/master/caffe_code/code.pdf) for expatiation.

The following is sorted by d2 (*depth dimension of 3d heatmap*) in increasing order.

  
- **d2 = 4**
  ```
  cd training/d2=4
  $CAFFE_ROOT/build/tools/caffe train --solver=solver_d4_ada.prototxt --weights=improved-hourglass_iter_640000.caffemodel
  ```
  
  | d2 | lr   |  "heatmap2" init std  | loss | Caffe Model  | Solver State |
  |:-:|:-:|:-:|:-:|:-:|:-:|
  | 4     | 2.5e-5 | 0.01   | Adaptive H1 | [net_iter_160533.caffemodel](https://drive.google.com/open?id=1LLTM2Ak4AHbdiZQZrsGhyXucDQOO313k) | [net_iter_160533.solverstate](https://drive.google.com/open?id=1Pyfocolp4gGg58ls-o6y30x9wX5Fembx) |

- **d2 = 8**
  - **Adaptive I1**
  ```
  cd ../../training/d2=8
  $CAFFE_ROOT/build/tools/caffe train --solver=solver_d8_ada.prototxt --snapshot=net_iter_160533.solverstate
  ```