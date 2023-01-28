# U2-Net

<script type="text/javascript" src="/js/src/bai.js"></script>


## 简介
设计了一个简单而强大的深度网络架构U2-Net，用于显著目标检测(SOD)。我们的U2-Net的体系结构是一个两层嵌套的U结构。

该设计有以下两点优势：
（1）它能够捕捉更多的上下文信息，因为提出了RSU(ReSidual U-blocks)结构，融合了不同尺度的感受野的特征；
（2）它增加了整个架构的深度但并没有显著增加计算成本，因为在这些RSU块中使用了池化操作。

这种架构使我们能够从头开始训练深度网络，而无需使用图像分类任务中的backbone。



### 网络架构
![](http://image.xpshuai.cn/20221014172349.png)

其中：
![](http://image.xpshuai.cn/20221014172414.png)

![](http://image.xpshuai.cn/20221014172427.png)


### RSU结构(ReSidual U-blocks)
![](http://image.xpshuai.cn/20221014172814.png)

![](http://image.xpshuai.cn/20221014172943.png)



**U型嵌套结构：**
![](http://image.xpshuai.cn/20221014173133.png)




### loss
由七个loss相加（一共六层，每一层一个loss。再加上最后mask融合时的一个loss）。







## 代码
> 参考的up主的代码：https://github.com/WZMIAOMIAO/deep-learning-for-image-processing/tree/master/pytorch_segmentation/u2net
> 替换为自己的遥感影像建筑物提取的数据集，简单测试了一下能跑通，代码有点乱还没改

整体目录如下：
![](http://image.xpshuai.cn/20221014164356.png)

my_dataset.py
```python
import os
import torch.utils.data as data
import torchvision
from PIL import Image


class DUTSDataset(data.Dataset):
    def __init__(self, root: str, train: bool = True, transform=None):
        self.trans = torchvision.transforms.Compose([
            torchvision.transforms.Resize([256, 256]),
            torchvision.transforms.ToTensor()
        ])
        # 当时简单测试，所以
        self.images_dir = "/home/xps/code/RemoteAICode/pytorch_segement/AttentionUnetTest/dataset/JPEGImages"
        self.masks_dir = "/home/xps/code/RemoteAICode/pytorch_segement/AttentionUnetTest/dataset/SegmentationClass"
        self.ids = os.listdir(self.images_dir)
        self.images_fps = [os.path.join(self.images_dir, image_id) for image_id in self.ids]
        self.masks_fps = [os.path.join(self.masks_dir, image_id) for image_id in self.ids]

    def __getitem__(self, idx):
        image = self.trans(Image.open(self.images_fps[idx]))
        mask = self.trans(Image.open(self.masks_fps[idx]))

        return image, mask

    def __len__(self):
        return len(self.ids)

```


model.py
```python
from typing import Union, List
import torch
import torch.nn as nn
import torch.nn.functional as F


class ConvBNReLU(nn.Module):
    """这三个经常一起使用"""
    def __init__(self, in_ch: int, out_ch: int, kernel_size: int = 3, dilation: int = 1):   # dilation>1代表是膨胀卷积
        super().__init__()

        padding = kernel_size // 2 if dilation == 1 else dilation
        self.conv = nn.Conv2d(in_ch, out_ch, kernel_size, padding=padding, dilation=dilation, bias=False)
        self.bn = nn.BatchNorm2d(out_ch)
        self.relu = nn.ReLU(inplace=True)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        return self.relu(self.bn(self.conv(x)))


class DownConvBNReLU(ConvBNReLU):
    """Encoder部分的下采样，卷积，BN，ReLU"""
    def __init__(self, in_ch: int, out_ch: int, kernel_size: int = 3, dilation: int = 1, flag: bool = True):
        super().__init__(in_ch, out_ch, kernel_size, dilation)
        self.down_flag = flag   # 是否启用下采样。默认为True

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        if self.down_flag:
            x = F.max_pool2d(x, kernel_size=2, stride=2, ceil_mode=True)    # 两倍下采样

        return self.relu(self.bn(self.conv(x)))


class UpConvBNReLU(ConvBNReLU):
    """Decoder部分的上采样，卷积，BN，ReLU"""
    def __init__(self, in_ch: int, out_ch: int, kernel_size: int = 3, dilation: int = 1, flag: bool = True):
        super().__init__(in_ch, out_ch, kernel_size, dilation)
        self.up_flag = flag

    def forward(self, x1: torch.Tensor, x2: torch.Tensor) -> torch.Tensor:  # 两个tensor进行拼接
        if self.up_flag:
            x1 = F.interpolate(x1, size=x2.shape[2:], mode='bilinear', align_corners=False)  # 这里采用双线性插值。x2是encoder输出的，.shape[2:]对应
        return self.relu(self.bn(self.conv(torch.cat([x1, x2], dim=1))))


class RSU(nn.Module):
    """通用的RSU模块
    height：深度，传入不同的height来实现RSU7..6..5...4
    ![20221013212413](http://image.xpshuai.cn/20221013212413.png)
    """
    def __init__(self, height: int, in_ch: int, mid_ch: int, out_ch: int):
        super().__init__()

        assert height >= 2
        self.conv_in = ConvBNReLU(in_ch, out_ch)

        encode_list = [DownConvBNReLU(out_ch, mid_ch, flag=False)]  # 最开始对应的是encoder中比较特殊的简单的那个模块(左上角)
        decode_list = [UpConvBNReLU(mid_ch * 2, mid_ch, flag=False)]  # 最开始对应的是decoder中比较特殊的简单的那个模块(右下角)
        for i in range(height - 2):  # height - 2就是其中有上下采样的模块的数量
            encode_list.append(DownConvBNReLU(mid_ch, mid_ch))
            decode_list.append(UpConvBNReLU(mid_ch * 2, mid_ch if i < height - 3 else out_ch))  # 除了最后一个模块之外的，其他都是mid_ch

        encode_list.append(ConvBNReLU(mid_ch, mid_ch, dilation=2))  # 最后添加一个(最底下的)
        self.encode_modules = nn.ModuleList(encode_list)
        self.decode_modules = nn.ModuleList(decode_list)

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        x_in = self.conv_in(x)

        x = x_in
        encode_outputs = []  # 用来收集每个encoder的输出
        for m in self.encode_modules:
            x = m(x)    # 输入的x依次放进去
            encode_outputs.append(x)

        x = encode_outputs.pop()    # 最后一个encoder的膨胀卷积的输出弹出来
        for m in self.decode_modules:
            x2 = encode_outputs.pop()
            x = m(x, x2)    # 两个tensor传入到decoder中得到一个输出

        return x + x_in  # 加上最开始模块的输出


class RSU4F(nn.Module):
    """在RSU4的基础上将所有上下采样替换为膨胀卷积
    ![20221013212305](http://image.xpshuai.cn/20221013212305.png)
    """
    def __init__(self, in_ch: int, mid_ch: int, out_ch: int):
        super().__init__()
        self.conv_in = ConvBNReLU(in_ch, out_ch)
        self.encode_modules = nn.ModuleList([ConvBNReLU(out_ch, mid_ch),
                                             ConvBNReLU(mid_ch, mid_ch, dilation=2),
                                             ConvBNReLU(mid_ch, mid_ch, dilation=4),
                                             ConvBNReLU(mid_ch, mid_ch, dilation=8)])

        self.decode_modules = nn.ModuleList([ConvBNReLU(mid_ch * 2, mid_ch, dilation=4),
                                             ConvBNReLU(mid_ch * 2, mid_ch, dilation=2),
                                             ConvBNReLU(mid_ch * 2, out_ch)])

    def forward(self, x: torch.Tensor) -> torch.Tensor:
        x_in = self.conv_in(x)

        x = x_in
        encode_outputs = []
        for m in self.encode_modules:
            x = m(x)
            encode_outputs.append(x)

        x = encode_outputs.pop()
        for m in self.decode_modules:
            x2 = encode_outputs.pop()
            x = m(torch.cat([x, x2], dim=1))

        return x + x_in


class U2Net(nn.Module):
    """最终的网络"""
    def __init__(self, cfg: dict, out_ch: int = 1):  # 这里显著性目标检测，所以out_ch=1
        super().__init__()
        assert "encode" in cfg
        assert "decode" in cfg
        self.encode_num = len(cfg["encode"])

        encode_list = []
        side_list = []
        for c in cfg["encode"]:
            # c: [height, in_ch, mid_ch, out_ch, RSU4F, side]
            assert len(c) == 6
            encode_list.append(RSU(*c[:4]) if c[4] is False else RSU4F(*c[1:4]))

            if c[5] is True:
                side_list.append(nn.Conv2d(c[3], out_ch, kernel_size=3, padding=1))
        self.encode_modules = nn.ModuleList(encode_list)

        decode_list = []
        for c in cfg["decode"]:
            # c: [height, in_ch, mid_ch, out_ch, RSU4F, side]
            assert len(c) == 6
            decode_list.append(RSU(*c[:4]) if c[4] is False else RSU4F(*c[1:4]))

            if c[5] is True:
                side_list.append(nn.Conv2d(c[3], out_ch, kernel_size=3, padding=1))
        self.decode_modules = nn.ModuleList(decode_list)
        self.side_modules = nn.ModuleList(side_list)
        self.out_conv = nn.Conv2d(self.encode_num * out_ch, out_ch, kernel_size=1)  # 1*1的卷积层融合不同尺度的信息

    def forward(self, x: torch.Tensor) -> Union[torch.Tensor, List[torch.Tensor]]:
        _, _, h, w = x.shape

        # collect encode outputs
        encode_outputs = []
        for i, m in enumerate(self.encode_modules):
            x = m(x)
            encode_outputs.append(x)
            if i != self.encode_num - 1:
                x = F.max_pool2d(x, kernel_size=2, stride=2, ceil_mode=True)

        # collect decode outputs
        x = encode_outputs.pop()
        decode_outputs = [x]
        for m in self.decode_modules:
            x2 = encode_outputs.pop()
            x = F.interpolate(x, size=x2.shape[2:], mode='bilinear', align_corners=False)
            x = m(torch.concat([x, x2], dim=1))
            decode_outputs.insert(0, x)

        # collect side outputs
        side_outputs = []
        for m in self.side_modules:
            x = decode_outputs.pop()
            x = F.interpolate(m(x), size=[h, w], mode='bilinear', align_corners=False)
            side_outputs.insert(0, x)

        x = self.out_conv(torch.concat(side_outputs, dim=1))

        if self.training:
            # do not use torch.sigmoid for amp safe
            return [x] + side_outputs
        else:
            return torch.sigmoid(x)


def u2net_full(out_ch: int = 1):
    cfg = {
        # height, in_ch, mid_ch, out_ch, RSU4F, side
        "encode": [[7, 3, 32, 64, False, False],      # En1
                   [6, 64, 32, 128, False, False],    # En2
                   [5, 128, 64, 256, False, False],   # En3
                   [4, 256, 128, 512, False, False],  # En4
                   [4, 512, 256, 512, True, False],   # En5
                   [4, 512, 256, 512, True, True]],   # En6
        # height, in_ch, mid_ch, out_ch, RSU4F, side
        "decode": [[4, 1024, 256, 512, True, True],   # De5
                   [4, 1024, 128, 256, False, True],  # De4
                   [5, 512, 64, 128, False, True],    # De3
                   [6, 256, 32, 64, False, True],     # De2
                   [7, 128, 16, 64, False, True]]     # De1
    }

    return U2Net(cfg, out_ch)


def u2net_lite(out_ch: int = 1):
    cfg = {
        # height, in_ch, mid_ch, out_ch, RSU4F, side
        "encode": [[7, 3, 16, 64, False, False],  # En1
                   [6, 64, 16, 64, False, False],  # En2
                   [5, 64, 16, 64, False, False],  # En3
                   [4, 64, 16, 64, False, False],  # En4
                   [4, 64, 16, 64, True, False],  # En5
                   [4, 64, 16, 64, True, True]],  # En6
        # height, in_ch, mid_ch, out_ch, RSU4F, side
        "decode": [[4, 128, 16, 64, True, True],  # De5
                   [4, 128, 16, 64, False, True],  # De4
                   [5, 128, 16, 64, False, True],  # De3
                   [6, 128, 16, 64, False, True],  # De2
                   [7, 128, 16, 64, False, True]]  # De1
    }

    return U2Net(cfg, out_ch)


def convert_onnx(m, save_path):
    m.eval()
    x = torch.rand(1, 3, 288, 288, requires_grad=True)

    # export the model
    torch.onnx.export(m,  # model being run
                      x,  # model input (or a tuple for multiple inputs)
                      save_path,  # where to save the model (can be a file or file-like object)
                      export_params=True,
                      opset_version=11)


if __name__ == '__main__':
    # n_m = RSU(height=7, in_ch=3, mid_ch=12, out_ch=3)
    # convert_onnx(n_m, "RSU7.onnx")
    #
    # n_m = RSU4F(in_ch=3, mid_ch=12, out_ch=3)
    # convert_onnx(n_m, "RSU4F.onnx")

    u2net = u2net_full()
    convert_onnx(u2net, "u2net_full.onnx")
```

train.py
```python
import os
import time
import datetime
from typing import Union, List

import torch
from torch.utils import data

from src.model import u2net_full
from utils.train_and_eval import train_one_epoch, evaluate, get_params_groups, create_lr_scheduler
from my_dataset import DUTSDataset
import transforms as T
import sys
sys.path.append('/home/xps/code/RemoteAICode/pytorch_segement/U2Net_Test')


class SODPresetTrain:
    """对验证集集数据预处理和增强"""
    def __init__(self, base_size: Union[int, List[int]], crop_size: int,
                 hflip_prob=0.5, mean=(0.485, 0.456, 0.406), std=(0.229, 0.224, 0.225)):
        self.transforms = T.Compose([
            T.ToTensor(),
            T.Resize(base_size, resize_mask=True),
            T.RandomCrop(crop_size),
            T.RandomHorizontalFlip(hflip_prob),
            T.Normalize(mean=mean, std=std)
        ])

    def __call__(self, img, target):
        return self.transforms(img, target)


class SODPresetEval:
    """对验证集集数据预处理和增强"""
    def __init__(self, base_size: Union[int, List[int]], mean=(0.485, 0.456, 0.406), std=(0.229, 0.224, 0.225)):
        self.transforms = T.Compose([
            T.ToTensor(),
            T.Resize(base_size, resize_mask=False),
            T.Normalize(mean=mean, std=std),
        ])

    def __call__(self, img, target):
        return self.transforms(img, target)


def main(args):
    device = torch.device(args.device if torch.cuda.is_available() else "cpu")
    batch_size = args.batch_size

    # 用来保存训练以及验证过程中信息
    results_file = "results{}.txt".format(datetime.datetime.now().strftime("%Y%m%d-%H%M%S"))

    train_dataset = DUTSDataset(args.data_path, train=True, transform=SODPresetTrain([320, 320], crop_size=256))  # 针对数据集进行处理，防止高宽变形
    val_dataset = DUTSDataset(args.data_path, train=False, transform=SODPresetEval([320, 320]))

    num_workers = min([os.cpu_count(), batch_size if batch_size > 1 else 0, 8])
    train_data_loader = data.DataLoader(train_dataset,
                                        batch_size=batch_size,
                                        num_workers=num_workers,
                                        shuffle=True,
                                        pin_memory=True)
                                        # collate_fn=train_dataset.collate_fn)

    val_data_loader = data.DataLoader(val_dataset,
                                      batch_size=1,  # must be 1 必须为1
                                      num_workers=num_workers,
                                      pin_memory=True,)
                                      # collate_fn=val_dataset.collate_fn)

    model = u2net_full()
    model.to(device)

    params_group = get_params_groups(model, weight_decay=args.weight_decay)
    optimizer = torch.optim.AdamW(params_group, lr=args.lr, weight_decay=args.weight_decay)
    lr_scheduler = create_lr_scheduler(optimizer, len(train_data_loader), args.epochs,
                                       warmup=True, warmup_epochs=2)    # 常规操作

    scaler = torch.cuda.amp.GradScaler() if args.amp else None

    if args.resume:
        checkpoint = torch.load(args.resume, map_location='cpu')
        model.load_state_dict(checkpoint['model'])
        optimizer.load_state_dict(checkpoint['optimizer'])
        lr_scheduler.load_state_dict(checkpoint['lr_scheduler'])
        args.start_epoch = checkpoint['epoch'] + 1
        if args.amp:
            scaler.load_state_dict(checkpoint["scaler"])

    current_mae, current_f1 = 1.0, 0.0  # 两个指标
    start_time = time.time()
    for epoch in range(args.start_epoch, args.epochs):
        # 每迭代一次就在训练集上训练一次，返回平均损失和学习率
        mean_loss, lr = train_one_epoch(model, optimizer, train_data_loader, device, epoch,
                                        lr_scheduler=lr_scheduler, print_freq=args.print_freq, scaler=scaler)

        save_file = {"model": model.state_dict(),
                     "optimizer": optimizer.state_dict(),
                     "lr_scheduler": lr_scheduler.state_dict(),
                     "epoch": epoch,
                     "args": args}
        if args.amp:
            save_file["scaler"] = scaler.state_dict()

        # 如果是整数倍&最后一个，就去进行评估
        if epoch % args.eval_interval == 0 or epoch == args.epochs - 1:
            # 每间隔eval_interval个epoch验证一次，减少验证频率节省训练时间
            mae_metric, f1_metric = evaluate(model, val_data_loader, device=device)
            mae_info, f1_info = mae_metric.compute(), f1_metric.compute()
            print(f"[epoch: {epoch}] val_MAE: {mae_info:.3f} val_maxF1: {f1_info:.3f}")
            # write into txt
            with open(results_file, "a") as f:
                # 记录每个epoch对应的train_loss、lr以及验证集各指标到txt文件中
                write_info = f"[epoch: {epoch}] train_loss: {mean_loss:.4f} lr: {lr:.6f} " \
                             f"MAE: {mae_info:.3f} maxF1: {f1_info:.3f} \n"
                f.write(write_info)

            # save_best，保存更好的权重
            if current_mae >= mae_info and current_f1 <= f1_info:
                torch.save(save_file, "save_weights/model_best.pth")

        # only save latest 10 epoch weights，保留最近10个epoch的权重
        if os.path.exists(f"save_weights/model_{epoch-10}.pth"):
            os.remove(f"save_weights/model_{epoch-10}.pth")

        torch.save(save_file, f"save_weights/model_{epoch}.pth")

    total_time = time.time() - start_time
    total_time_str = str(datetime.timedelta(seconds=int(total_time)))
    print("training time {}".format(total_time_str))


def parse_args():
    import argparse
    parser = argparse.ArgumentParser(description="pytorch u2net training")

    parser.add_argument("--data-path", default="./", help="DUTS root")
    parser.add_argument("--device", default="cuda", help="training device")
    parser.add_argument("-b", "--batch-size", default=1, type=int)
    parser.add_argument('--wd', '--weight-decay', default=1e-4, type=float,
                        metavar='W', help='weight decay (default: 1e-4)',
                        dest='weight_decay')
    parser.add_argument("--epochs", default=100, type=int, metavar="N", help="number of total epochs to train")
    parser.add_argument("--eval-interval", default=10, type=int, help="validation interval default 10 Epochs")  # 每个epoch都去验证比较费时间，所以没10个区验证一次

    parser.add_argument('--lr', default=0.001, type=float, help='initial learning rate')
    parser.add_argument('--print-freq', default=50, type=int, help='print frequency')
    parser.add_argument('--resume', default='', help='resume from checkpoint')
    parser.add_argument('--start-epoch', default=0, type=int, metavar='N', help='start epoch')
    # Mixed precision training parameters
    parser.add_argument("--amp", action='store_true', help="Use torch.cuda.amp for mixed precision training")

    args = parser.parse_args()

    return args


if __name__ == '__main__':
    args = parse_args()

    if not os.path.exists("./save_weights"):
        os.mkdir("./save_weights")

    main(args)
    torch.cuda.empty_cache()

```

train_and_eval.py
```python
import math
import torch
from torch.nn import functional as F
# from distributed_utils import MeanAbsoluteError, F1Score, MetricLogger, SmoothedValue
from torchvision.utils import  save_image

def criterion(inputs, target):
    losses = [F.binary_cross_entropy_with_logits(inputs[i], target) for i in range(len(inputs))]
    total_loss = sum(losses)    # 总损失

    return total_loss


def evaluate(model, data_loader, device):
    model.eval()
    mae_metric = MeanAbsoluteError()
    f1_metric = F1Score()
    metric_logger = MetricLogger(delimiter="  ")
    header = 'Test:'
    with torch.no_grad():
        for images, targets in metric_logger.log_every(data_loader, 100, header):
            images, targets = images.to(device), targets.to(device)
            output = model(images)

            # post norm
            # ma = torch.max(output)
            # mi = torch.min(output)
            # output = (output - mi) / (ma - mi)

            mae_metric.update(output, targets)
            f1_metric.update(output, targets)

        mae_metric.gather_from_all_processes()
        f1_metric.reduce_from_all_processes()

    return mae_metric, f1_metric


def train_one_epoch(model, optimizer, data_loader, device, epoch, lr_scheduler, print_freq=10, scaler=None):
    model.train()
    metric_logger = MetricLogger(delimiter="  ")
    metric_logger.add_meter('lr', SmoothedValue(window_size=1, fmt='{value:.6f}'))
    header = 'Epoch: [{}]'.format(epoch)
    i = 0
    for image, target in metric_logger.log_every(data_loader, print_freq, header):
        image, target = image.to(device), target.to(device)
        with torch.cuda.amp.autocast(enabled=scaler is not None):
            output = model(image)
            loss = criterion(output, target)
            # img = torch.stack([image[0], output[0]], dim=0)  # dim=0 [b,c,h,w]
            save_image(output[0], f'tmp_out_img/{i}.png')
            i += 1

        optimizer.zero_grad()
        if scaler is not None:
            scaler.scale(loss).backward()
            scaler.step(optimizer)
            scaler.update()
        else:
            loss.backward()
            optimizer.step()

        lr_scheduler.step()

        lr = optimizer.param_groups[0]["lr"]
        metric_logger.update(loss=loss.item(), lr=lr)

    return metric_logger.meters["loss"].global_avg, lr


def create_lr_scheduler(optimizer,
                        num_step: int,
                        epochs: int,
                        warmup=True,
                        warmup_epochs=1,
                        warmup_factor=1e-3,
                        end_factor=1e-6):
    assert num_step > 0 and epochs > 0
    if warmup is False:
        warmup_epochs = 0

    def f(x):
        """
        根据step数返回一个学习率倍率因子，
        注意在训练开始之前，pytorch会提前调用一次lr_scheduler.step()方法
        """
        if warmup is True and x <= (warmup_epochs * num_step):
            alpha = float(x) / (warmup_epochs * num_step)
            # warmup过程中lr倍率因子从warmup_factor -> 1
            return warmup_factor * (1 - alpha) + alpha
        else:
            current_step = (x - warmup_epochs * num_step)
            cosine_steps = (epochs - warmup_epochs) * num_step
            # warmup后lr倍率因子从1 -> end_factor
            return ((1 + math.cos(current_step * math.pi / cosine_steps)) / 2) * (1 - end_factor) + end_factor

    return torch.optim.lr_scheduler.LambdaLR(optimizer, lr_lambda=f)


def get_params_groups(model: torch.nn.Module, weight_decay: float = 1e-4):
    params_group = [{"params": [], "weight_decay": 0.},  # no decay
                    {"params": [], "weight_decay": weight_decay}]  # with decay

    for name, param in model.named_parameters():
        if not param.requires_grad:
            continue  # frozen weights

        if len(param.shape) == 1 or name.endswith(".bias"):
            # bn:(weight,bias)  conv2d:(bias)  linear:(bias)
            params_group[0]["params"].append(param)  # no decay
        else:
            params_group[1]["params"].append(param)  # with decay

    return params_group
```

distributed_utils.py
```python

from collections import defaultdict, deque
import datetime
import time
import torch
import torch.distributed as dist
import torch.nn.functional as F

import errno
import os


class SmoothedValue(object):
    """Track a series of values and provide access to smoothed values over a
    window or the global series average.
    """

    def __init__(self, window_size=20, fmt=None):
        if fmt is None:
            fmt = "{value:.4f} ({global_avg:.4f})"
        self.deque = deque(maxlen=window_size)
        self.total = 0.0
        self.count = 0
        self.fmt = fmt

    def update(self, value, n=1):
        self.deque.append(value)
        self.count += n
        self.total += value * n

    def synchronize_between_processes(self):
        """
        Warning: does not synchronize the deque!
        """
        if not is_dist_avail_and_initialized():
            return
        t = torch.tensor([self.count, self.total], dtype=torch.float64, device='cuda')
        dist.barrier()
        dist.all_reduce(t)
        t = t.tolist()
        self.count = int(t[0])
        self.total = t[1]

    @property
    def median(self):
        d = torch.tensor(list(self.deque))
        return d.median().item()

    @property
    def avg(self):
        d = torch.tensor(list(self.deque), dtype=torch.float32)
        return d.mean().item()

    @property
    def global_avg(self):
        return self.total / self.count

    @property
    def max(self):
        return max(self.deque)

    @property
    def value(self):
        return self.deque[-1]

    def __str__(self):
        return self.fmt.format(
            median=self.median,
            avg=self.avg,
            global_avg=self.global_avg,
            max=self.max,
            value=self.value)


def all_gather(data):
    """
    收集各个进程中的数据
    Run all_gather on arbitrary picklable data (not necessarily tensors)
    Args:
        data: any picklable object
    Returns:
        list[data]: list of data gathered from each rank
    """
    world_size = get_world_size()  # 进程数
    if world_size == 1:
        return [data]

    data_list = [None] * world_size
    dist.all_gather_object(data_list, data)

    return data_list


class MeanAbsoluteError(object):
    def __init__(self):
        self.mae_list = []

    def update(self, pred: torch.Tensor, gt: torch.Tensor):
        batch_size, c, h, w = gt.shape
        assert batch_size == 1, f"validation mode batch_size must be 1, but got batch_size: {batch_size}."
        resize_pred = F.interpolate(pred, (h, w), mode="bilinear", align_corners=False)
        error_pixels = torch.sum(torch.abs(resize_pred - gt), dim=(1, 2, 3)) / (h * w)
        self.mae_list.extend(error_pixels.tolist())

    def compute(self):
        mae = sum(self.mae_list) / len(self.mae_list)
        return mae

    def gather_from_all_processes(self):
        if not torch.distributed.is_available():
            return
        if not torch.distributed.is_initialized():
            return
        torch.distributed.barrier()
        gather_mae_list = []
        for i in all_gather(self.mae_list):
            gather_mae_list.extend(i)
        self.mae_list = gather_mae_list

    def __str__(self):
        mae = self.compute()
        return f'MAE: {mae:.3f}'


class F1Score(object):
    """
    refer: https://github.com/xuebinqin/DIS/blob/main/IS-Net/basics.py
    """

    def __init__(self, threshold: float = 0.5):
        self.precision_cum = None
        self.recall_cum = None
        self.num_cum = None
        self.threshold = threshold

    def update(self, pred: torch.Tensor, gt: torch.Tensor):
        batch_size, c, h, w = gt.shape
        assert batch_size == 1, f"validation mode batch_size must be 1, but got batch_size: {batch_size}."
        resize_pred = F.interpolate(pred, (h, w), mode="bilinear", align_corners=False)
        gt_num = torch.sum(torch.gt(gt, self.threshold).float())

        pp = resize_pred[torch.gt(gt, self.threshold)]  # 对应预测map中GT为前景的区域
        nn = resize_pred[torch.le(gt, self.threshold)]  # 对应预测map中GT为背景的区域

        pp_hist = torch.histc(pp, bins=255, min=0.0, max=1.0)
        nn_hist = torch.histc(nn, bins=255, min=0.0, max=1.0)

        # Sort according to the prediction probability from large to small
        pp_hist_flip = torch.flipud(pp_hist)
        nn_hist_flip = torch.flipud(nn_hist)

        pp_hist_flip_cum = torch.cumsum(pp_hist_flip, dim=0)
        nn_hist_flip_cum = torch.cumsum(nn_hist_flip, dim=0)

        precision = pp_hist_flip_cum / (pp_hist_flip_cum + nn_hist_flip_cum + 1e-4)
        recall = pp_hist_flip_cum / (gt_num + 1e-4)

        if self.precision_cum is None:
            self.precision_cum = torch.full_like(precision, fill_value=0.)

        if self.recall_cum is None:
            self.recall_cum = torch.full_like(recall, fill_value=0.)

        if self.num_cum is None:
            self.num_cum = torch.zeros([1], dtype=gt.dtype, device=gt.device)

        self.precision_cum += precision
        self.recall_cum += recall
        self.num_cum += batch_size

    def compute(self):
        pre_mean = self.precision_cum / self.num_cum
        rec_mean = self.recall_cum / self.num_cum
        f1_mean = (1 + 0.3) * pre_mean * rec_mean / (0.3 * pre_mean + rec_mean + 1e-8)
        max_f1 = torch.amax(f1_mean).item()
        return max_f1

    def reduce_from_all_processes(self):
        if not torch.distributed.is_available():
            return
        if not torch.distributed.is_initialized():
            return
        torch.distributed.barrier()
        torch.distributed.all_reduce(self.precision_cum)
        torch.distributed.all_reduce(self.recall_cum)
        torch.distributed.all_reduce(self.num_cum)

    def __str__(self):
        max_f1 = self.compute()
        return f'maxF1: {max_f1:.3f}'


class MetricLogger(object):
    def __init__(self, delimiter="\t"):
        self.meters = defaultdict(SmoothedValue)
        self.delimiter = delimiter

    def update(self, **kwargs):
        for k, v in kwargs.items():
            if isinstance(v, torch.Tensor):
                v = v.item()
            assert isinstance(v, (float, int))
            self.meters[k].update(v)

    def __getattr__(self, attr):
        if attr in self.meters:
            return self.meters[attr]
        if attr in self.__dict__:
            return self.__dict__[attr]
        raise AttributeError("'{}' object has no attribute '{}'".format(
            type(self).__name__, attr))

    def __str__(self):
        loss_str = []
        for name, meter in self.meters.items():
            loss_str.append(
                "{}: {}".format(name, str(meter))
            )
        return self.delimiter.join(loss_str)

    def synchronize_between_processes(self):
        for meter in self.meters.values():
            meter.synchronize_between_processes()

    def add_meter(self, name, meter):
        self.meters[name] = meter

    def log_every(self, iterable, print_freq, header=None):
        i = 0
        if not header:
            header = ''
        start_time = time.time()
        end = time.time()
        iter_time = SmoothedValue(fmt='{avg:.4f}')
        data_time = SmoothedValue(fmt='{avg:.4f}')
        space_fmt = ':' + str(len(str(len(iterable)))) + 'd'
        if torch.cuda.is_available():
            log_msg = self.delimiter.join([
                header,
                '[{0' + space_fmt + '}/{1}]',
                'eta: {eta}',
                '{meters}',
                'time: {time}',
                'data: {data}',
                'max mem: {memory:.0f}'
            ])
        else:
            log_msg = self.delimiter.join([
                header,
                '[{0' + space_fmt + '}/{1}]',
                'eta: {eta}',
                '{meters}',
                'time: {time}',
                'data: {data}'
            ])
        MB = 1024.0 * 1024.0
        for obj in iterable:
            data_time.update(time.time() - end)
            yield obj
            iter_time.update(time.time() - end)
            if i % print_freq == 0:
                eta_seconds = iter_time.global_avg * (len(iterable) - i)
                eta_string = str(datetime.timedelta(seconds=int(eta_seconds)))
                if torch.cuda.is_available():
                    print(log_msg.format(
                        i, len(iterable), eta=eta_string,
                        meters=str(self),
                        time=str(iter_time), data=str(data_time),
                        memory=torch.cuda.max_memory_allocated() / MB))
                else:
                    print(log_msg.format(
                        i, len(iterable), eta=eta_string,
                        meters=str(self),
                        time=str(iter_time), data=str(data_time)))
            i += 1
            end = time.time()
        total_time = time.time() - start_time
        total_time_str = str(datetime.timedelta(seconds=int(total_time)))
        print('{} Total time: {}'.format(header, total_time_str))


def mkdir(path):
    try:
        os.makedirs(path)
    except OSError as e:
        if e.errno != errno.EEXIST:
            raise


def setup_for_distributed(is_master):
    """
    This function disables printing when not in master process
    """
    import builtins as __builtin__
    builtin_print = __builtin__.print

    def print(*args, **kwargs):
        force = kwargs.pop('force', False)
        if is_master or force:
            builtin_print(*args, **kwargs)

    __builtin__.print = print


def is_dist_avail_and_initialized():
    if not dist.is_available():
        return False
    if not dist.is_initialized():
        return False
    return True


def get_world_size():
    if not is_dist_avail_and_initialized():
        return 1
    return dist.get_world_size()


def get_rank():
    if not is_dist_avail_and_initialized():
        return 0
    return dist.get_rank()


def is_main_process():
    return get_rank() == 0


def save_on_master(*args, **kwargs):
    if is_main_process():
        torch.save(*args, **kwargs)


def init_distributed_mode(args):
    if 'RANK' in os.environ and 'WORLD_SIZE' in os.environ:
        args.rank = int(os.environ["RANK"])
        args.world_size = int(os.environ['WORLD_SIZE'])
        args.gpu = int(os.environ['LOCAL_RANK'])
    elif 'SLURM_PROCID' in os.environ:
        args.rank = int(os.environ['SLURM_PROCID'])
        args.gpu = args.rank % torch.cuda.device_count()
    elif hasattr(args, "rank"):
        pass
    else:
        print('Not using distributed mode')
        args.distributed = False
        return

    args.distributed = True

    torch.cuda.set_device(args.gpu)
    args.dist_backend = 'nccl'
    print('| distributed init (rank {}): {}'.format(
        args.rank, args.dist_url), flush=True)
    torch.distributed.init_process_group(backend=args.dist_backend, init_method=args.dist_url,
                                         world_size=args.world_size, rank=args.rank)
    setup_for_distributed(args.rank == 0)


```

跑出来的效果：
![](http://image.xpshuai.cn/20221014171951.png)

![](http://image.xpshuai.cn/20221014172033.png)


> 代码没好好整合过，后面等换个好电脑再弄吧，




## 参考
https://blog.csdn.net/ling620/article/details/110127019

---

> 作者: [剑胆琴心](http://shuai06.github.io)  
> URL: https://shuai06.github.io/u2-net/  

