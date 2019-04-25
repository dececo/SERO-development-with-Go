# 常用命令

## 新建帐户

```javascript
> personal.newAccount("password")
"4CZWS8sPWqi2CLFVKANJsif9xSEYVBFG42ma84P4VNghWBrEQAVL9ATELwYJF4FPkSnsMyj9fVSJbmjfj4JbYJ2Y"
```

```bash
$ ls keystore/
UTC--2019-04-25T02-59-30.714364000Z--4CZWS8sPWqi2CLFVKANJsif9xSEYVBFG42ma84P4VNghWBrEQAVL9ATELwYJF4FPkSnsMyj9fVSJbmjfj4JbYJ2Y
```

## 查询余额

```text
web3.fromTa(sero.getBalance(sero.accounts[0]).tkn.SERO)
```

## 挖矿和停止

```text
miner.start()
miner.stop()
```

