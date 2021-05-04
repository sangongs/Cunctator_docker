# Overview

Cunctator is a framework that enables [best-effort lazy evaluation for Python software built on APIs](https://2021.ecoop.org/details/ecoop-2021-ecoop-research-papers/17/Best-Effort-Lazy-Evaluation-for-Python-Software-Built-On-APIs). It tries to defer the evaluation of library APIs as long as possible, such that an optimizer can be applied to optimize a sequence of API calls before true evaluation.

What follows describes the Docker image of Cunctator, including instructions for running it and brief introduction of Cunctator's source code.

# How to run

## Run Cunctator docker image
```
sudo docker run --privileged -p 8888:8888 -i -t sangongs/cunctator /bin/bash
```

The option `--privileged` is necessary for using `numactl` command inside docker, see the next section. The port forwarding `-p 8888:8888` is for accessing [jupyter](https://jupyter.org/) notebooks running in the docker from the host machine.

## Run one benchmark

### Run in web UI (recommended)

#### Open jupyter
The benchmarks are in the form of [jupyter](https://jupyter.org/) notebook. To run them, first launch `jupyter-notebook` server in docker:
```
cd ~/dharp

# `numactl --interleave all` reduces time variance of benchmark runs
numactl --interleave all jupyter-notebook --ip=0.0.0.0 --allow-root
```
We will see logs of `jupyter-notebook` like:
```
[W 08:09:56.624 NotebookApp] No web browser found: could not locate runnable browser.
[C 08:09:56.624 NotebookApp] 
    
    Copy/paste this URL into your browser when you connect for the first time,
    to login with a token:
        http://0.0.0.0:8888/?token=14a55ad16715da1da1ec3837e494b7888738b023e09f47de
```

Replace the `0.0.0.0` in the url with `localhost`, and then open it with a web browser on the host machine. We will see the web UI of jupyter, in which we can edit or run benchmarks.

If the docker image runs on a remote machine, make sure the local port `8888` is forwaded to the remote machine. Try using the following command on local machine:
```
ssh -L 8888:localhost:8888 <user>@<yourRemoteMachine>
```

#### Run jupyter notebook

In the web UI, open a benchmark under the directory `bench/`, and then use menu item `Kernel` > `Restart & Run All`.

### Run in command line

### Memory requirement

## Run all benchmarks in batch

# Source code

## Python extension

## Framework

## Optimizer
