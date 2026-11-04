## /opt/Mythic/.env を編集します。
```text
NGINX_PORT="443"
sed -i 's/NGINX_PORT="7443"/NGINX_PORT="443"/g' .env
```
- 既存と同じ ECDSA P-384 + SHA-384 の自己署名証明書とSEC1形式の秘密鍵
```
sudo openssl req -x509 -newkey ec \
  -pkeyopt ec_paramgen_curve:secp384r1 \
  -nodes \
  -keyout mythic-ssl.key \
  -out mythic-cert.crt \
  -sha384 \
  -days 365 \
  -subj "/CN=c2.example.com" \
  -addext "subjectAltName=DNS:c2.example.com"
```

```
sudo openssl req -x509 -newkey rsa:4096 \
  -nodes \
  -keyout mythic-ssl.key \
  -out mythic-cert.crt \
  -sha256 \
  -days 365 \
  -subj "/CN=c2.example.com" \
  -addext "subjectAltName=DNS:c2.example.com"
```

```
sudo chown root:root mythic-ssl.key mythic-cert.crt
sudo chmod 600 mythic-ssl.key
sudo chmod 644 mythic-cert.crt
cd /home/mythic/Mythic
sudo ./mythic-cli build
sudo ./mythic-cli start
```
