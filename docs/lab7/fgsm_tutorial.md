# 对抗性样本攻击实验

本实验根据 PyTorch 官网教程中 [Adversarial Example Generation](https://pytorch.org/tutorials/beginner/fgsm_tutorial.html#sphx-glr-beginner-fgsm-tutorial-py) 章节内容完整实现快速梯度符号攻击 （ Fast Gradient Sign Attack ） 算法。

## 0 实验环境

本次实验在Pytorch下完成，我们使用实验室（T2210）已经部署安装好的Anaconda集成环境。
1. 选中桌面的Anaconda图标，右键->以管理员身份运行；

<center><img src="../assets/1-1.png" width = 600></center>

2. 进入Anaconda后，在base虚拟环境中运行网页版Jupyter；

<center><img src="../assets/1-2.png" width = 600></center>

3. 在网页版Jupyter中新建一个Python3文件并重命名；

<center><img src="../assets/1-3.png" width = 600></center>

<center><img src="../assets/1-4.png" width = 600></center>

4. 最后就可以在网页版Jupyter编写并运行代码了，常用命令如下；

<center><img src="../assets/1-5.png" width = 600></center>


## 1 Overview

这是一篇机器学习中对抗样本攻击的入门实验教程。机器学习模型非常有效，大家都在致力于研究不断推动机器学习模型更快、更准确、更高效。然而，设计和训练模型一个经常被忽视的方面是安全性和健壮性，特别是在面对希望愚弄模型的对手时。

本实验教程将提高您对机器学习模型的安全漏洞的认识，并将初步了解对抗性机器学习话题， 在图像中添加不可察觉的扰动会导致截然不同的模型性能。鉴于这是一个实验教程，我们将通过图像分类器上的示例来探讨这个主题。本次实验使用最流行的攻击方法之一，Fast Gradient Sign Attack 来欺骗MNIST分类器。

**Fast Gradient Sign Attack(FGSM)** 快速梯度符号攻击是由 Goodfellow 等人在[Explaining and Harnessing Adversarial Examples] (https://arxiv.org/abs/1412.6572)中提出，是一种简单但是有效的对抗样本生成算法。它旨在通过在梯度方向上添加一i个扰动生成的图像，能够使得神经网络模型被错误分类。利用模型学习的方式和渐变来攻击神经网络。这个想法很简单，攻击调整输入数据以基于相同的反向传播梯度来最大化损失，而不是通过基于反向传播的梯度调整权重来最小化损失。具体来说，针对每一个输入 x ，我们模型（参数为 W ）都可以得到一个输出（ y_true )。利用此输出 y_true 和 y_true' 属于的类别，可以对输入求梯度。接着根据梯度的方向来对输入进行微小的扰动, 则可以最大化的改变输出。

<center><img src="../assets/0-1.png" width = 600></center>

**MINST数据集** MNIST是“Modified National Institute of Standards and Technology database”的简写，从MNIST的全称可以看出，该数据集来自美国国家标准与技术研究所，数据集合是由来自 250 个不同人手写的数字构成, 其中 50% 是高中学生, 50% 来自人口普查局 (the Census Bureau) 的工作人员，作为著名的手写数字机器视觉数据库，MNIST被广泛应用在各种图像分类与识别任务中。MNIST的发起人是当前著名深度学习学者（2019年图灵奖得主）Yann LeCun。

MNIST的训练集合中共有60 000个手写训练样本，测试集合中有10 000个测试样本。但测试集合中的数据并不“单纯”，其前5000个样本完全是从训练集合中抽取的（可作为验证集合），后5000个样本才是真正异于训练集合的手写数字，属于真正的测试集合。

在MNIST中，每个样本图像由28×28像素的手写数字组成。这些图片只包含灰度信息和它对应的标签（label）信息（即正确的手写数字），如下图所示。

<center><img src="../assets/0-5.png" width = 800></center>


## 2 威胁模型

**对抗样本**指的是攻击者在数据集原始输入样本通过添加人类无法察觉的细微扰动来形成新的输入样本，导致模型以高置信度给出一个错误的输出。它是在模型输入端实施攻击，攻击者在模型加入特定的扰动，输入微小扰动，输出巨大变化。

对抗样本最早由Szegedy等人提出，通过在数据集中添加轻微扰动干扰原始样本，导致模型以高置信度给出错误输出。在许多情况下，人类不会察觉原始样本和对抗样本之间的差异，但是神经网络会做出很大差异性的错误预测。

具体来说，我们将使用快速梯度符号攻击（FGSM），以愚弄MNIST分类器。有许多类别的对抗性攻击，每种攻击都有不同的目标和对攻击者知识的假设。但是，总体而言，首要目标是向输入数据添加最少量的扰动，从而导致所需的错误分类。攻击者有几种假设，其中两种是：白盒和黑盒。

**白盒攻击** 假设攻击者具有对模型的全部了解和访问权限，包括体系结构、输入、输出和权重。
**黑盒攻击** 假定攻击者只能访问模型的输入和输出，而对底层体系结构或权重一无所知。

根据对抗特异性可以分为针对目标攻击和非针对目标攻击，包括错误分类和源/目标错误分类。

**错误分类的目标** 意味着对手只希望输出分类是错误的，但并不关心新分类是什么。
**源/目标错误分类** 意味着对手想要更改原始属于特定源类的图像，以便将其分类为特定目标类。

FGSM 攻击是 白盒攻击，其目标是错误分类。

## 3 FGSM 快速梯度符号攻击

迄今为止最早和最流行的对抗性攻击之一被称为快速梯度符号攻击(FGSM)。它旨在通过利用模型学习的方式和渐变来攻击神经网络。这个想法很简单，攻击调整输入数据以基于相同的反向传播梯度来最大化损失，而不是通过基于反向传播的梯度调整权重来最小化损失。换句话说，攻击使用损失的梯度 w.r.t 输入数据，然后调整输入数据以最大化损失。
以最常见的图像识别为例，我们希望在原始图片上做肉眼难以识别的修改，但是却可以让图像识别模型产生误判。假设图片原始数据为x，图片识别的结果为 y，原始图像上细微的变化肉眼难以识别, 模型最终识别错误，而我们肉眼却能正确识别。比如下图中加入对抗样本后模型识别结果为长臂猿，而我们肉眼却能正确识别出为熊猫。

<center><img src="../assets/0-2.png" width = 600></center>



在我们进入代码之前，让我们看一下著名的 FGSM熊猫示例和摘录一些符号。

从图中可以看出，$\mathbf{x}$ 是原始输入图像，可以正确归类为“熊猫”， $y$ 是 $\mathbf{x}$ 的地面真相标签， $\mathbf{\theta}$ 表示模型参数，$J(\mathbf{\theta}, \mathbf{x}, y)$  是用于训练网络的损失。攻击者将梯度反向传播输入数据进行计算 $\nabla_{x} J(\mathbf{\theta}, \mathbf{x}, y)$ 。然后，它调整输入数据（$\epsilon$ or $0.007$ 中的图片）在方向（即 $sign(\nabla_{x} J(\mathbf{\theta}, \mathbf{x}, y))$）最大化损失。由此产生的扰动图像 $x'$  ，则被目标网络会错误地见该图片分类为“长臂猿”，但是实际上它仍然显然是一只“熊猫”。

下面进入代码实现

```python
from __future__ import print_function
import torch
import torch.nn as nn
import torch.nn.functional as F
import torch.optim as optim
from torchvision import datasets, transforms
import numpy as np
import matplotlib.pyplot as plt

# NOTE: This is a hack to get around "User-agent" limitations when downloading MNIST datasets
#       see, https://github.com/pytorch/vision/issues/3497 for more information
from six.moves import urllib
opener = urllib.request.build_opener()
opener.addheaders = [('User-agent', 'Mozilla/5.0')]
urllib.request.install_opener(opener)
```

## 4 FGSM 实现

在本节中，我们将讨论本教程的输入参数，定义受攻击的模型，然后对攻击进行编码并运行一些测试。

### 4.1 输入

本教程只有三个输入，定义为：

-  ``epsilons`` - 用于运行的 $epsilon$ 值的列表。重要的是在列表中保留 0，因为它表示模型原始测试集上的性能。此外，直观地说，我们会
预期 $epsilon$ 越大，扰动越明显但攻击在退化模型方面越有效准确性。由于此处的数据范围是  $[0,1]$ ，因此没有 $epsilon$ 值应超过 1。

-  ``pretrained_model`` - 通往预训练的MNIST模型的路径
   [pytorch/examples/mnist](https://github.com/pytorch/examples/tree/master/mnist)_.
   可以从下面链接下载模型（T2210实验室环境已经下载完成，在C:\user\Administrator\data路径和D:\data路径下都有） [下载模型](https://drive.google.com/drive/folders/1fn83DF14tWmit0RTKWRhPq5uVXt73e0h?usp=sharing)_.

-  ``use_cuda`` - 使用 CUDA 或不使用。本次实验中使用CPU速度也比较快，训练集比较小，这里我们代码改成False, 如果你在cuda的GPU上跑该程序。


```python
epsilons = [0, .05, .1, .15, .2, .25, .3]
pretrained_model = "data/lenet_mnist_model.pth"
use_cuda=True
```

任务1：请大家修改epsilons的类别值，然后测试比较结果。


### 4.2 受攻击的模型

如前所述，受到攻击的模型与来自[pytorch/examples/mnist](https://github.com/pytorch/examples/tree/master/mnist).
您可以训练并保存自己的MNIST模型，也可以下载并使用提供的模型(已下载完成)。此处的 Net 定义和测试数据加载器具有是从 MNIST 示例复制的。

本节的目的是定义模型和数据加载器，然后初始化模型并加载预先训练的weight。


```python
# LeNet Model definition
class Net(nn.Module):
    def __init__(self):
        super(Net, self).__init__()
        self.conv1 = nn.Conv2d(1, 10, kernel_size=5)
        self.conv2 = nn.Conv2d(10, 20, kernel_size=5)
        self.conv2_drop = nn.Dropout2d()
        self.fc1 = nn.Linear(320, 50)
        self.fc2 = nn.Linear(50, 10)

    def forward(self, x):
        x = F.relu(F.max_pool2d(self.conv1(x), 2))
        x = F.relu(F.max_pool2d(self.conv2_drop(self.conv2(x)), 2))
        x = x.view(-1, 320)
        x = F.relu(self.fc1(x))
        x = F.dropout(x, training=self.training)
        x = self.fc2(x)
        return F.log_softmax(x, dim=1)

# MNIST Test dataset and dataloader declaration
test_loader = torch.utils.data.DataLoader(
    datasets.MNIST('./data', train=False, download=True, transform=transforms.Compose([
            transforms.ToTensor(),
            ])), 
        batch_size=1, shuffle=True)

# Define what device we are using
print("CUDA Available: ",torch.cuda.is_available())
device = torch.device("cuda" if (use_cuda and torch.cuda.is_available()) else "cpu")

# Initialize the network
model = Net().to(device)

# Load the pretrained model
model.load_state_dict(torch.load(pretrained_model, map_location='cpu'))

# Set the model in evaluation mode. In this case this is for the Dropout layers
model.eval()
```



### 4.3 FGSM 攻击函数

现在，我们可以通过以下方式定义创建对抗性示例的函数：

扰动原始输入。fgsm_attack 函数需要三个输入，图像是原始干净的图像 ($x$), *epsilon* 是像素级扰动量 ($\epsilon$), 和 *data_grad* 是输入图像的损耗的梯度(w.r.t) 
($\nabla_{x} J(\mathbf{\theta}, \mathbf{x}, y)$). 然后fgsm_attack函数通过如下的方式来产生：

\begin{align}perturbed\_image = image + epsilon*sign(data\_grad) = x + \epsilon * sign(\nabla_{x} J(\mathbf{\theta}, \mathbf{x}, y))\end{align}

最后，为了保持数据的原始范围，扰动图像被裁剪到  $[0,1]$ 范围内。


```python
# FGSM attack code
def fgsm_attack(image, epsilon, data_grad):
    # Collect the element-wise sign of the data gradient
    //todo
    # Create the perturbed image by adjusting each pixel of the input image
    //todo
    # Adding clipping to maintain [0,1] range
    //todo
    # Return the perturbed image
    return perturbed_image
```

### 4.4 测试攻击效果函数
 
最后，本教程的核心结果来自测试攻击的结果。每次调用此测试函数都会在 MNIST 测试集上执行一个完整的测试步骤并报告最终精度。但是，请注意此函数还有一个 *epsilon* 参数输入，“测试”函数报告受攻击的模型的准确性来自  $\epsilon$ 的输入。更具体地说，对于测试集中的每个样本，该函数计算输入数据 ($data\_grad$) 的损失会产生扰动带有 “fgsm_attack”($perturbed\_data$) 的图像，然后检查以查看如果是对抗性的。除了测试模型的精度，该函数还保存并返回一些成功的对抗性示例将在以后可视化。


```python
def test( model, device, test_loader, epsilon ):

    # Accuracy counter
    correct = 0
    adv_examples = []

    # Loop over all examples in test set
    for data, target in test_loader:

        # Send the data and label to the device
        data, target = data.to(device), target.to(device)

        # Set requires_grad attribute of tensor. Important for Attack
        data.requires_grad = True

        # Forward pass the data through the model
        output = model(data)
        init_pred = output.max(1, keepdim=True)[1] # get the index of the max log-probability

        # If the initial prediction is wrong, don't bother attacking, just move on
        if init_pred.item() != target.item():
            continue

        # Calculate the loss
        loss = F.nll_loss(output, target)

        # Zero all existing gradients
        model.zero_grad()

        # Calculate gradients of model in backward pass
        loss.backward()

        # Collect ``datagrad``
        data_grad = data.grad.data

        # Call FGSM Attack
        perturbed_data = fgsm_attack(data, epsilon, data_grad)

        # Re-classify the perturbed image
        output = model(perturbed_data)

        # Check for success
        final_pred = output.max(1, keepdim=True)[1] # get the index of the max log-probability
        if final_pred.item() == target.item():
            correct += 1
            # Special case for saving 0 epsilon examples
            if (epsilon == 0) and (len(adv_examples) < 5):
                adv_ex = perturbed_data.squeeze().detach().cpu().numpy()
                adv_examples.append( (init_pred.item(), final_pred.item(), adv_ex) )
        else:
            # Save some adv examples for visualization later
            if len(adv_examples) < 5:
                adv_ex = perturbed_data.squeeze().detach().cpu().numpy()
                adv_examples.append( (init_pred.item(), final_pred.item(), adv_ex) )

    # Calculate final accuracy for this epsilon
    final_acc = correct/float(len(test_loader))
    print("Epsilon: {}\tTest Accuracy = {} / {} = {}".format(epsilon, correct, len(test_loader), final_acc))

    # Return the accuracy and an adversarial example
    return final_acc, adv_examples
```

### 4.5 实施攻击

本实验教程的最后一部分是实施攻击。在这里我们为 *epsilons* 输入中的每个 $\epsilon$ 值运行完整的测试步骤。

对于每一个 *epsilon* ，我们还保存了最终的精度和一些成功的结果。 在接下来的章节中，我们将列举一些对抗性的例子。注意随着 *epsilon*  值的增加，模型精度降低。而且注意 $\epsilon=0$ 表示原始测试精度，即不进行攻击。


```python
accuracies = []
examples = []

# Run test for each epsilon
for eps in epsilons:
    acc, ex = test(model, device, test_loader, eps)
    accuracies.append(acc)
    examples.append(ex)
```

## 5 结果分析

### 5.1 准确性 vs Epsilon

第一个结果是准确性与 epsilon 的关系图。
如前所述，早些时候，随着 epsilon 的增加，我们预计测试精度会降低。
这是因为更大的 epsilon 意味着我们在方向将最大化损失。请注意，即使 epsilon 值是线性间隔的，准确性也不是线性的。
例如，$\epsilon=0.05$ 时的精度仅低约4%
大于 $\epsilon=0$ ，但 $\epsilon=0.2$ 时的准确率为 25%，低于  $\epsilon=0.15$。另外，请注意模型的准确性达到 10 类分类器的随机精度，介于 $\epsilon=0.25$ 和 $\epsilon=0.3$。

<center><img src="../assets/0-3.png" width = 500></center>

代码如下：

```python
plt.figure(figsize=(5,5))
plt.plot(epsilons, accuracies, "*-")
plt.yticks(np.arange(0, 1.1, step=0.1))
plt.xticks(np.arange(0, .35, step=0.05))
plt.title("Accuracy vs Epsilon")
plt.xlabel("Epsilon")
plt.ylabel("Accuracy")
plt.show()
```

### 5.2 对抗样本实例

还记得没有免费午餐的想法吗？在这种情况下，随着 epsilon 的增加，测试正确率降低但是扰动变得更加容易感知。

实际上，攻击者必须考虑权衡。在这里，我们在每个 epsilon 上展示一些成功的对抗示例价值。图的每一行都显示不同的 epsilon 值。第一个行是 $\epsilon=0$ 示例，表示原始“干净”的图像，无扰动。每个图像的标题显示原始分类->对抗性分类。请注意，扰动在 $\epsilon=0.15$ 时开始变得明显，并且在 $\epsilon=0.3$ 变得很明显了。

然而，在所有情况下，人类都是仍然能够识别正确的类。

<center><img src="../assets/0-4.png" width = 500></center>

```python
# Plot several examples of adversarial samples at each epsilon
cnt = 0
plt.figure(figsize=(8,10))
for i in range(len(epsilons)):
    for j in range(len(examples[i])):
        cnt += 1
        plt.subplot(len(epsilons),len(examples[0]),cnt)
        plt.xticks([], [])
        plt.yticks([], [])
        if j == 0:
            plt.ylabel("Eps: {}".format(epsilons[i]), fontsize=14)
        orig,adv,ex = examples[i][j]
        plt.title("{} -> {}".format(orig, adv))
        plt.imshow(ex, cmap="gray")
plt.tight_layout()
plt.show()
```

## 6 接下来怎么做

希望本实验教程对对抗性机器学习的主题有一些帮助。 这种攻击代表了对抗性攻击研究的开始，并且由于后续有许多关于如何攻击和保护机器学习模型免受对手攻击的想法。 在NIPS2017年有一个对抗性攻击和防御竞争，本文描述了竞争中使用的许多方法[Adversarial Attacks and Defences Competition](https://arxiv.org/pdf/1804.00097.pdf)对抗性攻击和防御竞争。


