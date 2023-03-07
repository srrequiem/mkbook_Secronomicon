# Nombre de máquina

Write-up de la máquina <nombre_de_maquina> de la plataforma [echoCTF](https://echoCTF.red).

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
- [Ubicación de banderas](#ubicación-de-banderas)
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
| Nombre | [nombre](https://echoctf.red/target/) |
| Dificultad | Expert |
| Banderas | 11 (7: other, 2: system, env, root) |
| Puntos | 6,600: (other: 1,600, system: 2,600, env: 900, root: 1,500) |
| Descripción/Pistas |  |

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


## Ubicación de banderas

1. (other: 100 pts x2) `/app/frontend/forms.html` x2.
2. (other: 100 pts) `/app/frontend/forms.go`.
3. (other: 100 pts) `/app/frontend/ETSCTF.html`.
4. (other: 400 pts) `/app/backend/views/etsctf.ejs`.
5. (other: 500 pts) mysql event (Revisar [eventos mysql](#eventos-mysql)).
6. (other: 300 pts) scrapper databases (visto desde la plataforma interna o dentro de la base de datos).
7. (system: 1300 pts) `/etc/passwd`.
8. (system: 1300 pts) `/etc/shadow`.
9. (env: 900 pts) `strings /proc/1/environ | grep ETSCTF_`.
10. (root: 1500 pts) `/root`

## Conclusión


## Notas adicionales


## Referencias

