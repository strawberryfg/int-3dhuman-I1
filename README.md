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
  
  | Method |d2   |  MPJPE(mm)  | Caffe Model  | Solver State |
|:-:|:-:|:-:|:-:|:-:|
| Mine     | 64 | [67.1]    | [Google Drive (net_iter_720929.caffemodel)] | [Google Drive (net_iter_720929.solverstate)]|
