---
title: 关于bitcoin数据制作k线
tags: 区块链 bitcoin
categories: python
---

## 简介

> 伴随着312的股市熔断和bitcoin腰斩仅仅过了8个月，如今apple创下20000亿美金的市值，再次刷新人类历史有史第一次突破20000亿美金（第一次突破10000亿美元也是apple，在2018年8月左右）。bitcoin也随也从最低的3800美金一路上扬并且突破了历史最高并达到新的历史最高点19888美金。
>
> 开始有些老哥开始询问关于量化交易的一些问题，首先我不会做量化交易，我认为更好的策略更重要。其次本人对于量化交易属于完全的一窍不通，所以暂时不会碰这块，更不会用别人写好的量化交易程序而用自己的钱做实验。
>
> 考虑到自己是个程序员，那么自己打算用python来制作个bitcoin的k线图。提前熟悉下交易所的api，为以后进行量化交易做准备。

## bitcoin数据来源

```
https://www.coindesk.com/price/bitcoin
```

## 程序编写

```python
import matplotlib.pyplot as plt
import pandas as pd

date_array=[]
price_array=[]

path_image='/Users/bulingfeng/Desktop/image/test.png'
path_bitcoin='/Users/bulingfeng/Desktop/image/bitcoin.csv'

def bitcoin_image():
    # plot函数作图
    plt.plot(date_array, price_array, color="g")

    # 设置横坐标和纵坐标的描述
    plt.xlabel("date")
    plt.ylabel("price")
    # 加网格
    plt.grid(color="k", linestyle=":")
    # show函数展示出这个图，如果没有这行代码，则程序完成绘图，但看不到
    # plt.show()
    # 这个按照具体的路径进行把图片给保存
    plt.savefig(path_image)

def read_csv():
    data = pd.read_csv(path_bitcoin)  # 读取csv文件
    # print(data)
    print(data.columns.size)
    print(data.loc[1:2])  # 打印第1到2行
    # data.loc[2:4, ['PassengerId', 'Sex']]
    for index, row in data.iterrows():
        # print(row['Date'], row['Closing Price (USD)'], type(row['Date']), type(row['Closing Price (USD)']))
        date=row['Date']
        price=row['Closing Price (USD)']
        date_array.append(date.replace("-",""))
        price_array.append((int)(price))

if __name__ == '__main__':
    read_csv()
    bitcoin_image()

```

## 参考文档

```
https://blog.csdn.net/guoziqing506/article/details/78975150
```

