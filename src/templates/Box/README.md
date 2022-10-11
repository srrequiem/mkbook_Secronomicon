# Introducción

## Técnicas vistas / Tags

- Técnica 1
- Técnica 2
- Técnica 3

## Estadísticas

| Característica | Descripción |
|---|---|
| Nombre | [Box]() |
| OS | Windows / Linux |
| Dificultad oficial | Easy |
| Dificultad de comunidad | ![Dificultad]() |
| Puntos | 20 |
| Creadores | [Creator]() |

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


# Conclusión


# Notas adicionales


# Referencias

