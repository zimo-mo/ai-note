# lightning-transformers

## Pytorch-Lightning

### LightningDataModule

LightningDataModule定义了5个api:
  - prepare_data (how to download(), tokenize, etc…)
  - setup (how to split, etc…)
  - train_dataloader
  - val_dataloader(s)
  - test_dataloader(s)

**prepare_data**

该函数负责预处理数据，包括下载，tokenize等。

*注意*：prepare_data is called from a single process (e.g. GPU 0). Do not use it to assign state (self.x = y).


**setup**

如果希望对数据的操作能在每个GPU上生效，可以通过`setup`实现一下逻辑：

  - 统计分类数
  - 构建词典
  - 数据分割（train/val/test）
  - 数据转换
  - ...

例如：

```python
import pytorch_lightning as pl


class MNISTDataModule(pl.LightningDataModule):
    def setup(self, stage: Optional[str] = None):

        # Assign Train/val split(s) for use in Dataloaders
        if stage in (None, "fit"):
            mnist_full = MNIST(self.data_dir, train=True, download=True, transform=self.transform)
            self.mnist_train, self.mnist_val = random_split(mnist_full, [55000, 5000])
            self.dims = self.mnist_train[0][0].shape

        # Assign Test split(s) for use in Dataloaders
        if stage in (None, "test"):
            self.mnist_test = MNIST(self.data_dir, train=False, download=True, transform=self.transform)
            self.dims = getattr(self, "dims", self.mnist_test[0][0].shape)
```

`stage`为`setup`可选参数。为trainer实现分割逻辑`trainer.{fit,validate,test}`。如果`stage=None`，包含fit/validate/test全部逻辑。

**train_dataloader/val_dataloader/test_dataloader**

`train_dataloader/val_dataloader/test_dataloader`封装并返回`setup`中分割的数据。例如：

```python
import pytorch_lightning as pl


class MNISTDataModule(pl.LightningDataModule):
    def train_dataloader(self):
        return DataLoader(self.mnist_train, batch_size=64)
        
    def val_dataloader(self):
        return DataLoader(self.mnist_val, batch_size=64)
        
    def test_dataloader(self):
        return DataLoader(self.mnist_test, batch_size=64)
```

**predict_dataloader**

返回用于推理的数据集，用于`trainer.predict`方法使用。 如下所示：

```python
import pytorch_lightning as pl


class MNISTDataModule(pl.LightningDataModule):
    def predict_dataloader(self):
        return DataLoader(self.mnist_test, batch_size=64)
```

### LightningModule

`LightningModule`其实也是`torch.nn.Module`，包括5部分：

  - init
  - training_step
  - validation_step
  - test_step
  - configure_optimizers
  
  更详细的用法参考[LightningModule](https://pytorch-lightning.readthedocs.io/en/stable/common/lightning_module.html?highlight=LightningModule)

## Datasets of transformers

datasets后端采用Apache Arrow格式，极大的提高了数据的处理速度。以下主要从datasets加载数据、处理等两方面做介绍。其他功能参考[datasets文档](https://huggingface.co/docs/datasets)

### 数据加载

数据加载支持以下加载方式：
- The Hub
- Local files
- In-memory data
- Offline
- A specific slice of a split

**加载本地及远程文件**

支持加载`csv/json/txt/Parquet`等文件

```python
from datasets import load_dataset

file_type = "csv" # json/text/parquet
data_files = {"train": train_file_path}
# data_files = train_file_path
dataset = load_dataset(file_type, data_files=data_files)
```

**从内存加载**

从dict对象加载

```python
from datasets import Dataset

my_dict = {"a": [1, 2, 3]}

dataset = Dataset.from_dict(my_dict)
```

从pandas Dataframe加载

```python
import pandas as pd
from datasets import Dataset


df = pd.DataFrame({"a": [1, 2, 3]})

dataset = Dataset.from_pandas(df)
```
