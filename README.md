
## Free DNS
https://my.freenom.com/clientarea.php

## DNS 驗證, 不會auto renew
### Gen
acme.sh  --issue  -d localhost --standalone

### Renew
./acme.sh  --renew   -d localhost  \
  --yes-I-know-dns-manual-mode-enough-go-ahead-please