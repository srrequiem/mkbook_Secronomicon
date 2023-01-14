# Nombre de máquina

## Tabla de Contenido <!-- omit from toc -->

- [Introducción](#introducción)
  - [Técnicas vistas / Tags](#técnicas-vistas--tags)
  - [Estadísticas](#estadísticas)
- [Reconocimiento](#reconocimiento)
  - [Escaneo de host](#escaneo-de-host)
    - [Escaneo completo de puertos](#escaneo-completo-de-puertos)
    - [Escaneo específico](#escaneo-específico)
- [Enumeración](#enumeración)
  - [Servicios](#servicios)
    - [Nombre de servicio - Puerto](#nombre-de-servicio---puerto)
      - [Manual](#manual)
      - [Nombre de herramienta](#nombre-de-herramienta)
- [Explotación](#explotación)
  - [Tipo de explotación](#tipo-de-explotación)
    - [Pasos previos | Preparación](#pasos-previos--preparación)
    - [Ejecución](#ejecución)
- [Post Explotación](#post-explotación)
  - [Enumeración](#enumeración-1)
  - [Escalación de privilegios](#escalación-de-privilegios)
- [Conclusión](#conclusión)
- [Notas adicionales](#notas-adicionales)
- [Referencias](#referencias)


## Introducción

### Técnicas vistas / Tags

- Técnica 1
- Técnica 2
- Técnica 3

### Estadísticas

| Característica | Descripción |
|---|---|
| Nombre | [Box]() |
| OS | Windows / Linux |
| Dificultad oficial | Easy |
| Dificultad de comunidad | ![Dificultad]() |
| Puntos | 20 |
| Creadores | [Creator]() |

## Reconocimiento

### Escaneo de host

#### Escaneo completo de puertos

```bash
sudo nmap -T5 -open -vvv --min-rate=5000 -p- -n -Pn -oG nmap/all_ports $BOX_TARGET

```

#### Escaneo específico

```bash
nmap -sCV -p80,27017,37500 -oN nmap/targeted $BOX_TARGET

```

## Enumeración

### Servicios

#### Nombre de servicio - Puerto

##### Manual



##### Nombre de herramienta



## Explotación

### Tipo de explotación

#### Pasos previos | Preparación



#### Ejecución



## Post Explotación

### Enumeración



### Escalación de privilegios


## Conclusión


## Notas adicionales


## Referencias

