# int-3dhuman-I1
Hey there, this is the implementation of **I1** (heatmap loss + integral loss) in [this paper](https://arxiv.org/pdf/1711.08229.pdf). Training is on H36M *exclusively* with 2-stage [*coarse to fine*](https://arxiv.org/pdf/1611.07828.pdf) [Hourglass](https://arxiv.org/pdf/1603.06937.pdf). Kindly jump to **Training procedure** below if you are not using C++.

----
## Source code & Installation
Please refer to the **"caffe_code/"** folder and **"Installation"** section in readme of [this repo](https://github.com/strawberryfg/c2f-3dhm-human-caffe)

----
## Configuration
This is for **Caffe** Ubuntu with 12 GB Titan card.

----
## Data
Same as [**"data/"**](https://github.com/strawberryfg/c2f-3dhm-human-caffe/tree/master/data). Note this is different from commonly used data like [effective baseline](https://github.com/una-dinosauria/3d-pose-baseline). Major difference is my training(testing) set has 1559571(548999) images. 	

----
## MPJPE Performance 
  | d2 | MPJPE(mm) of *this repo (I1)*  |  Caffe Model  | *(For reference)* MPJPE(mm) of *corresponding* [*H1 repo*](https://github.com/strawberryfg/c2f-3dhm-human-caffe)|
  |:-:|:-:|:-:|:-:|
  | 16     | [71.0](https://github.com/strawberryfg/int-3dhuman-I1/blob/master/figs/test_d16_full.png) | [net_iter_526322.caffemodel](https://drive.google.com/open?id=1RoIHw9KGQglohC7wGPSOAkwwB-veWjY1) | [73.6](https://github.com/strawberryfg/c2f-3dhm-human-caffe/blob/master/figs/test_d16_full.png) |
  | 32     | [55.9](https://github.com/strawberryfg/int-3dhuman-I1/blob/master/figs/test_d32_full.png) | [net_iter_567946.caffemodel](https://drive.google.com/open?id=1M4yQT-C4cu2MmylRKewP1ussQ_AKGw-n)  | [68.6](https://github.com/strawberryfg/c2f-3dhm-human-caffe/blob/master/figs/test_d32_full.png) |
  | 64     | [65.9](https://github.com/strawberryfg/int-3dhuman-I1/blob/master/figs/test_d64_full.png) | [net_iter_671782.caffemodel](https://drive.google.com/open?id=1tCDiVq2Uzv9eQePTalNB0xMqwWyynkQU)   | [67.1](https://github.com/strawberryfg/c2f-3dhm-human-caffe/blob/master/figs/test_d64_full.png) |  

*Note that* in order to restrain one from tuning loss weight of 2D/3D HM/integral to the best of one's ability, [adaptive euclidean loss weight balancing technique](https://github.com/strawberryfg/c2f-3dhm-human-caffe/blob/master/caffe_code/src/caffe/layers/Operations/adaptive_weight_euc_loss_layer.cpp) (as detailed below) is initialized insofar as it does not degrade performance.

*I have a deep [suspicion](http://selfpace.uconn.edu/class/percep/DescartesMeditations.pdf)* concerning the bearing of different training technique on final number. And so I would say what matters most is the methodology itself rather than the superiority or inferiority of number.
  
---
## Training procedure
*First off*, **THERE IS NO GUARANTEE THAT OTHER TRAINING STRATEGY WOULD NOT YIELD SAME/SIMILAR OR BETTER RESULTS. THIS IS JUST FOR REFERENCE.**

*Having said that*, **I WOULD BE MORE THAN HAPPY TO KNOW IF ANYONE HAS A MORE ELEGANT WAY TO TRAIN THIS *WACKY* STUFF. MUCH APPRECIATED. THANKS!** 

*Back to our topic*,

The starting point is MPII pretrained model. See head of [training section](https://github.com/strawberryfg/c2f-3dhm-human-caffe) .

**Note 1 [*heatmap2 init std*]:** The init of layer "heatmap2" (which reduces dimension to generate final 3d heatmap output) is gaussian with standard deviation of *a hyper param*.

**Note 2 [*heatmap2_flat_scale*]:** Before softmax normalization, hereinafter *a scale* **30.0** is multiplied by 3d heatmap output for numerical reason. The semantic of heatmap no longer preserves any more, as alluded in [*this issue*](https://github.com/JimmySuen/integral-human-pose/issues/8).
Otherwise I have no idea how to train **I1**. **CAN ANYONE ENLIGHTEN ME?**

**Note 3 [*loss*]:** Adaptive weight balancing is employed. Found [*this paper*](https://arxiv.org/pdf/1707.04822.pdf) several weeks after writing the [*AdaptiveWeightEucLossLayer*](https://github.com/strawberryfg/c2f-3dhm-human-caffe/blob/master/caffe_code/src/caffe/layers/Operations/adaptive_weight_euc_loss_layer.cpp)

1. **Adaptive H1** adaptively computes gradient magnitude of 2D/3D heatmap w.r.t neuron, and tries to **balance gradients flowing from 2D and 3D heatmap**. Search *AdaptiveWeightEucLoss* in [this pdf](https://github.com/strawberryfg/c2f-3dhm-human-caffe/blob/master/caffe_code/code.pdf) for expatiation.

2. **Adaptive I1** adaptively balances the gradient of 2D/3D heatmap/integral loss w.r.t neuron, restraining one from tuning weight ratio between *heatmap euclidean loss* and *integral regression loss*.

3. **Manual** sets loss ratio of heatmap:integral manually after **Adaptive I1** *warm-up*.

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
  
  | d2 | lr   |  "heatmap2" init std  | loss | Caffe Model  | Solver State |
  |:-:|:-:|:-:|:-:|:-:|:-:|
  | 8     | 2.5e-5 | 0.01   | Adaptive I1 | [net_iter_265747.caffemodel](https://drive.google.com/open?id=1kTNmfoXtIVAmYJXRAGtzRBK6MFB-EPg9) | [net_iter_265747.solverstate](https://drive.google.com/open?id=1koqQ8SrOclTbEcNXQeCaqsc7gv79rl8d)|
  
  - **Manual**
  ```
  $CAFFE_ROOT/build/tools/caffe train --solver=solver_d8_ada_manual.prototxt --snapshot=net_iter_265747.solverstate
  ```
  | d2 | lr   |  loss ratio of 2D HM:3D HM:integral   | loss | Caffe Model  | Solver State |
  |:-:|:-:|:-:|:-:|:-:|:-:|
  | 8     | **5e-6** | 1:0.1:1   | Manual | [net_iter_350433.caffemodel](https://drive.google.com/open?id=1WQTbk1gCHOqK1FsnTVw9I7k687QqQr7A)| [net_iter_350433.solverstate](https://drive.google.com/open?id=1YpPs6Fx08KGbzpLUbopOQ3puho5im33M)|
  
  [net_iter_350433.caffemodel](https://drive.google.com/open?id=1WQTbk1gCHOqK1FsnTVw9I7k687QqQr7A) train around **71 *mm***, test around **83 *mm***.
- **d2 = 16**
  - **Adaptive I1**
  ```
  cd ../../training/d2=16
  $CAFFE_ROOT/build/tools/caffe train --solver=solver_16_ada.prototxt --snapshot=net_iter_350433.solverstate
  ```
  | d2 | lr   |  "heatmap2" init std   | loss | Caffe Model  | Solver State |
  |:-:|:-:|:-:|:-:|:-:|:-:|
  | 16     | 2.5e-5 | **0.002**   | Adaptive I1 | [net_iter_434657.caffemodel](https://drive.google.com/open?id=1qBbSrbkzlK9neE7DmaZDVwFkprr0u3p7)| [net_iter_434657.solverstate](https://drive.google.com/open?id=1JwHjhVwt3ivumKunTagSqkHx5pJb0LoO)|
  
  [net_iter_434657.caffemodel](https://drive.google.com/open?id=1qBbSrbkzlK9neE7DmaZDVwFkprr0u3p7) train around **50 *mm***, test around **73 *mm***.
  
  - **Manual** 
  ```
  $CAFFE_ROOT/build/tools/caffe train --solver=solver_16_ada_manual.prototxt --snapshot=net_iter_434657.solverstate
  ```
  | d2 | lr   |  loss ratio of 2D HM:3D HM:integral   | loss | Caffe Model  | Solver State |
  |:-:|:-:|:-:|:-:|:-:|:-:|
  | 16     | **5e-6** | 1:0.3:1   | Manual | [net_iter_526322.caffemodel](https://drive.google.com/open?id=1RoIHw9KGQglohC7wGPSOAkwwB-veWjY1) | [net_iter_526322.solverstate](https://drive.google.com/open?id=1MkLjLGUmELZ8nFywI2R7PnHoF8cYkfL2)|
 
  [net_iter_526322.caffemodel](https://drive.google.com/open?id=1RoIHw9KGQglohC7wGPSOAkwwB-veWjY1) train around **47 *mm***, test around [**71 *mm***](https://github.com/strawberryfg/int-3dhuman-I1/blob/master/figs/test_d16_full.png) (click [this](https://github.com/strawberryfg/int-3dhuman-I1/blob/master/figs/test_d16_full.png) for screenshot).
  
- **d2 = 32**
  - **Adaptive I1** 
  ```
  cd ../../training/d2=32
  $CAFFE_ROOT/build/tools/caffe train --solver=solver_32_ada.prototxt --snapshot=net_iter_526322.solverstate
  ```
  | d2 | lr   |  "heatmap2" init std   | loss | Caffe Model  | Solver State |
  |:-:|:-:|:-:|:-:|:-:|:-:|
  | 32     | 2.5e-5 | 0.002   | Adaptive I1 | [net_iter_567946.caffemodel](https://drive.google.com/open?id=1M4yQT-C4cu2MmylRKewP1ussQ_AKGw-n) | [net_iter_567946.solverstate](https://drive.google.com/open?id=1AFXPO7SondbiUYbbJZ5cBBu10vuJIQuS) |
  
  [net_iter_567946.caffemodel](https://drive.google.com/open?id=1M4yQT-C4cu2MmylRKewP1ussQ_AKGw-n) train around **49 *mm*** test around [**56 *mm***](https://github.com/strawberryfg/int-3dhuman-I1/blob/master/figs/test_d32_full.png) (click [this](https://github.com/strawberryfg/int-3dhuman-I1/blob/master/figs/test_d32_full.png) for screenshot).

- **d2 = 64**
  - **Adaptive I1**
  ```
  cd ../../training/d2=64
  $CAFFE_ROOT/build/tools/caffe train --solver=solver_64_ada.prototxt --snapshot=net_iter_567946.solverstate
  ```
  | d2 | lr   |  "heatmap2" init std   | loss | Caffe Model  | Solver State |
  |:-:|:-:|:-:|:-:|:-:|:-:|
  | 64     | 2.5e-5 | 0.002   | Adaptive I1 | [net_iter_600890.caffemodel](https://drive.google.com/open?id=185DC6Md-hv3Tibb842BuecnUjOz-cgV9) | [net_iter_600890.solverstate](https://drive.google.com/open?id=1MS-B4i6bCEqRIvCNhU9atAX4kGwvWPV1) |
  
  [net_iter_600890.caffemodel](https://drive.google.com/open?id=185DC6Md-hv3Tibb842BuecnUjOz-cgV9) train around **43 *mm***, test around **68 *mm***.
  
  - **Manual** 
  ```
  $CAFFE_ROOT/build/tools/caffe train --solver=solver_64_ada_manual.prototxt --snapshot=net_iter_600890.solverstate
  ```
  
  | d2 | lr   |  loss ratio of 2D HM:3D HM:integral   | loss | Caffe Model  | Solver State |
  |:-:|:-:|:-:|:-:|:-:|:-:|
  | 64     | **5e-6** | 1:0.08:1   | Manual | [net_iter_671782.caffemodel](https://drive.google.com/open?id=1tCDiVq2Uzv9eQePTalNB0xMqwWyynkQU) | [net_iter_671782.solverstate](https://drive.google.com/open?id=1EamXyx6zRt-PiC6UB1pjsKVYXAa8xjX5) |
 
  [net_iter_671782.caffemodel](https://drive.google.com/open?id=1tCDiVq2Uzv9eQePTalNB0xMqwWyynkQU) train around **40 *mm***, test around [**66 *mm***](https://github.com/strawberryfg/int-3dhuman-I1/blob/master/figs/test_d64_full.png) (click [this](https://github.com/strawberryfg/int-3dhuman-I1/blob/master/figs/test_d64_full.png) for screenshot).