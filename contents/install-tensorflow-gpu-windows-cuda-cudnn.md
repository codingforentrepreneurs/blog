---
title: Install Tensorflow GPU on Windows using CUDA and cuDNN
slug: install-tensorflow-gpu-windows-cuda-cudnn

publish_timestamp: Jan. 27, 2018
url: https://www.codingforentrepreneurs.com/blog/install-tensorflow-gpu-windows-cuda-cudnn/

---

So I picked myself up a [GeForce GTX 1080 Ti](http://amzn.to/2DUvLXa) to use with Tensorflow for some deep learning on my *Windows 10* machine.

I hoped the installation process would be as simple as:

```
pip install tensorflow-gpu
```

Although it was *close* to that, there are still several other, mildly frustrating, steps you must take to get Nvidia GPU fully working. I'll say that once you get it working, training deep learning models is on orders of magnitude faster!

### Guide Overview

1. Install Nvidia's card on your computer along with drivers

2. Download & Install CUDA

3. Download & "Install" cuDNN

4. Uninstall Tensorflow, Install Tensorflow GPU

5. Update the `%PATH%` on the system

5. Verify installation


### Requirements

- NVIDA Graphics Card (Probably a 1050 & up); I'm using a 1080Ti

- Windows 10 (recommended; older versions *might* work)


## A word of caution: VERSIONS WILL CHANGE

As of this writing, CUDA 9.1 is out. Tensorflow's GPU supports **CUDA 8** and ***not*** CUDA 9. Well, as far as their Windows [install docs](https://www.tensorflow.org/install/install_windows#requirements_to_run_tensorflow_with_gpu_support) state:

> Requirements to run TensorFlow with GPU support
If you are installing TensorFlow with GPU support using one of the mechanisms described in this guide, then the following NVIDIA software must be installed on your system:

>> CUDA® Toolkit 8.0. For details, see NVIDIA's documentation Ensure that you append the relevant Cuda pathnames to the %PATH% environment variable as described in the NVIDIA documentation.

>> The NVIDIA drivers associated with CUDA Toolkit 8.0.
cuDNN v6.0. For details, see NVIDIA's documentation. Note that cuDNN is typically installed in a different location from the other CUDA DLLs. Ensure that you add the directory where you installed the cuDNN DLL to your %PATH% environment variable.

>> GPU card with CUDA Compute Capability 3.0 or higher. See NVIDIA documentation for a list of supported GPU cards.

> If you have a different version of one of the preceding packages, please change to the specified versions. In particular, the cuDNN version must match exactly: TensorFlow will not load if it cannot find cuDNN64_6.dll. To use a different version of cuDNN, you must build from source.


## Let's get started

### 1. Install Nvidia Graphics Card & Drivers

Let us know in the comments if you need help here.

### 2. Download & Install CUDA
CUDA has different versions. We need CUDA Version 8.0. I have 8.0, 9.0, and 9.1 installed and setup identically to this guide for each version. Stick with 8.0 for now to get that working. I setup the other versions to prepare for the possiblity of Tensorflow GPU supporting other CUDA versions.

1. Go to [CUDA Toolkit downloads](https://developer.nvidia.com/cuda-downloads)
2. Scroll down to **Legacy Releases** or [here](https://developer.nvidia.com/cuda-toolkit-archive)
3.  Click on the version you want *CUDA Toolkit X.Y*:
	- for 8.0, we'll see *CUDA Toolkit 8.0 GA<Z>* so replace `*<Z>*` with the highest number available. I downloaded `CUDA Toolkit 8.0 GA2`
	- for 9.0, the file is *CUDA Toolkit 9.0*
	- for 9.1, the file is *CUDA Toolkit 9.1*:

4. Select your operating system, mine is:
	- OS: Windows
	- Architecture: x86_64
	- Version: 10
5. After CUDA downloads, run the file downloaded & install with `Express Settings`. This might take a while and flicker the screen (due to it being for the graphics card and all).
6. Verify
You should know have the following path on your system:
```
C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v8.0
```


### 3. Download & "Install" cuDNN
For this, you'll need an NVIDIA developer account. It's free. 

1. Create a free NVIDIA Developer Membership [here](https://developer.nvidia.com/developer-program/signup)
2. After you sign up go to: [https://developer.nvidia.com/cudnn](https://developer.nvidia.com/cudnn)
3. Click "Download" (ignore the current listed version for now)
4. Agree to the Terms
5. Remember how above we need `cuDNN v6.0` from above? You might see this listed here, you might not. If you don't, just select **Archived cuDNN Releases**
6. Click the version you need as well as the system you need. I clicked:
	- `Download cuDNN v6.0 (April 27, 2017), for CUDA 8.0`
	- then, `cuDNN v6.0 Library for Windows 10`
7. Go to your recent downloaded zip file, something like:
```
C:\Users\teamcfe\Downloads\cudnn-8.0-windows10-x64-v6.0.zip
```
8. Unzip the file
9. Open "cuda", you should see:
```
bin/
include/
lib/
```

10. Copy and paste the 3 folders in `C:\Users\j\Downloads\cudnn-8.0-windows10-x64-v6.0.zip\cuda` to `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v8.0`
> Do note that dragging and dropping will merge the folders and not replace them; I don't believe the same is true for Mac/Linux. If it asks you to *replace* anything, say no and just drag and drop each folder's contents from cuDNN to Cuda. It might as about admin privileges which you should just say yes.

11. Verify
If you did the last step correctly, you should be able to find this path:
```
C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v8.0\lib\x64\cudnn.lib
```


### 4. Uninstall Tensorflow, Install Tensorflow GPU
Remove tensorflow from your system if it's currently installed with:
```
pip uninstall tensorflow
```
Because we want to use tensorflow with GPU support. It's easy just do:
```
pip install tensorflow-gpu
```
I'm glad that was easy :)

### 5. Update the `%PATH%` on the system
Update your **system** environment variables' `PATH` to have:

- `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v8.0\bin`
- `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v8.0\libnvvp`

To get here, do a start menu/cortana search for `Edit the system environment variables`
- It should open `System Properties` and the `Advanced` tab.
- Click 'Environment Variables'
- Under `System Variables` look for `PATH`, click `edit`
- Add the 2 lines from above

### 6. Verify installation.

There are two python scripts you can use:

##### Verify GPU & Tensorflow -- Option 1 -- without suggestions
```
from tensorflow.python.client import device_lib
print(device_lib.list_local_devices())

```
In that print statement you should see something like:
```
[name: "/device:CPU:0"
device_type: "CPU"
memory_limit: 268435456
locality {
}
incarnation: 3790989966624548110
, name: "/device:GPU:0"
device_type: "GPU"
memory_limit: 9226016523
locality {
  bus_id: 1
}
incarnation: 14277882179479335037
physical_device_desc: "device: 0, name: GeForce GTX 1080 Ti, pci bus id: 0000:01:00.0, compute capability: 6.1"
]
```

##### Verify GPU & Tensorflow -- Option 2 -- with suggestions

```
# Copyright 2015 The TensorFlow Authors. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================
"""A script for testing that TensorFlow is installed correctly on Windows.

The script will attempt to verify your TensorFlow installation, and print
suggestions for how to fix your installation.
"""

import ctypes
import imp
import sys

def main():
  try:
    import tensorflow as tf
    print("TensorFlow successfully installed.")
    if tf.test.is_built_with_cuda():
      print("The installed version of TensorFlow includes GPU support.")
    else:
      print("The installed version of TensorFlow does not include GPU support.")
    sys.exit(0)
  except ImportError:
    print("ERROR: Failed to import the TensorFlow module.")

  candidate_explanation = False

  python_version = sys.version_info.major, sys.version_info.minor
  print("\n- Python version is %d.%d." % python_version)
  if not (python_version == (3, 5) or python_version == (3, 6)):
    candidate_explanation = True
    print("- The official distribution of TensorFlow for Windows requires "
          "Python version 3.5 or 3.6.")
  
  try:
    _, pathname, _ = imp.find_module("tensorflow")
    print("\n- TensorFlow is installed at: %s" % pathname)
  except ImportError:
    candidate_explanation = False
    print("""
- No module named TensorFlow is installed in this Python environment. You may
  install it using the command `pip install tensorflow`.""")

  try:
    msvcp140 = ctypes.WinDLL("msvcp140.dll")
  except OSError:
    candidate_explanation = True
    print("""
- Could not load 'msvcp140.dll'. TensorFlow requires that this DLL be
  installed in a directory that is named in your %PATH% environment
  variable. You may install this DLL by downloading Microsoft Visual
  C++ 2015 Redistributable Update 3 from this URL:
  https://www.microsoft.com/en-us/download/details.aspx?id=53587""")

  try:
    cudart64_80 = ctypes.WinDLL("cudart64_80.dll")
  except OSError:
    candidate_explanation = True
    print("""
- Could not load 'cudart64_80.dll'. The GPU version of TensorFlow
  requires that this DLL be installed in a directory that is named in
  your %PATH% environment variable. Download and install CUDA 8.0 from
  this URL: https://developer.nvidia.com/cuda-toolkit""")

  try:
    nvcuda = ctypes.WinDLL("nvcuda.dll")
  except OSError:
    candidate_explanation = True
    print("""
- Could not load 'nvcuda.dll'. The GPU version of TensorFlow requires that
  this DLL be installed in a directory that is named in your %PATH%
  environment variable. Typically it is installed in 'C:\Windows\System32'.
  If it is not present, ensure that you have a CUDA-capable GPU with the
  correct driver installed.""")

  cudnn5_found = False
  try:
    cudnn5 = ctypes.WinDLL("cudnn64_5.dll")
    cudnn5_found = True
  except OSError:
    candidate_explanation = True
    print("""
- Could not load 'cudnn64_5.dll'. The GPU version of TensorFlow
  requires that this DLL be installed in a directory that is named in
  your %PATH% environment variable. Note that installing cuDNN is a
  separate step from installing CUDA, and it is often found in a
  different directory from the CUDA DLLs. You may install the
  necessary DLL by downloading cuDNN 5.1 from this URL:
  https://developer.nvidia.com/cudnn""")

  cudnn6_found = False
  try:
    cudnn = ctypes.WinDLL("cudnn64_6.dll")
    cudnn6_found = True
  except OSError:
    candidate_explanation = True

  if not cudnn5_found or not cudnn6_found:
    print()
    if not cudnn5_found and not cudnn6_found:
      print("- Could not find cuDNN.")
    elif not cudnn5_found:
      print("- Could not find cuDNN 5.1.")
    else:
      print("- Could not find cuDNN 6.")
      print("""
  The GPU version of TensorFlow requires that the correct cuDNN DLL be installed
  in a directory that is named in your %PATH% environment variable. Note that
  installing cuDNN is a separate step from installing CUDA, and it is often
  found in a different directory from the CUDA DLLs. The correct version of
  cuDNN depends on your version of TensorFlow:
  
  * TensorFlow 1.2.1 or earlier requires cuDNN 5.1. ('cudnn64_5.dll')
  * TensorFlow 1.3 or later requires cuDNN 6. ('cudnn64_6.dll')
    
  You may install the necessary DLL by downloading cuDNN from this URL:
  https://developer.nvidia.com/cudnn""")
    
  if not candidate_explanation:
    print("""
- All required DLLs appear to be present. Please open an issue on the
  TensorFlow GitHub page: https://github.com/tensorflow/tensorflow/issues""")

  sys.exit(-1)

if __name__ == "__main__":
  main()
```
[source](https://gist.github.com/mrry/ee5dbcfdd045fa48a27d56664411d41c)



### Final note

I have CUDA 8.0, 9.0, and 9.1 installed with the corresponding cuDNN in each version. The cuDNN installation process is the *exact* same for each version of CUDA except you might need a different version of cuDNN depending on the suggestions given from option 2 in step 6.


=============================================

Find a bug in this guide? Please let us know in the comments using markdown syntax so we can all benefit from this. 

Thank you!