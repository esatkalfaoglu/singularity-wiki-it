# Singularity Setup

This document describes how to create a singularity container from a NGC docker image and how to install further requirements upon this image.

## Pulling NGC docker image and building with singularity

First, create folder under your home folder with name `containers`

```bash
$ cd $HOME 
$ mkdir containers
```

Then, build a Singularity container from NGC's docker repository [https://ngc.nvidia.com/catalog/containers/](https://ngc.nvidia.com/catalog/containers/). Find the appropriate tag for your needs and replace it with `nvcr.io/nvidia/pytorch:21.06-py3`. You can see the contents of containers from [container contents](https://docs.nvidia.com/deeplearning/frameworks/pytorch-release-notes/rel_21-07.html#rel_21-07) by selecting them from the left menu. 
Note: Do not forget to name your container (`pytorch2106.sig` as in the example below) accordingly with the tag version.

```bash
$ singularity build --sandbox containers/pytorch2106.sig docker://nvcr.io/nvidia/pytorch:21.06-py3
```

Note that `--sandbox` option will later allow us to update the container contents.

## Installing additional requirements upon the NGS image

Run the container in `--writable` mode with `--fakeroot`

```bash
$ singularity run --nv --writable --fakeroot containers/pytorch2106.sig 
``` 

Now, you will be able to use `apt`.

```bash
Singularity> apt update
Singularity> apt install tigervnc-standalone-server htop

```

If you have dependencies to be installed with `setup.py`, then you will also need these options to run the container. e.g. for LaneATT:

```bash
Singularity> cd git/LaneATT/lib/nms
Singularity> python setup.py install

```

And install the required `pip` packages. e.g. for LaneATT:

```bash
Singularity> pip install imgaug ujson shapely p_tqdm gdown thop xmljson lxml PyYAML
```

## Setting up a desktop environment for visualization

Exit from the container and run vncserver outside the container. Observe the display number created by the vncserver. In this example, it is :1. We will later export this value to DISPLAY inside the Singularity container. 

```bash
$ vncserver

New 'tair-gibraltar:5 (deneme2)' desktop is tair-gibraltar:1

Starting applications specified in /home/deneme2/.vnc/xstartup
Log file is /home/deneme2/.vnc/tair-gibraltar:5.log

```

## Running the container with the required bindings 

Now the container is ready to run your code. You can observe the opencv output through a VNC viewer. `--bind` option will directly link the physical locations in your computer to the symbolic folder in the container.

```bash
$ singularity run --nv --bind /datasets,/workspace,/media containers/pytorch2106.sig
```


```bash
Singularity> export DISPLAY=:1
Singularity> cd git/LaneATT
Singularity> python main.py test --exp_name /workspace/experiments/laneatt_r34_culane --view all
```

## Connecting container to jupyterLab
You can look at [Making Python not awful with containers](https://pawseysc.github.io/singularity-containers/24-ml-python/index.html)

In order to connect singularity folder or image to Jupyter, enter the command below where "mmseg2" is the name of the container. 
```
$ singularity exec --nv --bind /datasets,/workspace -C -B $(pwd):$HOME ~/containers/mmseg_2107.sig jupyter-lab --no-browser --port=8888 --ip 0.0.0.0 --notebook-dir=$HOME
```
If you want to connect remotely to the server from your local computer, you can link this port with the command
```
$ ssh ek21@10.29.40.180 -L 8888:localhost:8888
```

Then connect to below from the browser  
```
localhost:8888
```

Note: If you run the container with --fakeroot, remove -C argument. Here the -C flag is to isolate the container from the host.

## Converting sandbox to image
It can be implemented as below. It decreases the dimension of example from 16GB to 5.7GB.
```bash
$ singularity build --fakeroot --nv mmseg_2107.sif mmseg_2107.sig/
```
