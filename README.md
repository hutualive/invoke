# invoke
use invoke as project cli:

1. we can run:  
-invoke build to build the project with ccache and parallelized
-invoke console will connect to the device’s serial console
-invoke gdbserver will spawn JLinkGDBServer with the correct configuration
-invoke flash will flash the binary through GDB and give us a prompt to the device
-invoke debug will attach to a running device using GDB without flashing

2. for each command, we will check that the proper binaries/packages are installed using pre tasks
3. we can run inv --list and inv <command> --help for help menus

# install invoke  
$ virtualenv invoke
$ source invoke/bin/activate
$ pip install invoke

# invoke script  --> put a tasks.py under project root
```python
# /nrf5_sdk/tasks.py

import os

from invoke import Collection, task

ROOT_DIR = os.path.dirname(__file__)
BLINKY_DIR = os.path.join("nrf5_sdk", "examples", "peripheral",
                          "blinky", "pca10056", "blank", "armgcc")

@task
def build(ctx):
    """Build the project"""
    with ctx.cd(PCA10056_BLINKY_ARMGCC_DIR):
        ctx.run("make")
```

// test  
$ inv --echo build

// iterate
```python
@task
def build(ctx):
    """Build the project"""
    with ctx.cd(PCA10056_BLINKY_ARMGCC_DIR):
        ctx.run("make build -j4")
```

// add ccache  
# ⁨nRF5_SDK_15.2.0/components⁩/toolchain⁩/gcc⁩/Makefile.common

CCACHE ?= $(if $(filter Windows%,$(OS)),, \
               $(if $(wildcard /usr/bin/ccache),ccache))  

```python
from shutil import which

@task
def build(ctx, ccache=True):
    """Build the project"""
    env = {}
    if ccache:
      env = {'CCACHE': which(ccache)}

    with ctx.cd(PCA10056_BLINKY_ARMGCC_DIR):
        ctx.run("make build -j4", env=env)
```

// add dependency check
```python
@task
def check_ccache(ctx):
    ccache = which(ccache)
    if not ccache:
        msg = (
            "Couldn't find `ccache`.\n"
            "Please install it using the following command:\n\n"
            ">  `brew install ccache` OR `apt install ccache`"
        )
        raise Exit(msg)


@task(pre=[check_ccache])
def build(ctx, ccache=True):
    """Build the project"""
    ...
```  

# reference  
https://github.com/memfault/interrupt/tree/master/example/invoke-basic
