# Demo of a self-driving car steering model for Tensorport

This is a demo of a self-steering car model for the tensorport deep learning computation platform.
Supporting blog posts:
- https://medium.com/towards-data-science/what-i-learnt-building-a-simple-self-steering-car-in-tensorflow-c8d7cab6f6d
- https://medium.com/@malomarrec/how-to-write-distributed-tensorflow-code-with-an-example-on-tensorport-70bf3306adcb

## Run locally

1) Get the data [here](https://s3-us-west-2.amazonaws.com/tppublicdata/comma-train.zip)
Unzip the file, and create a tensorport dataset as explained [here](https://github.com/tensorport/mnist#12-deep-dive-into-dataset-setup).
2) Get the code by cloning this repo.

In the code we assume that the data tree is:

Documents/comma/data-repo

Documents/tensorport-self-driving-demo

You will need to change the code in main_tf (search for "snippet2") to update the
FLAGS to your local arborescence.


3) The tensorport CLI facilitates local development and debugging by emulating the
remote environment and simulating distributed tensorflow. Install it with:

```bash
$ pip install tensorport
```

Now from your code repo, run:

```bash
$ tport run --local --single-node
```
for single node training and:
```bash
$ tport run --local --distributed
```
for simulated multi-node training.

The tensorport CLI enables you to simulate the tensorport environment, so that
you can make sure the code will run on the matrix before actually launching it.

At the prompt, if you select "0 - Use a requirements file", the cli will create
a new empty environment and load the requirements specified in your requirements
file (`requirements.txt` in the demo), just as it would happen on tensorport. This
process can be a little long, so you can still use your local environment during
development, but we recommend testing on that tensorport-like environment at least
once before switching to  the matrix.

```bash
-> Requirements file not specified. You can
    0- Specify a requirements file
    1- Run without installing any requirments
    2- Use your current environemt (not recommended).
```



For help on the CLI, try:
```bash
$ tport help
```

or

```bash
$ tport <any_command> --help
```

Now that we've checked that the code works locally and in a simulated distributed
mode, let's push it to tensorport to get some real speed up!


## Run on tensorport

0) Login using `tport login`

1) Clone this repo [if you skipped part 1]

2) From the repo, run:
```bash
$ tport create project
Display name:
$ comma-demo
Description [Tensorport Project]:
$ Demo of the comma self steering car.
Project created successfully
Waiting for Gitlab Repository
Saving Tensorport Config File
Tensorport Remote is already added
Project comma-demo and Repository are ready
```

3) Your project has now been created on tensorport, we just need to push the code. Run:
``` bash
$ git push tensorport master
```

4) We will now upload the data using Git LFS.
You need to have git lfs installed, see [here](https://git-lfs.github.com/)

From the data repo run:

```bash
$ git init
$ git lfs install
$ git lfs track "*.h5"
$ git add .gitattributes
$ git add camera labels
$ git commit -m "first commit: add data"
```
Your repo is now ready to push to tensorport.

```bash
$ tport create dataset
 -> Choose a valid dataset name: comma-sample
Creating dataset (it may take up to a few minutes) comma ........
Dataset malo/comma-sample is created.
Matrix: https://tensorport.com/matrix/malo/comma-sample
```

Now push the data (this will take a moment):

```bash
$ git push tensorport master
```


5) We are now ready to start distributed training on tensorport. From the CLI (see <#TODO HERE> for the GUI version):
``` bash
$ tport create job
Display name:
$ first-comma
Description [Tensorport Job]:
$ Run the comma demo
Please select project you want to use or specify --project parameter
0 | comma-demo | comma-demo
1 | mnist
Please select project to use, or type 0 to use latest one: [0]:
$ 0
Project selected: comma-demo
Please select commit to use in a job, or type 0 to use latest one:
0 | #1e8262c318dde1d328883bf34ee22cafdb9149d0 | add missing requirements
1 | #8177d69b0c12eb580b02e2bca3123a991114e4c6 | removing suprefluous pygame use
2 | #4ac1b9397ba2d198f69988155ec83937c77d3065 | Updating requirements
3 | #6e609185d091d0e1be29e95e5c367cb6ea9f08bf | clean code
4 | #b96a793f1687ffc6dfae96bd27e6b07f607288c5 | Clean code and repo
...
$ 0
Commit selected: #1e8262c318dde1d328883bf34ee22cafdb9149d0
Please specify python module name to run [main]:
$ main_tf
Please specify python path []:
$
List of available instance types:
......................................
# | Name | CPU | GPU | Memory(GiB)
0 | c4.2xlarge | 1 | 0 | 10
Please select instance type:
$ 0
How many workers? [1]:
$ 3
How many PS replicas? [1]:
$ 1
Please specify requirements file: [requirements.txt]:
$
Do you want to add a dataset to the job? Y/n [n]: y
Please select dataset you want to use
0 | comma-public | Public comma repo for comma demo
$ 0
Please select commit to use for this dataset in a job, or type 0 to use latest one:
$ 0
Please specify the dataset mounting point: []:
$
Do you want to add another dataset to the job? Y/n [n]:
$ n
Sending Job Create Request
Job Created Successfully
Starting job
```bash
tport watch
```

6) Your job is now running on tensorport! You can access the matrix and view the
logs and tensorboard.
You can also use the CLI to track what is happening on the server. There is a
pretty cool command that just listens to the events on the paltform and displays
them in terminal.
Try: `tport watch`




## Dataset
The dataset consists of videos clips of variable size recorded at 20 Hz
with a camera mounted on the windshield of an Acura ILX 2016. In parallel to the videos
we also recorded some measurements such as car's speed, acceleration,
steering angle, GPS coordinates, gyroscope angles.

The dataset folder structure is the following:
```bash
+-- dataset
|   +-- camera
|   |   +-- 2016-04-21--14-48-08
|   |   ...
|   +-- labels
|   |   +-- 2016-04-21--14-48-08
|   |   ...
```

All the files come in hdf5 format and are named with the time they were recorded.
The camera dataset has shape `number_frames x 3 x 160 x 320` and `uint8` type.

See the original comma.ai repo for details.


## Requirements

[tensorport-0.8.9]
[tensorflow-1.0](https://github.com/tensorflow/tensorflow)  


## Credits
Author: Malo Marrec, malo@goodailab.com, (c) GoodAILab as specified in LICENSE

Data and data_reader.py -  https://github.com/commaai/research - Licensed as specified in LICENSE_COMMA
