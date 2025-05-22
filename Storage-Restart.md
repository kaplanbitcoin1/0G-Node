<p align="center">
  <img src="https://github.com/user-attachments/assets/5c76256c-4aad-457d-a43f-8de77238dd22" width="450"/>
</p>


* Storage node bazen bazı blocklarda takılabiliyor. 
* Bunun önüne geçmek için 30 saniyede bir block kontrolü yapan, aynı block 2 defa üst üste gelirse service dosyasına restart atan bir script hazırladım.


* Senaryo dosyasını oluşturalım

```
sudo nano /usr/local/bin/inspection.sh
```

* Script'i içeriye tek komut halinde yapıştıralım

```
#!/bin/bash

LOG_FILE="$HOME/0g-storage-node/run/log/zgs.log.$(TZ=UTC date +%Y-%m-%d)"
SERVICE_NAME="zgsd"
CHECK_INTERVAL=30  # 30 saniye
STALE_THRESHOLD=2  # Aynı tx_seq kaç kez görülürse restart edilsin

last_seq=""
unchanged_count=0

while true; do
    current_seq=$(grep "tx_seq" "$LOG_FILE" | tail -1 | grep -oP 'tx_seq:\K[0-9]+')

    if [[ -z "$current_seq" ]]; then
        echo "$(date) - tx_seq bulunamadı, log hatalı olabilir." >> "$HOME/inspection.log"
    else
        if [[ "$current_seq" == "$last_seq" ]]; then
            ((unchanged_count++))
            echo "$(date) - tx_seq $current_seq değişmedi ($unchanged_count/$STALE_THRESHOLD)" >> "$HOME/inspection.log"
        else
            unchanged_count=0
            last_seq=$current_seq
            echo "$(date) - tx_seq güncellendi: $current_seq" >> "$HOME/inspection.log"
        fi

        if (( unchanged_count >= STALE_THRESHOLD )); then
            echo "$(date) - tx_seq uzun süredir değişmedi. $SERVICE_NAME servisi yeniden başlatılıyor..." >> "$HOME/inspection.log"
            systemctl restart $SERVICE_NAME
            unchanged_count=0
            sleep 60  # restart sonrası bekleme
        fi
    fi

    sleep $CHECK_INTERVAL
done

```

* Çalışma yetkisi veriyoruz


```
sudo chmod +x /usr/local/bin/inspection.sh
```


* Çalışıyor mu diye kontrol etmek için service dosyası


```
sudo tee /etc/systemd/system/inspection.service > /dev/null <<EOF
[Unit]
Description=ZGS Log Kontrolü ve Otomatik Restart
After=network-online.target

[Service]
ExecStart=/usr/local/bin/inspection.sh
Restart=always
RestartSec=10
User=root

[Install]
WantedBy=multi-user.target
EOF
```

* Sona yaklaştık

```
sudo systemctl daemon-reload
sudo systemctl enable inspection
sudo systemctl start inspection
```

* Durum kontrolü

```
sudo systemctl status inspection
```

* Loglara bakmak için


```
tail -f ~/inspection.log
```
