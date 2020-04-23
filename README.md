# PyTorch Distributed K-FAC Preconditioner

Distributed K-FAC Preconditioner in PyTorch using [Horovod](https://github.com/horovod/horovod) for communication.

The KFAC code was originally forked from Chaoqi Wang's [KFAC-PyTorch](https://github.com/alecwangcq/KFAC-Pytorch).
The ResNet models for Cifar10 are from Yerlan Idelbayev's [pytorch_resnet_cifar10](https://github.com/akamaster/pytorch_resnet_cifar10).
The CIFAR-10 and ImageNet-1k training scripts are modeled afer Horovod's example PyTorch training scripts.

## Requirements

PyTorch and Horovod are required to use K-FAC.

This code is validated to run with PyTorch v1.1, Horovod 0.19.0, CUDA 10.0/1, CUDNN 7.6.4, and NCCL 2.4.7.

## Usage

The K-FAC Preconditioner can be easily added to exisiting training scripts that use `horovod.DistributedOptimizer()`.

```Python
... 
optimizer = optim.SGD(model.parameters(), ...)
optimizer = hvd.DistributedOptimizer(optimizer, ...)
preconditioner = KFAC(model, ...)
... 
for i, (data, target) in enumerate(train_loader):
    optimizer.zero_grad()
    output = model(data)
    loss = criterion(output, target)
    loss.backward()
    optimizer.synchronize()
    preconditioner.step()
    with optimizer.skip_synchronize():
        optimizer.step()
...
```

Note that the K-FAC Preconditioner expects gradients to be averaged across workers before calling `preconditioner.step()` so we call `optimizer.synchronize()` before hand (Normally `optimizer.synchronize()` is not called until `optimizer.step()`). 

## Example Scripts

Example scripts for K-FAC + SGD training on CIFAR-10 and ImageNet-1k are provided.

### CIFAR-10
```
$ mpiexec -hostfile /path/to/hostfile -N 4 python examples/pytorch_cifar10_resnet.py \
      --lr 0.1 --epochs 100 --kfac-update-freq 10 --model resnet32 --lr-decay 35 75 90
```

### ImageNet-1k
```
$ mpiexec -hostfile /path/to/hostfile -N 4 python examples/pytorch_imagenet_resnet.py \
      --lr 0.0125 --epochs 55 --kfac-update-freq 10 --model resnet32 --lr-decay 25 35 40 45 50
```

See `python examples/pytorch_{dataset}_resnet.py --help` for a full list of hyper-parameters.
Note: if `--kfac-update-freq 0`, the K-FAC Preconditioning is skipped entirely, i.e. training is just with normal SGD.

## Citation

Coming soon.
