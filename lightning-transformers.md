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

该函数负责预处理数据，包括下载，转换，tokenize。
*注意*：prepare_data is called from a single process (e.g. GPU 0). Do not use it to assign state (self.x = y).


## Datasets of transformers

datasets后端采用Apache Arrow格式，极大的提高了数据的处理速度。

### 安装
```bash
pip install datasets
```
