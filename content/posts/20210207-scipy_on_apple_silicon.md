+++
title = "Build SciPy on Apple Silicon"
date = 2021-02-07T03:18:18+01:00
slug = "scipy_on_apple_silicon"

[taxonomies]
categories = ["macos", "python"]
+++

Have you managed to get your hands on one of Apple's new machines with their very own Apple SoC? And then you needed to install `scipy` in some freshly cloned Python project?

Yeah, I went through that and `scipy` just not budging, failing dramatically during its build process. Here's what I did while waiting for an official native whl package. 

Make sure you're using Python 3.9.1 and then just try this out.

```sh
brew install openblas
pip install --upgrade pip setuptools wheel
pip install cython
OPENBLAS=$(brew --prefix openblas) pip install scipy --no-binary :all: --no-use-pep517
```

Waiting time may vary. Good luck!