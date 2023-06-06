```python
# For tips on running notebooks in Google Colab, see
# https://pytorch.org/tutorials/beginner/colab
%matplotlib inline
```

# 对抗性样本攻击实验

本实验根据 PyTorch 官网教程中 [Adversarial Example Generation](https://pytorch.org/tutorials/beginner/fgsm_tutorial.html#sphx-glr-beginner-fgsm-tutorial-py) 章节内容，完整实现 Fast Gradient Sign Attack (FGSM) 算法。

**Author:** [Nathan Inkawhich](https://github.com/inkawhich)_

如果你正在阅读这篇文章，希望你能理解一些机器学习模型是多么有效。研究不断推动ML模型更快、更准确、更高效。然而，设计和训练模型的一个经常被忽视的方面是安全性和健壮性，特别是在面对希望愚弄模型的对手时。

本教程将提高您对ML模型的安全漏洞的认识，并将深入了解对抗性机器学习的热门话题。 大家可能会惊讶地发现，在图像中添加不可察觉的扰动会导致截然不同的模型性能。鉴于这是一个教程，我们将通过图像分类器上的示例来探讨这个主题。具体来说，我们将使用第一种也是最流行的攻击方法之一，Fast Gradient Sign Attack(FGSM)来欺骗MNIST分类器。


## 威胁模型

添加不可察觉对图像的扰动可以导致截然不同的模型性能。

我们将探讨该主题通过图像分类器上的示例。具体来说，我们将使用其中之一第一种也是最流行的攻击方法，快速梯度符号攻击（FGSM），以愚弄MNIST分类器。

有许多类别的对抗性攻击，每种攻击都有不同的目标和对攻击者知识的假设。但是，总体而言，首要目标是向输入数据添加最少量的扰动，从而导致所需的错误分类。攻击者的知识有几种假设，其中两种是：白盒和黑匣子。

白盒攻击假设攻击者具有对模型的全部了解和访问权限，包括体系结构、输入、输出和权重。
黑盒攻击假定攻击者只能访问模型的输入和输出，而对底层体系结构或权重一无所知。
还有几种类型的目标，包括错误分类和源/目标错误分类。

错误分类的目标意味着对手只希望输出分类是错误的，但并不关心新分类是什么。
源/目标错误分类意味着对手想要更改原始属于特定源类的图像，以便将其分类为特定目标类。
在这种情况下，FGSM 攻击是 白盒 攻击，其目标是错误分类。有了这些背景信息，我们现在就可以详细讨论攻击。

## FGSM 快速梯度符号攻击

迄今为止最早和最流行的对抗性攻击之一被称为快速梯度符号攻击(FGSM)，并由Goodfellow等人描述。
它旨在通过以下方式攻击神经网络：利用他们的学习方式，梯度。这个想法很简单，而是而不是通过根据反向传播梯度，攻击调整输入数据以最大化基于相同反向传播梯度的损失。换句话说，攻击使用损失的梯度 w.r.t 输入数据，然后调整输入数据以最大化损失。

在我们跳入代码之前，让我们看看著名的FGSM 熊猫示例并提取一些符号。

[FGSM](https://arxiv.org/abs/1412.6572)_ panda example and extract
some notation.

.. figure:: /_static/img/fgsm_panda_image.png
   :alt: fgsm_panda_image

在我们进入代码之前，让我们看一下著名的 FGSM熊猫示例 和摘录一些符号。

从图中可以看出，x \mathbf{x}x 是原始输入图像正确归类为“熊猫”，y yy是地面真相标签for x \mathbf{x}x， θ \mathbf{\theta}θ 表示模型参数，J （ θ ， x ， y ） J（\mathbf{\theta}， \mathbf{x}， y）J（θ，x，y） 是损失用于训练网络。攻击反向传播梯度返回输入数据进行计算∇ x J （ θ ， x ， y ） \nabla_{x} J（\mathbf{\theta}， \mathbf{x}， y）∇xJ（θ，x，y）.然后，它调整按一小步输入数据（ϵ \epsilonϵ 或 0.007 0.0070.007中的图片）在方向（即s i g n （ ∇ x J （ θ ， x ， y ） ） sign（\nabla_{x} J（\mathbf{\theta}， \mathbf{x}， y））sign（∇xJ（θ，x，y）））最大化损失。由此产生的扰动图像x ′ x'x ′ ，则被目标网络错误地分类为“长臂猿”，当它仍然显然是一只“熊猫”。

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

## FGSM 实现

在本节中，我们将讨论本教程的输入参数，定义受攻击的模型，然后对攻击进行编码并运行一些测试。

### 输入

本教程只有三个输入，定义为：

-  ``epsilons`` - 用于运行的 ϵ \epsilonϵ值的列表。重要的是在列表中保留 0，因为它表示模型原始测试集上的性能。此外，直观地说，我们会
预期 ϵ \epsilonϵ越大，扰动越明显但攻击在退化模型方面越有效准确性。由于此处的数据范围是 [ 0 ， 1 ] [0，1][0，1]，因此没有 epsilon 值应超过 1。

-  ``pretrained_model`` - 通往预训练的MNIST模型的路径
   [pytorch/examples/mnist](https://github.com/pytorch/examples/tree/master/mnist)_.
   For simplicity, download the pretrained model [here](https://drive.google.com/drive/folders/1fn83DF14tWmit0RTKWRhPq5uVXt73e0h?usp=sharing)_.

-  ``use_cuda`` - 使用 CUDA 或不使用。本次实验中使用CPU速度也比较快，训练集比较小。


```python
epsilons = [0, .05, .1, .15, .2, .25, .3]
pretrained_model = "data/lenet_mnist_model.pth"
use_cuda=True
```

### 受攻击的模型

如前所述，受到攻击的模型与来自[pytorch/examples/mnist](https://github.com/pytorch/examples/tree/master/mnist)_.
您可以训练并保存自己的MNIST模型，也可以下载并使用提供的模型。此处的 Net 定义和测试数据加载器具有是从 MNIST 示例复制的。

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

### FGSM 攻击函数

现在，我们可以通过以下方式定义创建对抗性示例的函数：

扰动原始输入。fgsm_attack函数需要三个输入，图像是原始干净的图像 ($x$), *epsilon* 是像素级扰动量 ($\epsilon$), 和 *data_grad* 是输入图像的损耗的梯度 w.r.t
($\nabla_{x} J(\mathbf{\theta}, \mathbf{x}, y)$). 功能
然后创建扰动图像作为

\begin{align}perturbed\_image = image + epsilon*sign(data\_grad) = x + \epsilon * sign(\nabla_{x} J(\mathbf{\theta}, \mathbf{x}, y))\end{align}

最后，为了保持数据的原始范围，扰动图像被裁剪到  $[0,1]$ 范围内。


```python
# FGSM attack code
def fgsm_attack(image, epsilon, data_grad):
    # Collect the element-wise sign of the data gradient
    sign_data_grad = data_grad.sign()
    # Create the perturbed image by adjusting each pixel of the input image
    perturbed_image = image + epsilon*sign_data_grad
    # Adding clipping to maintain [0,1] range
    perturbed_image = torch.clamp(perturbed_image, 0, 1)
    # Return the perturbed image
    return perturbed_image
```

### 测试攻击效果函数
 
最后，本教程的核心结果来自 ``test``。每次调用此测试函数都会执行一个完整的测试步骤MNIST测试集并报告最终精度。但是，请注意此函数还采用 *epsilon* 输入。这是因为“测试”函数报告受攻击的模型的准确性来自具有实力的对手  $\epsilon$ 。更具体地说，对于测试集中的每个样本，该函数计算输入数据 ($data\_grad$) 的损失会产生扰动带有“fgsm_attack”($perturbed\_data$) 的图像，然后检查以查看如果令人不安的例子是对抗性的。除了测试模型的精度，该函数还保存并返回一些成功的对抗性示例将在以后可视化。


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

### 实施攻击

实现的最后一部分是实际运行攻击。在这里我们为 *epsilons* 输入中的每个 $\epsilon$ 值运行完整的测试步骤。

对于每一个 ε，我们还保存了最终的精度和一些成功的结果.在接下来的章节中，我们将列举一些对抗性的例子。注意随着ε值的增加，模型精度降低。而且注意 $\epsilon=0$ 表示原始测试精度，即不进行攻击。


```python
accuracies = []
examples = []

# Run test for each epsilon
for eps in epsilons:
    acc, ex = test(model, device, test_loader, eps)
    accuracies.append(acc)
    examples.append(ex)
```

## 结果分析

### 准确性 vs Epsilon

第一个结果是准确性与 epsilon 的关系图。

如前所述，早些时候，随着 epsilon 的增加，我们预计测试精度会降低。

这是因为更大的 epsilon 意味着我们在方向将最大化损失。请注意，即使 epsilon 值是线性间隔的，准确性也不是线性的。

例如，$\epsilon=0.05$ 时的精度仅低约4%
大于 $\epsilon=0$ ，但 $\epsilon=0.2$ 时的准确率为 25%，低于  $\epsilon=0.15$。另外，请注意模型的准确性达到 10 类分类器的随机精度，介于 $\epsilon=0.25$ 和 $\epsilon=0.3$。


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

### 对抗样本实例

还记得没有免费午餐的想法吗？在这种情况下，随着 epsilon 的增加，测试正确率降低但是扰动变得更加容易感知。

实际上，攻击者必须考虑权衡。在这里，我们在每个 epsilon 上展示一些成功的对抗示例价值。图的每一行都显示不同的 epsilon 值。第一个行是 $\epsilon=0$ 示例，表示原始“干净”的图像，无扰动。每个图像的标题显示原始分类 - >对抗性分类。请注意，扰动在 $\epsilon=0.15$ 时开始变得明显，并且很明显在 $\epsilon=0.3$。

然而，在所有情况下，人类都是仍然能够识别正确的类。



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

## 接下来怎么做

希望本教程对对抗性机器学习的主题有一些见解。 从这里有许多潜在的方向要走。 这种攻击代表了对抗性攻击研究的开始，并且由于后续有许多关于如何攻击和保护ML模型免受对手攻击的想法。 事实上，在NIPS2017年有一个对抗性攻击和防御竞争，本文描述了竞争中使用的许多方法： [Adversarial Attacks and Defences Competition](https://arxiv.org/pdf/1804.00097.pdf).对抗性攻击和防御竞争。 在防御方面的工作也导致了这样的想法，即使机器学习模型在一般情况下更加健壮，既自然地令人不安，又有对抗性的输入。

另一个方向是不同领域的对抗性攻击和防御。 对抗性研究不限于图像领域，请查看对语音到文本模型的这种攻击。尝试实现与NIPS2017年竞争不同的攻击，并看看它与FGSM有何不同。 然后，试着保护模型免受你自己的攻击。

