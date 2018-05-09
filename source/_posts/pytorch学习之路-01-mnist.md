---
title: pytorch学习之路-01-mnist
date: 2018/05/09 8:43
comments: true
categories: 
- Pytorch
tags: 
- Pytorch
- mnist
---

# 说明

本文使用pytorch从mnsit原始图像开始学习并完成输入图像的预测

# 数据集介绍

mnsit是一个广泛使用的手写数字数据集，其包含60000张训练数据集，10000张测试数据集。数据集长这样。

![mnist数据](https://ws1.sinaimg.cn/large/75779dbdgy1fr4t4e8d0ej207305oaa8.jpg)

但是从网上下载的数据集一般不是jpg或png格式的,下载的数据长这样。
![mnist数据集格式](https://ws1.sinaimg.cn/large/75779dbdgy1fr4t57bnedj20em02k3yq.jpg)

如何将数据集转换成jpg或者png文件，请参考这篇博客 [mnist数据集转换](https://blog.csdn.net/qq_32166627/article/details/52640730)

# 自定义dataset

这里使用自定义的dataset来完成对mnsit数据集的加载,在pytorch中自定义dataset需要继承自`torch.utils.data.Dataset`，并且要实现`__init__`,`__getitem__`和`__len__`方法。

```python
class MyDataset(torch.utils.data.Dataset):
    def __init__(self, txt, data_shape,channel=3, transform=None, target_transform=None):
        '''

        :param txt:
        :param data_shape:
        :param channel:
        :param transform:
        :param target_transform:
        '''
        fh = open(txt, 'r')
        data = []
        for line in fh:
            line = line.strip('\n')
            line = line.rstrip()
            words = line.split()
            data.append((words[0], int(words[1])))
        self.data = data
        self.transform = transform
        self.target_transform = target_transform
        self.data_shape = data_shape
        self.channel = channel


    def __getitem__(self, index):
        img_path, label = self.data[index]
        if self.channel == 3:
            img = cv2.imread(img_path,1)
        else:
            img = cv2.imread(img_path, 0)
        img = cv2.resize(img, (self.data_shape[0], self.data_shape[1]))
        img = np.reshape(img,(self.data_shape[0], self.data_shape[1],self.channel))
        if self.transform is not None:
            img = self.transform(img)
        return img, label

    def __len__(self):
        return len(self.data)
```
# 网络

这里使用`torchvision`里自带的`alexnet`

# 训练

```python
import torch
import torchvision
import torch.utils.data as Data
from torchvision import transforms
from data_loader import MyDataset
import time
from tensorboardX import SummaryWriter

device = torch.device("cuda:0")
print('training with:', device)
num_epochs = 3
batch_size = 64

train_data = MyDataset(txt='/data/datasets/mnist/train.txt', data_shape=(227, 227), channel=3,
                       transform=transforms.ToTensor())
train_loader = Data.DataLoader(
    dataset=train_data, batch_size=batch_size, shuffle=True, num_workers=3)

model = torchvision.models.AlexNet(num_classes=10)
# 准备写tensorboard, 必须放在'.to(device)'之前，不然会报错
writer = SummaryWriter()
dummy_input = torch.autograd.Variable(torch.rand(1, 3, 227, 227))
writer.add_graph(model=model, input_to_model=(dummy_input, ))

model = model.to(device)

criterion = torch.nn.CrossEntropyLoss()
optimizer = torch.optim.Adam(model.parameters(), lr=0.001)
scheduler = torch.optim.lr_scheduler.StepLR(optimizer, 5, gamma=0.1)
model.train()

for epoch in range(num_epochs):
    scheduler.step()
    train_acc, train_loss = 0., 0.
    start = time.time()
    for i, (images, labels) in enumerate(train_loader):
        images, labels = images.to(device), labels.to(device)
        # Forward
        optimizer.zero_grad()
        out = model(images)
        # Backward
        loss = criterion(out, labels)
        loss.backward()
        optimizer.step()
        train_loss += loss.item()
        _, preds = torch.max(out.data, 1)
        correct = preds.eq(labels.data).sum().item()
        acc = correct / labels.size(0)
        train_acc += correct

        cur_step = epoch * (train_data.__len__()/batch_size)+i
        writer.add_scalar(tag='Train/loss', scalar_value=loss.item(), global_step=cur_step)
        writer.add_scalar(tag='Train/acc', scalar_value=acc, global_step=cur_step)
        # if i % 1 == 0:
        # print('Iteration: {}. Loss: {}. Accuracy: {}'.format(cur_step, loss.item(), acc))
    print('epoch [%d/%d], loss: %.4f, acc: %.4f, time:%0.4f, lr:%s' % (
        epoch + 1, num_epochs, train_loss /
        len(train_data), train_acc / len(train_loader.dataset),
        time.time() - start, str(scheduler.get_lr()[0])))
writer.close()
torch.save(model, 'AlexNet1.pkl')
```

在训练时有一些超参数设置如下
```sh
epoch = 3 # 数据集学习多少次
batch_size = 64 # 每次计算多少张图像
```

为了可视化训练结果，使用了[tensorboardX](https://github.com/lanpa/tensorboard-pytorch)包,
这个包将tensorflow的tensorboard功能弄到了pytorch里面。需要注意的是，可视化网络要在`model.to(device)`之后使用，不然会报错。

在终端中使用下面的命令来启动训练
```sh
python train.py
```

训练结果如下

![训练结果](https://ws1.sinaimg.cn/large/75779dbdgy1fr4tryk4hyj20f702jdfw.jpg)

可视化结果

loss
![loss](https://ws1.sinaimg.cn/large/75779dbdgy1fr4ttxkaxdj21540aqq77.jpg)

acc
![acc](https://ws1.sinaimg.cn/large/75779dbdgy1fr4ttxkaxdj21540aqq77.jpg)

网络结构图

![Alexnet网络结构图](https://ws1.sinaimg.cn/large/75779dbdgy1fr4tvaxhvmj20bo2omaet.jpg)

# 预测

拿到模型之后，我们来预测一下

```python
import torch
import torch.nn.functional as F
from torch.autograd import Variable
from torchvision import transforms
import time
import cv2
import os
import numpy as np
import argparse

class Pytorch_model:
    def __init__(self, model_path, img_shape, img_channel=3, gpu_id=None, classes_txt=None):
        self.gpu_id = gpu_id
        self.img_shape = img_shape
        self.img_channel = img_channel
        if self.gpu_id is not None and isinstance(self.gpu_id, int) and torch.cuda.is_available():
            self.device = torch.device("cuda:%s" %(self.gpu_id))
        else:
            self.device = torch.device("cpu")
        print(self.device)

        if self.gpu_id is not None and isinstance(self.gpu_id, int):
            self.use_gpu = True
        else:
            self.use_gpu = False

        if not self.use_gpu:
            self.net = torch.load(model_path, map_location=lambda storage, loc: storage.cpu())
        else:
            self.net = torch.load(model_path, map_location=lambda storage, loc: storage.cuda(gpu_id))
        self.net.eval()

        if classes_txt is not None:
            with open(classes_txt, 'r') as f:
                self.idx2label = dict(line.strip().split(' ') for line in f if line)
        else:
            self.idx2label = None

    def predict(self, img, is_numpy=False, topk=1):
        if len(self.img_shape) not in [2, 3] or self.img_channel not in [1, 3]:
            raise NotImplementedError

        if not is_numpy and self.img_channel in [1, 3]: # read image
            if os.path.exists(img):
                img = cv2.imread(img, 0 if self.img_channel == 1 else 1)
            else:
                return 'file is not exists'

        img = cv2.resize(img, (self.img_shape[0], self.img_shape[1]))
        if len(img.shape) == 2 and self.img_channel == 3:
            img = cv2.cvtColor(img, cv2.COLOR_GRAY2BGR)
        elif len(img.shape) == 3 and self.img_channel == 1:
            img = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)


        img = img.reshape([self.img_shape[0], self.img_shape[1], self.img_channel])
        tensor = transforms.ToTensor()(img)
        tensor = tensor.unsqueeze_(0)
        tensor = Variable(tensor)

        tensor = tensor.to(self.device)
        outputs = F.softmax(self.net(tensor),dim=1)
        result = torch.topk(outputs.data[0], k=topk)

        if self.device != "cpu":
            index = result[1].cpu().numpy().tolist()
            prob = result[0].cpu().numpy().tolist()
        else:
            index = result[1].numpy().tolist()
            prob = result[0].numpy().tolist()
        if self.idx2label is not None:
            label = []
            for idx in index:
                label.append(self.idx2label[str(idx)])
            result = label, prob
        else:
            result = index, prob
        return result


if __name__ == '__main__':
    parser = argparse.ArgumentParser()
    parser.add_argument('-f','--file', type=str, help='test image file')
    parser.add_argument('-m','--model', type=str, help='model used')
    args = parser.parse_args()
    img_path = args.file
    model_path = args.model
    img_shape = [227,227]
    img_channel = 3
     # test cpu speed
    model = Pytorch_model(model_path, img_shape=img_shape,img_channel=img_channel, classes_txt='labels.txt')
    start_cpu = time.time()
    epoch = 1
    for _ in range(epoch):
        start = time.time()
        result = model.predict(img_path,topk=3)
        print('device: cpu, result:%s, time: %.4f' % (str(result), time.time() - start))
    end_cpu = time.time()

    # test gpu speed
    model1 = Pytorch_model(model_path=model_path, img_shape=img_shape,img_channel=img_channel, gpu_id=0, classes_txt='labels.txt')
    start_gpu = time.time()
    for _ in range(epoch):
        start = time.time()
        result = model1.predict(img_path,topk=3)
        print('device: gpu, result:%s, time: %.4f' % (str(result), time.time() - start))
    end_gpu = time.time()
    print('cpu avg time: %.4f' % ((end_cpu - start_cpu) / epoch))
    print('gpu avg time: %.4f' % ((end_gpu - start_gpu) / epoch))
```

调用代码
```python
python predict.py -f /data/datasets/mnist/test/0/0_1.png -m AlexNet.pkl
```

使用的预测图像为

![预测图像](https://ws1.sinaimg.cn/large/75779dbdgy1fr4tyc9zt9j200k00k0ih.jpg)

预测结果如下，脚本输出了top3分别为

| 类别       | 置信度   |
| --------   | -----:  |
| 0          | 0.999999|
| 9          | 0.000002|
| 6          | 0.000002|

![预测结果](https://ws1.sinaimg.cn/large/75779dbdgy1fr4twpxk5jj20r803574h.jpg)