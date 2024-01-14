# object-detection-with-raspberry-pi-5

Object detection with Raspberry Pi 5, Raspberry Pi Camera Module 3, and Google Coral Edge TPU

## How To Setup M.2 or Mini PCIe Accelerator Coral Edge TPU with a Virtual Environment

### Introduction

This README guide provides step-by-step instructions on setting up the Coral Edge TPU, either in the M.2 or Mini PCIe form-factor. We will also cover setting up a Python virtual environment for running TensorFlow Lite models on the Edge TPU.

### Requirements

- A computer with one of the following operating systems:
  - Linux: 64-bit Debian 10 or Ubuntu 16.04 (or newer), x86-64 or ARMv8 architecture
  - Windows: 64-bit Windows 10, x86-64 architecture
- Support for MSI-X as defined in the PCI 3.0 specification
- At least one available Mini PCIe or M.2 module slot
- Python 3.6-3.9

### Setup Steps

#### 1. Connect the Module

- Ensure the host system is shut down.
- Connect the Coral Mini PCIe or M.2 module to the corresponding slot on your host system.

#### 2. Install the PCIe Driver and Edge TPU Runtime

##### On Linux

- Check your Linux kernel version: `uname -r`
  - If it's 4.18 or lower, proceed with the installation.
  - If it's 4.19 or higher, check for a pre-built Apex driver: `lsmod | grep apex`
    - If nothing is printed, proceed with the installation.
    - If an Apex module is listed, follow the workaround to disable Apex and Gasket.
- Add the Debian package repository:

```bash
echo "deb https://packages.cloud.google.com/apt coral-edgetpu-stable main" | sudo tee /etc/apt/sources.list.d/coral-edgetpu.list
curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo apt-get update
```

- Install the PCIe driver and Edge TPU runtime:

  ```bash
  sudo apt-get install gasket-dkms libedgetpu1-std
  ```

- If necessary, add a udev rule and ensure your user is in the 'apex' group:

```bash
sudo sh -c "echo 'SUBSYSTEM==\"apex\", MODE=\"0660\", GROUP=\"apex\"' >> /etc/udev/rules.d/65-apex.rules"
sudo groupadd apex
sudo adduser $USER apex
```

- Reboot the system and verify the module is detected: `lspci -nn | grep 089a`
- Verify the driver: `ls /dev/apex_0`

##### On Windows

- Install the latest Microsoft Visual C++ 2019 redistributable.
- Download and extract `edgetpu_runtime_20221024.zip`, then run `install.bat`.

#### 3. Install the PyCoral Library

##### On Linux

```bash
sudo apt-get install python3-pycoral
```

- If you receive an installation error that looks like the below snippet, you will have to setup a Python virtual environment with `pyenv` with the Python version being between 3.6 and 3.9 (see Step 4):

```bash
The following packages have unmet dependencies:
 python3-pycoral : Depends: python3-tflite-runtime (= 2.5.0.post1) but it is not going to be installed
                   Depends: python3 (< 3.10) but 3.11.2-1+b1 is to be installed
E: Unable to correct problems, you have held broken packages.
pi@pi:~$
```

##### On Windows

```bash
python3 -m pip install --extra-index-url https://google-coral.github.io/py-repo/ pycoral~=2.0
```

#### 4. Setup Python Virtual Environment with `pyenv`

- Add `pyenv` to `~/.bash_profile`:

```bash
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
```

- Create and activate a new virtual environment:

```bash
cd /path/to/your/project # Where you want to create the virtual environment, install packages, and run your code, e.g.: cd ~/Documents/Projects/Repos/object-detection-with-raspberry-pi-5
sudo apt-get install -y python3-venv
pyenv install 3.9
pyenv virtualenv 3.9 myenv
pyenv activate myenv
```

- Upgrade pip and install necessary packages:

```bash
pip install --upgrade pip
pip install pycoral Pillow numpy
```

#### 5. Run Pre-Built Example Model on the Edge TPU

##### Navigate to the example directory and run a classification model

```bash
mkdir coral && cd coral
git clone https://github.com/google-coral/pycoral.git
cd pycoral
```

##### Download the model, labels, and bird photo

```bash
bash examples/install_requirements.sh classify_image.py
```

##### Run the image classifier with the bird photo

```bash
python3 examples/classify_image.py \
--model test_data/mobilenet_v2_1.0_224_inat_bird_quant_edgetpu.tflite \
--labels test_data/inat_bird_labels.txt \
--input test_data/parrot.jpg
```

### Troubleshooting on Linux

- Resolve HIB errors on ARM64 platforms by modifying kernel arguments to include `gasket.dma_bit_mask=32`.
- Resolve pcieport errors by adding `pcie_aspm=off` to the kernel command line arguments.
- To disable pre-built Apex and Gasket drivers, use the `/etc/modprobe.d/blacklist-apex.conf` workaround.

### Next Steps

- Explore other models and example projects provided by Coral.
- Train your own models using TensorFlow and make them compatible with the Edge TPU.

For more detailed information, visit the [Coral Accelerator Get Started Guide](https://coral.ai/docs/m2/get-started/).

### Resources

- [Install Docker Engine on Debian](https://docs.docker.com/engine/install/debian/)
- [Install TensorFlow Lite for Python on a Raspberry Pi](https://github.com/tensorflow/tensorflow/blob/v2.5.0/tensorflow/lite/g3doc/guide/python.md#install-tensorflow-lite-for-python)
- [TensorFlow Lite runtime package - Quickstart for Linux-based devices with Python](https://www.tensorflow.org/lite/guide/python)