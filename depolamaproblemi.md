### 0G Storage depolama probleminden bu yöntemle kurtulabilirsiniz. Ekstra olarak Log dosyalarını (öncelikle stop edip) - dosya yolu: /root/0g-storage-node/run/log/ - en yakın tarih kalacak şekilde temizleyebilirsiniz.

```console
sudo systemctl stop zgsd
```

```console
nano $HOME/0g-storage-node/run/log_config
```

warn,hyper=warn,h2=warn 


```console
sudo systemctl start zgsd
```

```console
sudo systemctl restart zgsd
```

```console
 tail -f ~/0g-storage-node/run/log/zgs.log.$(TZ=UTC date +%Y-%m-%d)
```

Loglar şıkır şıkır
