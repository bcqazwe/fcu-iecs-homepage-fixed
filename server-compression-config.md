# Priority 4: 伺服器壓縮設定

Live Server 開發環境無法設定壓縮，以下為正式部署時的設定。

## Nginx

```nginx
# 啟用 gzip 壓縮
gzip on;
gzip_vary on;
gzip_min_length 256;
gzip_proxied any;
gzip_comp_level 6;
gzip_types
    text/html
    text/css
    text/javascript
    application/javascript
    application/json
    image/svg+xml
    application/xml;

# 靜態資源快取（帶 hash 的檔案）
location ~* \.[a-f0-9]{12}\.(css|js|svg|ico|png)$ {
    expires 1y;
    add_header Cache-Control "public, max-age=31536000, immutable";
}

# 圖片快取
location ~* \.(jpg|jpeg|png|webp|avif|gif)$ {
    expires 30d;
    add_header Cache-Control "public, max-age=2592000";
}
```

## Apache (.htaccess)

```apache
# 啟用壓縮
<IfModule mod_deflate.c>
    AddOutputFilterByType DEFLATE text/html text/css application/javascript image/svg+xml application/json
</IfModule>

# 快取設定
<IfModule mod_expires.c>
    ExpiresActive On
    ExpiresByType image/jpeg "access plus 30 days"
    ExpiresByType image/png "access plus 30 days"
    ExpiresByType image/webp "access plus 30 days"
    ExpiresByType text/css "access plus 1 year"
    ExpiresByType application/javascript "access plus 1 year"
</IfModule>
```

## Brotli 壓縮（推薦，比 gzip 小 15-20%）

```nginx
# Nginx Brotli（需安裝 ngx_brotli 模組）
brotli on;
brotli_comp_level 6;
brotli_types text/html text/css application/javascript application/json image/svg+xml;
```

## Back/Forward Cache (bfcache)

確保不使用 `unload` 事件監聽器（改用 `pagehide`），並且 HTTP headers 不禁止 bfcache：

```nginx
# 不要對 HTML 設定 no-store
location / {
    add_header Cache-Control "no-cache, must-revalidate";
    # 避免 Cache-Control: no-store，它會阻止 bfcache
}
```

## 安全性 Headers（不影響效能）

```nginx
# 推薦同時加入的安全性 headers
add_header X-Content-Type-Options "nosniff" always;
add_header X-Frame-Options "SAMEORIGIN" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
```
