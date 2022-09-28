# Templates

## Box

### Distribución de archivos

```bash
Box
├── images
└── README.md
```

### Contenido de `README.md`

````markdown
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

# Enumeración

## Servicios

### Nombre de servicio - Puerto

#### Manual



#### Nombre de herramienta



# Explotación

## Tipo de explotación

### Pasos previos | Preparación



### Ejecución



# Post Explotación

## Enumeración



## Escalación de privilegios


# Referencias

````