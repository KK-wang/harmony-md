# learn-1

## 1.什么是 VITS

VITS（Variational Inference with adversarial learning for end-to-end Text-to-Speech）是一种结合变分推理（variational inference）、标准化流（normalizing flows）和对抗训练的高表现力语音合成模型。VITS通过隐变量而非频谱串联起来语音合成中的声学模型和声码器，在隐变量上进行随机建模并利用随机时长预测器，提高了合成语音的多样性，输入同样的文本，能够合成不同声调和韵律的语音。

## 2.什么是 google colab

Colaboratory（简称 Colab），是Google公司的一款产品，可以浏览器中编写和执行 Python 代码。

最重要的是，Colab可以给我们分配**免费的 GPU** 使用。这真的对我们这种**没显卡**还要做深度学习的科研民工的福音！

并且Colab 无需任何配置，常用的库基本上都有，默认使用的深度学习的库是keras。

google colab 的官网为：https://research.google.com/colaboratory/faq.html?hl=zh-cn。

## 3.什么是 jupyter

Jupyter Notebook是基于网页的用于交互计算的应用程序。其可被应用于全过程计算：开发、文档编写、运行代码和展示结果。

简而言之，Jupyter Notebook是以网页的形式打开，可以在网页页面中**直接编写代码**和**运行代码**，代码的**运行结果**也会直接在代码块下显示的程序。如在编程过程中需要编写说明文档，可在同一个页面中直接编写，便于作及时的说明和解释。

> 前端编写代码，后端运行代码，有点像 leetcode。

## 4.VITS 教程

https://github.com/Plachtaa/VITS-fast-fine-tuning/blob/main/README_ZH.md。

## 5.什么是内网穿透？

内网穿透，即 NAT（Network Address Translator）穿透，是**指计算机在内网（局域网）内使用私有IP地址，在连接外网（互联网）时使用全局IP地址的技术**。

举个例子：比如我在实验室配置了一个服务器 Server A，当我在实验室的时候，就可以通过自己的笔记本使用SSH连接【**因为我和服务器处于一个局域网**】，当我回宿舍以后，就没有办法直接使用SSH连接了【**因为我和服务器不在一个局域网**】，这个时候就需要进行NAT穿透，让我在宿舍也可以使用SSH连接Server A。

一般来讲，内网穿透主要用于**外网/公网访问内网**。除了上面的例子外，还有家用 NAS 的例子：

家里有NAS作文件服务器，在单位或出差需要访问家里NAS时，就需要通过一些设置才可以实现访问：

* 如果家里宽带有公网IP（现在运营商分配的一般是内网 IP），那么简单配置下DDNS服务，然后通过域名就可以在外网访问。
* 如果宽带没有没有公网IP，只是大内网IP，那只能通过穿透方式实现外网的访问。



