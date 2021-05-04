# Overview

Cunctator is a framework that enables [best-effort lazy evaluation for Python software built on APIs](https://2021.ecoop.org/details/ecoop-2021-ecoop-research-papers/17/Best-Effort-Lazy-Evaluation-for-Python-Software-Built-On-APIs). It tries to defer the evaluation of library APIs as long as possible, such that an optimizer can be applied to optimize a sequence of API calls before true evaluation.

What follows describes the docker image of Cunctator, including instructions for running it and brief introduction of Cunctator's source code.

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

Replace the `0.0.0.0` in the url with `localhost`, and then open it with a web browser on the host machine. We will see the web UI of jupyter, in which we can run or edit benchmarks.

If the docker image runs on a remote machine, make sure the local port `8888` is forwaded to the remote machine. Try using the following command on local machine:
```
ssh -L 8888:localhost:8888 <yourRemoteMachine>
```

#### Run bechmark

In the web UI of jupyter, open a benchmark under the directory `bench/`, and then use menu item `Kernel` > `Restart & Run All`.

### Run in command line
```
cd ~/dharp/bench

# execute the benchmark
jupyter nbconvert --to notebook --inplace --ExecutePreprocessor.timeout=3600 --execute numpy/p1_add3_baseline.ipynb

# inspect the result
jupyter nbconvert --to rst --stdout numpy/p1_add3_baseline.ipynb
```

### Memory requirement

Due to a memory leak issue of [weld](https://github.com/weld-project/weld), some benchmarks may take up to 128GB memory. To reduce memory usage, try editing the benchmark to use smaller input, or adjusting the number of runs using option `-r` of the [`%timeit` command](https://ipython.readthedocs.io/en/stable/interactive/magics.html#magic-timeit) in the benchmark notebook.

Additionally, after closing a notebook, jupyter does not automatically kill its python kernel, which may use a lot of memory. We need to manually kill the kernels to release memory.

## Run all benchmarks in batch
```
cd ~/dharp/bench
numactl --interleave all bash run_all_bench.sh
```

# Source code

## Python extension (`/root/cpython`)
Cunctator extends Python to support monitoring access to a Python object. The source code is under directory `/root/cpython`. Check its git history for what the extension changes.

## Framework (`/root/dharp`)

Directory `/root/dharp` contains the source code of Cunctator. 

Some important files are:
* `watch.py` implements the watch mechanism.
* `IR.py` implements the IR scrathpad for recording and evaluating deferred API calls.
* `lazy.py` implements lazy propagation.
* `optimier.py` defines the base class for optimizers.
* `intercept.py` contains some helper functions for redirecting API calls.

## Optimizers (`/root/harp/optimizers`)

There are four optimizer prototypes based on Cunctator:
* `np_reduce_temp.py`: Reducing temporary variable of numpy through using in-place update.
* `numpy2weld.py`: Converting numpy APIs to weldnumpy APIs.
* `pandas2spark.py`: Converting pandas APIs to Spark APIs.
* `spark_optimier.py`: Fixing the usage of `cache()` APIs in Spark programs.
