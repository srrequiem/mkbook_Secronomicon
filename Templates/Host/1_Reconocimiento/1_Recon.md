# Reconocimiento

## Escaneo de host

### Escaneo completo de puertos

```bash
sudo nmap -T5 -open -vvv --min-rate=5000 -p- -n -Pn -oG nmap/all_ports $BOX_TARGET

```

### Escaneo específico

```bash
nmap -sCV -p80,27017,37500 -oN nmap/targeted $BOX_TARGET

```