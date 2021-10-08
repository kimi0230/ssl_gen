## DNS 驗證, 不會auto renew
### Gen
./acme.sh  --issue --standalone --dns   -d www.localhost \
 --yes-I-know-dns-manual-mode-enough-go-ahead-please

### Renew
./acme.sh  --renew   -d www.localhost  \
  --yes-I-know-dns-manual-mode-enough-go-ahead-please