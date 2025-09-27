# Gensyn-Node-Kurulum
Gensyn Node Kurulum
Gereksinimler

Ubuntu 20.04 / 22.04

En az 2 CPU, 4GB RAM (tercihen daha yüksek)

Yeterli disk alanı (chain data boyutuna göre)

İnternet bağlantısı

Hızlı kurulum (örnek)
# sunucuya bağlandıktan sonra
chmod +x install.sh
sudo ./install.sh

Servis kurulduktan sonra:

sudo systemctl enable --now gensyn.service
sudo journalctl -u gensyn.service -f
install.sh (açıklamalı, örnek)
#!/usr/bin/env bash
set -euo pipefail


# Basit ön kontroller
if [[ $(id -u) -ne 0 ]]; then
  echo "Lütfen root olarak çalıştırın: sudo ./install.sh"
  exit 1
fi


# 1) Paket güncelleme ve temel bağımlılıklar
apt update && apt upgrade -y
apt install -y build-essential curl wget git jq unzip ca-certificates


# 2) Kullanıcı oluştur (opsiyonel)
NODE_USER=gensyn
if ! id -u "$NODE_USER" >/dev/null 2>&1; then
  useradd -m -s /bin/bash $NODE_USER
fi


# 3) Gerekli dizinler
INSTALL_DIR=/opt/gensyn
mkdir -p $INSTALL_DIR
chown $NODE_USER:$NODE_USER $INSTALL_DIR


# 4) Binary veya repo çekme (örnek - güncelleyin)
# Örnek: github repo'dan release indir
GITHUB_REPO="github.com/example/gensyn-node"
RELEASE_URL="https://github.com/example/gensyn-node/releases/latest/download/gensyn-node-linux-amd64.tar.gz"


wget -O /tmp/gensyn-node.tar.gz "$RELEASE_URL"
tar -xzvf /tmp/gensyn-node.tar.gz -C $INSTALL_DIR
chown -R $NODE_USER:$NODE_USER $INSTALL_DIR


# 5) .env örneğini kopyala
cp .env.example $INSTALL_DIR/.env
chown $NODE_USER:$NODE_USER $INSTALL_DIR/.env


# 6) systemd servis dosyasını kopyala ve daemon reload
cp gensyn.service /etc/systemd/system/gensyn.service
systemctl daemon-reload
systemctl enable gensyn.service


echo "Kurulum tamamlandı. 'sudo systemctl start gensyn.service' ile başlatın veya restart edin."

Not: RELEASE_URL ve GITHUB_REPO değerlerini projenize göre güncelleyin. Eğer binary yoksa git clone ve make build adımlarını ekleyin.

gensyn.service (örnek systemd)
[Unit]
Description=Gensyn Node
After=network.target
[Service]
User=gensyn
Group=gensyn
Type=simple
Restart=on-failure
RestartSec=5
EnvironmentFile=/opt/gensyn/.env
ExecStart=/opt/gensyn/gensyn-node --config /opt/gensyn/config.toml
WorkingDirectory=/opt/gensyn
LimitNOFILE=65536
# Loglar journal'a gidecek
StandardOutput=journal
StandardError=journal


[Install]
WantedBy=multi-user.target
.env.example
# Gensyn node environment
NODE_NAME=gensyn-node-01
RPC_LISTEN=0.0.0.0:26657
P2P_LISTEN=0.0.0.0:26656
# JSON-RPC endpoint veya diğer özel değişkenler
JSON_RPC_URL=https://your-rpc.example
start.sh (örnek kullanımlar)
#!/usr/bin/env bash
set -euo pipefail


case "${1:-start}" in
  start)
    sudo systemctl start gensyn.service
    sudo systemctl status -l gensyn.service --no-pager
    ;;
  stop)
    sudo systemctl stop gensyn.service
    ;;
  restart)
    sudo systemctl restart gensyn.service
    ;;
  logs)
    sudo journalctl -u gensyn.service -f
    ;;
  update)
    echo "update işlemini proje yapısına göre doldurun (pull, build, restart)"
    ;;
  *)
    echo "kullanım: $0 {start|stop|restart|logs|update}"
    ;;
esac
docs/troubleshooting.md (kısa)

Port conflict: failed to bind hatası alırsanız ss -tuln | grep PORT ile hangi process'in portu kullandığını bulun.

Disk dolu: df -h ve du -sh /opt/gensyn komutları ile yer kontrolü.

Too many open files: ulimit -n arttırın (/etc/security/limits.conf).

Tendermint / ABCI hataları: veri tabanını yedekleyip unsafe_reset_all veya data dizinini temizleyip yeniden sync düşünün.
