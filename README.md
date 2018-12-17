# int-3dhuman-I1
Hello this is just a test 

----
## Source code & Installation
Please refer to the **"caffe_code/"** folder and **"Installation"** section in readme of [This repo](https://github.com/strawberryfg/c2f-3dhm-human-caffe)

----
## Configuration
This is for Ubuntu with 12 GB Titan card. Will release a windows version w/ batch size = 1 later.

----
## Data
Same as [**"data/"**](https://github.com/strawberryfg/c2f-3dhm-human-caffe/tree/master/data). Note this is different from commonly used data like [effective baseline](https://github.com/una-dinosauria/3d-pose-baseline). Major difference is my training(testing) set has 1559571(548999) images. 	

---
## Training procedure
The starting point is MPII pretrained model. See head of [training section](https://github.com/strawberryfg/c2f-3dhm-human-caffe) .

The following is sorted by d2 (*depth dimension of 3d heatmap*) in increaseing order.

- **d2 = 4**
  ```
  cd training/d2=4
  $CAFFE_ROOT/build/tools/caffe train --solver=solver_d4_ada.prototxt --weights=improved-hourglass_iter_640000.caffemodel
  ```
  
  | d2 | lr   |  "heatmap2" init std  | loss | Caffe Model  | Solver State |
  |:-:|:-:|:-:|:-:|:-:|:-:|
  | 4     | 2.5e-5 | 0.01   | Adaptive H1 | [net_iter_160533.caffemodel](https://drive.google.com/open?id=1LLTM2Ak4AHbdiZQZrsGhyXucDQOO313k) | [net_iter_160533.solverstate](https://drive.google.com/open?id=1Pyfocolp4gGg58ls-o6y30x9wX5Fembx) |
 