# Quay Registry Kurulum Dökümanı (RHEL 9.7)

Aşağıdaki komutlar RHEL 9.7 üzerinde Quay Registry kurmak için örnek bir kurulum akışını gösterir.

## Adımlar

### 1. Tüm çalışan konteynerleri durdur ve sil
```bash
podman rm -f $(podman ps -aq)
```

### 2. Eski dosyaları tamamen sil
```bash
sudo rm -rf /var/lib/quay
```

### 3. Dizinleri sıfırdan oluştur
```bash
sudo mkdir -p /var/lib/quay/config
sudo mkdir -p /var/lib/quay/storage
sudo mkdir -p /var/lib/quay/postgres
```

### 4. İzinleri tekrar ver
```bash
sudo chmod -R 777 /var/lib/quay
```

### 5. Quay ağı oluştur
```bash
podman network create quay-net
```

### 6. Redis container başlat
```bash
podman run -d --name redis   --restart always   --net quay-net   registry.redhat.io/rhel8/redis-6
```

### 7. PostgreSQL container başlat
```bash
podman run -d --name postgresql   --restart always   --net quay-net   -e POSTGRESQL_USER=quayuser   -e POSTGRESQL_PASSWORD=quaypassword   -e POSTGRESQL_DATABASE=quay   -e POSTGRESQL_ADMIN_PASSWORD=adminpassword   -v /var/lib/quay/postgres:/var/lib/pgsql/data:Z   registry.redhat.io/rhel8/postgresql-13
```

### 8. PostgreSQL uzantısını etkinleştir
```bash
sleep 10
podman exec -it postgresql psql -U quayuser -d quay -c "CREATE EXTENSION IF NOT EXISTS pg_trgm;"
```

### 9. config.yaml dosyasını oluştur
```yaml
SERVER_HOSTNAME: quay.ocplab.yargitay.gov.tr
SETUP_COMPLETE: true
PREFERRED_URL_SCHEME: http
DB_URI: postgresql://quayuser:quaypassword@postgresql/quay
AUTHENTICATION_TYPE: Database
DEFAULT_TAG_EXPIRATION: 2w
TAG_EXPIRATION_OPTIONS: [0s, 1d, 1w, 2w, 4w]
FEATURE_DIRECT_LOGIN: true
FEATURE_BUILD_SUPPORT: false
FEATURE_SECURITY_SCANNER: false
FEATURE_USER_CREATION: true
FEATURE_ANONYMOUS_ACCESS: false
REGISTRY_TITLE: "Yargitay Quay Registry"
REGISTRY_TITLE_SHORT: "Yargitay Quay"
DISTRIBUTED_STORAGE_DEFAULT_LOCATIONS: []
DISTRIBUTED_STORAGE_PREFERENCE: [default]
DISTRIBUTED_STORAGE_CONFIG:
    default:
        - LocalStorage
        - storage_path: /datastorage
BUILDLOGS_REDIS:
    host: redis
    port: 6379
USER_EVENTS_REDIS:
    host: redis
    port: 6379
```

### 10. SECRET_KEY ve DATABASE_SECRET_KEY ekle
```bash
echo "SECRET_KEY: $(openssl rand -base64 30)" >> /var/lib/quay/config/config.yaml
echo "DATABASE_SECRET_KEY: $(openssl rand -base64 30)" >> /var/lib/quay/config/config.yaml

sudo chmod 777 /var/lib/quay/config/config.yaml
sudo chown 1001:0 /var/lib/quay/config/config.yaml
```

### 11. Quay container başlat
```bash
podman run -d --name quay   --restart always   --net quay-net   --sysctl net.core.somaxconn=4096   -p 8082:8080 -p 8443:8443   -v /var/lib/quay/config/config.yaml:/conf/stack/config.yaml   -v /var/lib/quay/storage:/datastorage   registry.redhat.io/quay/quay-rhel8:v3.12
```

### 12. Logları takip et
```bash
podman logs -f quay
```

---

Bu döküman, Quay Registry’nin temel kurulum adımlarını içerir.
