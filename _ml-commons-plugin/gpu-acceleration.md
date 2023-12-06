---
layout: default
title: GPU加速度
parent: 在OpenSearch中使用ML模型
nav_order: 150
---
a---
layout: default
title: GPU加速度
parent: 在OpenSearch中使用ML模型
nav_order: 150
---


# GPU加速度

使用机器学习（ML）节点在OpenSearch集群中运行自然语言处理（NLP）模型时，您可以使用图形处理单元（GPU）加速在ML节点上获得更好的性能。GPU可以与群集的CPU协同工作，以加快模型上传和培训。

## 支持的GPU

目前，ML节点支持以下GPU实例：

- [NVIDIA实例与CUDA 11.6](https://aws.amazon.com/nvidia/)
- [AWS推理](https://aws.amazon.com/machine-learning/inferentia/)

如果您需要GPU功率，则可以通过[亚马逊弹性计算云（Amazon EC2）](https://aws.amazon.com/ec2/)。有关如何提供GPU实例的更多信息，请参阅[推荐的GPU实例](https://docs.aws.amazon.com/dlami/latest/devguide/gpu.html)。

## 支持的图像

您可以使用GPU加速[Docker图像](https://gitlab.com/nvidia/container-images/cuda/blob/master/doc/supported-tags.md) 与CUDA 11.6和[亚马逊机器图像（AMIS）](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html)。

## Pytorch

GPU-加速的ML节点需要[Pytorch](https://pytorch.org/docs/stable/index.html) 1.12.1使用ML模型。

## 设置GPU-加速ML节点

根据GPU，您可以提供GPU-手动或使用自动初始化脚本加速ML节点。

### 准备NVIDIA ML节点

NVIDIA使用CUDA来提高节点性能。为了利用CUDA，您需要确保您的司机包括`nvidia-uvm` 内核`/dev` 目录。要检查内核，请输入`ls -al /dev | grep nvidia-uvm`。

如果是`nvidia-uvm` 内核不存在，运行`nvidia-uvm-init.sh`：

```
#!/bin/bash
## Script to initialize nvidia device nodes.
## https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#runfile-verifications
/sbin/modprobe nvidia
if [ "$?" -eq 0 ]; then
  # Count the number of NVIDIA controllers found.
  NVDEVS=`lspci | grep -i NVIDIA`
  N3D=`echo "$NVDEVS" | grep "3D controller" | wc -l`
  NVGA=`echo "$NVDEVS" | grep "VGA compatible controller" | wc -l`
  N=`expr $N3D + $NVGA - 1`
  for i in `seq 0 $N`; do
    mknod -m 666 /dev/nvidia$i c 195 $i
  done
  mknod -m 666 /dev/nvidiactl c 195 255
else
  exit 1
fi
/sbin/modprobe nvidia-uvm
if [ "$?" -eq 0 ]; then
  # Find out the major device number used by the nvidia-uvm driver
  D=`grep nvidia-uvm /proc/devices | awk '{print $1}'`
  mknod -m 666 /dev/nvidia-uvm c $D 0
  mknod -m 666 /dev/nvidia-uvm-tools c $D 0
else
  exit 1
fi
```

验证这一点`nvidia-uvm` 存在于下面`/dev`，您可以在群集内启动OpenSearch。

### 准备AWS推理ML节点

根据在AWS推理上运行的Linux操作系统，您可以使用以下命令和脚本来提供ML节点并在群集内运行OpenSearch。

开始，[下载并安装OpenSearch]({{site.url}}{{site.baseurl}}/install-and-configure/index/) 在你的集群上。

然后导出OpenSearch并设置您的环境变量。此示例将OpenSearch导出到目录`opensearch-2.5.0`， 所以`OPENSEARCH_HOME` =`opensearch-2.5.0`：

```
echo "export OPENSEARCH_HOME=~/opensearch-2.5.0" | tee -a ~/.bash_profile
echo "export PYTORCH_VERSION=1.12.1" | tee -a ~/.bash_profile
source ~/.bash_profile
```

接下来，创建一个名为的shell脚本文件`prepare_torch_neuron.sh`。您可以根据Linux操作系统复制和自定义以下示例之一：

- [Ubuntu 20.04](#ubuntu-2004)
- [亚马逊Linux 2](#amazon-linux-2)

运行脚本后，退出当前终端并打开一个新终端以启动OpenSearch。

GPU加速度仅在Ubuntu 20.04和Amazon Linux2上进行了测试。但是，您可以使用其他Linux操作系统。
{:.note}

#### Ubuntu 20.04

```
. /etc/os-release
sudo tee /etc/apt/sources.list.d/neuron.list > /dev/null <<EOF
deb https://apt.repos.neuron.amazonaws.com ${VERSION_CODENAME} main
EOF
wget -qO - https://apt.repos.neuron.amazonaws.com/GPG-PUB-KEY-AMAZON-AWS-NEURON.PUB | sudo apt-key add -

# Update OS packages
sudo apt-get update -y

################################################################################################################
# To install or update to Neuron versions 1.19.1 and newer from previous releases:
# - DO NOT skip 'aws-neuron-dkms' install or upgrade step, you MUST install or upgrade to latest Neuron driver
################################################################################################################

# Install OS headers
sudo apt-get install linux-headers-$(uname -r) -y

# Install Neuron Driver
sudo apt-get install aws-neuronx-dkms -y

####################################################################################
# Warning: If Linux kernel is updated as a result of OS package update
#          Neuron driver (aws-neuron-dkms) should be re-installed after reboot
####################################################################################

# Install Neuron Tools
sudo apt-get install aws-neuronx-tools -y

######################################################
#   Only for Ubuntu 20 - Install Python3.7
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt-get install python3.7
######################################################
# Install Python venv and activate Python virtual environment to install    
# Neuron pip packages.
cd ~
sudo apt-get install -y python3.7-venv g++
python3.7 -m venv pytorch_venv
source pytorch_venv/bin/activate
pip install -U pip

# Set pip repository to point to the Neuron repository
pip config set global.extra-index-url https://pip.repos.neuron.amazonaws.com

#Install Neuron PyTorch
pip install torch-neuron torchvision
# If you need to trace the neuron model, install torch neuron with this command
# pip install torch-neuron neuron-cc[tensorflow] "protobuf==3.20.1" torchvision

# If you need to trace neuron model, install the transformers for tracing the Huggingface model.
# pip install transformers

# Copy torch neuron lib to OpenSearch
PYTORCH_NEURON_LIB_PATH=~/pytorch_venv/lib/python3.7/site-packages/torch_neuron/lib/
mkdir -p $OPENSEARCH_HOME/lib/torch_neuron; cp -r $PYTORCH_NEURON_LIB_PATH/ $OPENSEARCH_HOME/lib/torch_neuron
export PYTORCH_EXTRA_LIBRARY_PATH=$OPENSEARCH_HOME/lib/torch_neuron/lib/libtorchneuron.so
echo "export PYTORCH_EXTRA_LIBRARY_PATH=$OPENSEARCH_HOME/lib/torch_neuron/lib/libtorchneuron.so" | tee -a ~/.bash_profile

# Increase JVm stack size to >=2MB
echo "-Xss2m" | tee -a $OPENSEARCH_HOME/config/jvm.options
# Increase max file descriptors to 65535
echo "$(whoami) - nofile 65535" | sudo tee -a /etc/security/limits.conf
# max virtual memory areas vm.max_map_count to 262144
sudo sysctl -w vm.max_map_count=262144
```

#### 亚马逊Linux 2

```
# Configure Linux for Neuron repository updates
sudo tee /etc/yum.repos.d/neuron.repo > /dev/null <<EOF
[neuron]
name=Neuron YUM Repository
baseurl=https://yum.repos.neuron.amazonaws.com
enabled=1
metadata_expire=0
EOF
sudo rpm --import https://yum.repos.neuron.amazonaws.com/GPG-PUB-KEY-AMAZON-AWS-NEURON.PUB
# Update OS packages
sudo yum update -y
################################################################################################################
# To install or update to Neuron versions 1.19.1 and newer from previous releases:
# - DO NOT skip 'aws-neuron-dkms' install or upgrade step, you MUST install or upgrade to latest Neuron driver
################################################################################################################
# Install OS headers
sudo yum install kernel-devel-$(uname -r) kernel-headers-$(uname -r) -y
# Install Neuron Driver
####################################################################################
# Warning: If Linux kernel is updated as a result of OS package update
#          Neuron driver (aws-neuron-dkms) should be re-installed after reboot
####################################################################################
sudo yum install aws-neuronx-dkms -y
# Install Neuron Tools
sudo yum install aws-neuronx-tools -y

# Install Python venv and activate Python virtual environment to install    
# Neuron pip packages.
cd ~
sudo yum install -y python3.7-venv gcc-c++
python3.7 -m venv pytorch_venv
source pytorch_venv/bin/activate
pip install -U pip

# Set Pip repository  to point to the Neuron repository
pip config set global.extra-index-url https://pip.repos.neuron.amazonaws.com

# Install Neuron PyTorch
pip install torch-neuron torchvision
# If you need to trace the neuron model, install torch neuron with this command
# pip install torch-neuron neuron-cc[tensorflow] "protobuf<4" torchvision

# If you need to run the trace neuron model, install transformers for tracing Huggingface model.
# pip install transformers

# Copy torch neuron lib to OpenSearch
PYTORCH_NEURON_LIB_PATH=~/pytorch_venv/lib/python3.7/site-packages/torch_neuron/lib/
mkdir -p $OPENSEARCH_HOME/lib/torch_neuron; cp -r $PYTORCH_NEURON_LIB_PATH/ $OPENSEARCH_HOME/lib/torch_neuron
export PYTORCH_EXTRA_LIBRARY_PATH=$OPENSEARCH_HOME/lib/torch_neuron/lib/libtorchneuron.so
echo "export PYTORCH_EXTRA_LIBRARY_PATH=$OPENSEARCH_HOME/lib/torch_neuron/lib/libtorchneuron.so" | tee -a ~/.bash_profile
# Increase JVm stack size to >=2MB
echo "-Xss2m" | tee -a $OPENSEARCH_HOME/config/jvm.options
# Increase max file descriptors to 65535
echo "$(whoami) - nofile 65535" | sudo tee -a /etc/security/limits.conf
# max virtual memory areas vm.max_map_count to 262144
sudo sysctl -w vm.max_map_count=262144
```

脚本完成运行后，打开一个新终端以进行设置生效。然后，启动OpenSearch。

OpenSearch现在应该在GPU内部运行-加速群集。但是，如果在配置过程中发生任何错误，则可以手动安装GPU加速器驱动程序。

#### 手动准备ML节点

如果前两个脚本没有提供您的GPU-正确加速节点，您可以手动安装AWS驱动程序：

1. 根据所选Linux操作系统部署AWS加速器实例。有关说明，请参阅[在AWS加速器实例上部署](https://awsdocs-neuron.readthedocs-hosted.com/en/latest/frameworks/torch/torch-neuron/setup/pytorch-install.html#deploy-on-aws-ml-accelerator-instance)。

2. 将神经元库复制到OpenSearch中。以下命令使用名为的目录`opensearch-2.5.0`：

   ```
   OPENSEARCH_HOME=~/opensearch-2.5.0
   ```

3. 设置`PYTORCH_EXTRA_LIBRARY_PATH` 小路。在此示例中，我们创建一个`pytorch` opensearch_home文件夹中的虚拟环境：

   ```
   pytorch_neuron_lib_path =〜/pytorch_venv/lib/python3.7/site-包装/TORCH_NEURON/LIB/


   Mkdir-p $ opensearch_home/lib/torch_neuron;CP-r $ pytorch_neuron_lib_path/$ opensearch_home/lib/torch_neuron
   export pytorch_extra_library_path = $ opensearch_home/lib/torch_neuron/lib/libtorchneuron.so so
  ```

4. (Optional) To monitor the GPU usage of your accelerator instance, install [Neuron tools](https://awsdocs-neuron.readthedocs-hosted.com/en/latest/tools/index.html), which allows models to be used inside your instance:

   ```
   # 安装神经元工具
   sudo apt-获得安装AWS-神经元-工具-y
   ```

   ```
   # 添加神经元工具您的路径
   导出路径=/opt/aws/neuron/bin：$路径
   ```
  
   ```
   # 测试神经元工具
   神经元-顶部
   ```


5. To make sure you have enough memory to upload a model, increase the JVM stack size to `>+2MB`:

   ```
   回声"-Xss2m" | Sudo Tee-a $ opensearch_home/config/jvm.options
   ```

6. Start OpenSearch. 

## Troubleshooting

Due to the amount of data required to work with ML models, you might encounter the following `最大文件描述符` or `vm.max_map_count` 尝试在您的群集中运行OpenSearch时出错：

```
[1]: max file descriptors [8192] for opensearch process is too low, increase to at least [65535]
[2]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
```

要解决最大文件描述符错误，请运行以下命令：

```
echo "$(whoami) - nofile 65535" | sudo tee -a /etc/security/limits.conf
```

修复`vm.max_map_count` 错误，运行此命令将计数增加到`262114`：

```
sudo sysctl -w vm.max_map_count=262144
```

## 下一步

如果您想尝试GPU-使用AWS推理的加速群集与预处理的拥抱面模型，请参阅[编译和部署拥抱面预先审慎的伯特](https://awsdocs-neuron.readthedocs-hosted.com/en/latest/src/examples/pytorch/bert_tutorial/tutorial_pretrained_bert.html)。




# GPU acceleration 

When running a natural language processing (NLP) model in your OpenSearch cluster with a machine learning (ML) node, you can achieve better performance on the ML node using graphics processing unit (GPU) acceleration. GPUs can work in tandem with the CPU of your cluster to speed up the model upload and training. 

## Supported GPUs

Currently, ML nodes support the following GPU instances:

- [NVIDIA instances with CUDA 11.6](https://aws.amazon.com/nvidia/)
- [AWS Inferentia](https://aws.amazon.com/machine-learning/inferentia/)

If you need GPU power, you can provision GPU instances through [Amazon Elastic Compute Cloud (Amazon EC2)](https://aws.amazon.com/ec2/). For more information on how to provision a GPU instance, see [Recommended GPU Instances](https://docs.aws.amazon.com/dlami/latest/devguide/gpu.html).

## Supported images

You can use GPU acceleration with both [Docker images](https://gitlab.com/nvidia/container-images/cuda/blob/master/doc/supported-tags.md) with CUDA 11.6 and [Amazon Machine Images (AMIs)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html).

## PyTorch

GPU-accelerated ML nodes require [PyTorch](https://pytorch.org/docs/stable/index.html) 1.12.1 work with ML models.

## Setting up a GPU-accelerated ML node

Depending on the GPU, you can provision a GPU-accelerated ML node manually or by using automated initialization scripts. 

### Preparing an NVIDIA ML node

NVIDIA uses CUDA to increase node performance. In order to take advantage of CUDA, you need to make sure that your drivers include the `nvidia-uvm` kernel inside the `/dev` directory. To check for the kernel, enter `ls -al /dev | grep nvidia-uvm`.

If the `nvidia-uvm` kernel does not exist, run `nvidia-uvm-init.sh`:

```
#!/bin/bash
## Script to initialize nvidia device nodes.
## https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#runfile-verifications
/sbin/modprobe nvidia
if [ "$?" -eq 0 ]; then
  # Count the number of NVIDIA controllers found.
  NVDEVS=`lspci | grep -i NVIDIA`
  N3D=`echo "$NVDEVS" | grep "3D controller" | wc -l`
  NVGA=`echo "$NVDEVS" | grep "VGA compatible controller" | wc -l`
  N=`expr $N3D + $NVGA - 1`
  for i in `seq 0 $N`; do
    mknod -m 666 /dev/nvidia$i c 195 $i
  done
  mknod -m 666 /dev/nvidiactl c 195 255
else
  exit 1
fi
/sbin/modprobe nvidia-uvm
if [ "$?" -eq 0 ]; then
  # Find out the major device number used by the nvidia-uvm driver
  D=`grep nvidia-uvm /proc/devices | awk '{print $1}'`
  mknod -m 666 /dev/nvidia-uvm c $D 0
  mknod -m 666 /dev/nvidia-uvm-tools c $D 0
else
  exit 1
fi
```

After verifying that `nvidia-uvm` exists under `/dev`, you can start OpenSearch inside your cluster. 

### Preparing AWS Inferentia ML node

Depending on the Linux operating system running on AWS Inferentia, you can use the following commands and scripts to provision an ML node and run OpenSearch inside your cluster. 

To start, [download and install OpenSearch]({{site.url}}{{site.baseurl}}/install-and-configure/index/) on your cluster.

Then export OpenSearch and set up your environment variables. This example exports OpenSearch into the directory `opensearch-2.5.0`, so `OPENSEARCH_HOME` = `opensearch-2.5.0`:

```
echo "export OPENSEARCH_HOME=~/opensearch-2.5.0" | tee -a ~/.bash_profile
echo "export PYTORCH_VERSION=1.12.1" | tee -a ~/.bash_profile
source ~/.bash_profile
```

Next, create a shell script file called `prepare_torch_neuron.sh`. You can copy and customize one of the following examples based on your Linux operating system:

- [Ubuntu 20.04](#ubuntu-2004)
- [Amazon Linux 2](#amazon-linux-2)

After you've run the scripts, exit your current terminal and open a new terminal to start OpenSearch.

GPU acceleration has only been tested on Ubuntu 20.04 and Amazon Linux 2. However, you can use other Linux operating systems.
{:.note}

#### Ubuntu 20.04

```
. /etc/os-release
sudo tee /etc/apt/sources.list.d/neuron.list > /dev/null <<EOF
deb https://apt.repos.neuron.amazonaws.com ${VERSION_CODENAME} main
EOF
wget -qO - https://apt.repos.neuron.amazonaws.com/GPG-PUB-KEY-AMAZON-AWS-NEURON.PUB | sudo apt-key add -

# Update OS packages
sudo apt-get update -y

################################################################################################################
# To install or update to Neuron versions 1.19.1 and newer from previous releases:
# - DO NOT skip 'aws-neuron-dkms' install or upgrade step, you MUST install or upgrade to latest Neuron driver
################################################################################################################

# Install OS headers
sudo apt-get install linux-headers-$(uname -r) -y

# Install Neuron Driver
sudo apt-get install aws-neuronx-dkms -y

####################################################################################
# Warning: If Linux kernel is updated as a result of OS package update
#          Neuron driver (aws-neuron-dkms) should be re-installed after reboot
####################################################################################

# Install Neuron Tools
sudo apt-get install aws-neuronx-tools -y

######################################################
#   Only for Ubuntu 20 - Install Python3.7
sudo add-apt-repository ppa:deadsnakes/ppa
sudo apt-get install python3.7
######################################################
# Install Python venv and activate Python virtual environment to install    
# Neuron pip packages.
cd ~
sudo apt-get install -y python3.7-venv g++
python3.7 -m venv pytorch_venv
source pytorch_venv/bin/activate
pip install -U pip

# Set pip repository to point to the Neuron repository
pip config set global.extra-index-url https://pip.repos.neuron.amazonaws.com

#Install Neuron PyTorch
pip install torch-neuron torchvision
# If you need to trace the neuron model, install torch neuron with this command
# pip install torch-neuron neuron-cc[tensorflow] "protobuf==3.20.1" torchvision

# If you need to trace neuron model, install the transformers for tracing the Huggingface model.
# pip install transformers

# Copy torch neuron lib to OpenSearch
PYTORCH_NEURON_LIB_PATH=~/pytorch_venv/lib/python3.7/site-packages/torch_neuron/lib/
mkdir -p $OPENSEARCH_HOME/lib/torch_neuron; cp -r $PYTORCH_NEURON_LIB_PATH/ $OPENSEARCH_HOME/lib/torch_neuron
export PYTORCH_EXTRA_LIBRARY_PATH=$OPENSEARCH_HOME/lib/torch_neuron/lib/libtorchneuron.so
echo "export PYTORCH_EXTRA_LIBRARY_PATH=$OPENSEARCH_HOME/lib/torch_neuron/lib/libtorchneuron.so" | tee -a ~/.bash_profile

# Increase JVm stack size to >=2MB
echo "-Xss2m" | tee -a $OPENSEARCH_HOME/config/jvm.options
# Increase max file descriptors to 65535
echo "$(whoami) - nofile 65535" | sudo tee -a /etc/security/limits.conf
# max virtual memory areas vm.max_map_count to 262144
sudo sysctl -w vm.max_map_count=262144
```

#### Amazon Linux 2

```
# Configure Linux for Neuron repository updates
sudo tee /etc/yum.repos.d/neuron.repo > /dev/null <<EOF
[neuron]
name=Neuron YUM Repository
baseurl=https://yum.repos.neuron.amazonaws.com
enabled=1
metadata_expire=0
EOF
sudo rpm --import https://yum.repos.neuron.amazonaws.com/GPG-PUB-KEY-AMAZON-AWS-NEURON.PUB
# Update OS packages
sudo yum update -y
################################################################################################################
# To install or update to Neuron versions 1.19.1 and newer from previous releases:
# - DO NOT skip 'aws-neuron-dkms' install or upgrade step, you MUST install or upgrade to latest Neuron driver
################################################################################################################
# Install OS headers
sudo yum install kernel-devel-$(uname -r) kernel-headers-$(uname -r) -y
# Install Neuron Driver
####################################################################################
# Warning: If Linux kernel is updated as a result of OS package update
#          Neuron driver (aws-neuron-dkms) should be re-installed after reboot
####################################################################################
sudo yum install aws-neuronx-dkms -y
# Install Neuron Tools
sudo yum install aws-neuronx-tools -y

# Install Python venv and activate Python virtual environment to install    
# Neuron pip packages.
cd ~
sudo yum install -y python3.7-venv gcc-c++
python3.7 -m venv pytorch_venv
source pytorch_venv/bin/activate
pip install -U pip

# Set Pip repository  to point to the Neuron repository
pip config set global.extra-index-url https://pip.repos.neuron.amazonaws.com

# Install Neuron PyTorch
pip install torch-neuron torchvision
# If you need to trace the neuron model, install torch neuron with this command
# pip install torch-neuron neuron-cc[tensorflow] "protobuf<4" torchvision

# If you need to run the trace neuron model, install transformers for tracing Huggingface model.
# pip install transformers

# Copy torch neuron lib to OpenSearch
PYTORCH_NEURON_LIB_PATH=~/pytorch_venv/lib/python3.7/site-packages/torch_neuron/lib/
mkdir -p $OPENSEARCH_HOME/lib/torch_neuron; cp -r $PYTORCH_NEURON_LIB_PATH/ $OPENSEARCH_HOME/lib/torch_neuron
export PYTORCH_EXTRA_LIBRARY_PATH=$OPENSEARCH_HOME/lib/torch_neuron/lib/libtorchneuron.so
echo "export PYTORCH_EXTRA_LIBRARY_PATH=$OPENSEARCH_HOME/lib/torch_neuron/lib/libtorchneuron.so" | tee -a ~/.bash_profile
# Increase JVm stack size to >=2MB
echo "-Xss2m" | tee -a $OPENSEARCH_HOME/config/jvm.options
# Increase max file descriptors to 65535
echo "$(whoami) - nofile 65535" | sudo tee -a /etc/security/limits.conf
# max virtual memory areas vm.max_map_count to 262144
sudo sysctl -w vm.max_map_count=262144
```

When the script completes running, open a new terminal for the settings to take effect. Then, start OpenSearch.

OpenSearch should now be running inside your GPU-accelerated cluster. However, if any errors occur during provisioning, you can install the GPU accelerator drivers manually.

#### Prepare ML node manually

If the previous two scripts do not provision your GPU-accelerated node properly, you can install the drivers for AWS Inferentia manually:

1. Deploy an AWS accelerator instance based on your chosen Linux operating system. For instructions, see [Deploy on AWS accelerator instance](https://awsdocs-neuron.readthedocs-hosted.com/en/latest/frameworks/torch/torch-neuron/setup/pytorch-install.html#deploy-on-aws-ml-accelerator-instance).

2. Copy the Neuron library into OpenSearch. The following command uses a directory named `opensearch-2.5.0`:

   ```
   OPENSEARCH_HOME=~/opensearch-2.5.0
   ```

3. Set the `PYTORCH_EXTRA_LIBRARY_PATH` path. In this example, we create a `pytorch` virtual environment in the OPENSEARCH_HOME folder:

   ```
   PYTORCH_NEURON_LIB_PATH=~/pytorch_venv/lib/python3.7/site-packages/torch_neuron/lib/


   mkdir -p $OPENSEARCH_HOME/lib/torch_neuron; cp -r  $PYTORCH_NEURON_LIB_PATH/ $OPENSEARCH_HOME/lib/torch_neuron
   export PYTORCH_EXTRA_LIBRARY_PATH=$OPENSEARCH_HOME/lib/torch_neuron/lib/libtorchneuron.so
  ```

4. (Optional) To monitor the GPU usage of your accelerator instance, install [Neuron tools](https://awsdocs-neuron.readthedocs-hosted.com/en/latest/tools/index.html), which allows models to be used inside your instance:

   ```
   # Install Neuron Tools
   sudo apt-get install aws-neuronx-tools -y
   ```

   ```
   # Add Neuron tools your PATH
   export PATH=/opt/aws/neuron/bin:$PATH
   ```
  
   ```
   # Test Neuron tools
   neuron-top
   ```


5. To make sure you have enough memory to upload a model, increase the JVM stack size to `>+2MB`:

   ```
   echo "-Xss2m" | sudo tee -a $OPENSEARCH_HOME/config/jvm.options
   ```

6. Start OpenSearch. 

## Troubleshooting

Due to the amount of data required to work with ML models, you might encounter the following `max file descriptors` or `vm.max_map_count` errors when trying to run OpenSearch in a your cluster: 

```
[1]: max file descriptors [8192] for opensearch process is too low, increase to at least [65535]
[2]: max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]
```

To troubleshoot the max file descriptors error, run the following command:

```
echo "$(whoami) - nofile 65535" | sudo tee -a /etc/security/limits.conf
```

To fix the `vm.max_map_count` error, run this command to increase the count to `262114`:

```
sudo sysctl -w vm.max_map_count=262144
```

## Next steps

If you want to try a GPU-accelerated cluster using AWS Inferentia with a pretrained HuggingFace model, see [Compiling and Deploying HuggingFace Pretrained BERT](https://awsdocs-neuron.readthedocs-hosted.com/en/latest/src/examples/pytorch/bert_tutorial/tutorial_pretrained_bert.html).

