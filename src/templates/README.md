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
# Tabla de Contenidos

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


````

## Compendio CTF / Writeup

```markdown
# Nombre CTF

## Tabla de Contenidos

## Resumen de Vulnerabilidades

## Web

### Web Reto 1

#### Info

| Atributo | Valor |
|---|---|
| Descripción | - |
| Archivos | - |

#### Solución

## Crypto

### Crypto Reto 1

#### Info

| Atributo | Valor |
|---|---|
| Descripción | - |
| Archivos | - |

#### Solución

## Reversing

### Reversing Reto 1

#### Info

| Atributo | Valor |
|---|---|
| Descripción | - |
| Archivos | - |

#### Solución

## Pwning

### Pwning Reto 1

#### Info

| Atributo | Valor |
|---|---|
| Descripción | - |
| Archivos | - |

#### Solución

## Forense

### Forense Reto 1

#### Info

| Atributo | Valor |
|---|---|
| Descripción | - |
| Archivos | - |

#### Solución

```
