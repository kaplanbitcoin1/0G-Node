0G Storage depolama probleminden bu yöntemle kurtulabilirsiniz. Ekstra olarak Log dosyalarını (öncelikle stop edip) - dosya yolu: /root/0g-storage-node/run/log/ - en yakın tarih kalacak şekilde temizleyebilirsiniz.

```shell
sudo systemctl stop zgsd
```

```shell
nano $HOME/0g-storage-node/run/log_config
```

warn,hyper=warn,h2=warn 


```shell
sudo systemctl start zgsd
```

```shell
sudo systemctl restart zgsd
```

```shell
 tail -f ~/0g-storage-node/run/log/zgs.log.$(TZ=UTC date +%Y-%m-%d)
```

Loglar şıkır şıkır
