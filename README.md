# Lab 3 SD 2026-2: Base de Datos Legada

**Sistemas Distribuidos – UTFSM – Segundo Semestre 2026**

Este repositorio contiene la **Base de Datos Legada (PostgreSQL)** de la sección 3.3 del enunciado. Cada grupo usa **solo la variante de su enunciado**:

| Enunciado | Carpeta | Imagen |
|---|---|---|
| AeroDist (vuelos) | [`aerodist-bd/`](aerodist-bd/) | `ghcr.io/felipemarchantvargas/aerodist-bd:1.0` |
| DistriEats (pedidos) | [`distrieats-bd/`](distrieats-bd/) | `ghcr.io/felipemarchantvargas/distrieats-bd:1.0` |

El `LEEME.md` de cada carpeta trae el esquema definitivo, las credenciales de solo lectura, cómo levantar la base y la descripción del ruido en los datos. **Léanlo completo.**

## Descargar los archivos directamente en la MV1

No hace falta copiar nada desde su computador: la MV puede descargar los archivos directamente. Reemplacen `<variante>` por `aerodist` o `distrieats`.

```bash
curl -LO https://github.com/FelipeMarchantVargas/sd-lab3-bd-legada/releases/download/v1.0/entrega-<variante>-bd.zip
```

```bash
unzip entrega-<variante>-bd.zip && cd <variante>-bd
```

Si la MV no tiene `unzip`, usen `python3 -m zipfile -e entrega-<variante>-bd.zip .`.

Si tienen `git`, también pueden clonar el repositorio:

```bash
git clone https://github.com/FelipeMarchantVargas/sd-lab3-bd-legada.git
```

## Levantar la base para probar

```bash
docker compose up -d && docker ps
```

Cuando el contenedor `bd-legada` aparece como `(healthy)`, los datos ya están cargados. Para integrarla a su entrega, copien el servicio `bd-legada` al `docker-compose.yml` de su MV1.

## Respaldo de la imagen

Si `docker pull` desde `ghcr.io` falla, el Release también trae la imagen como archivo:

```bash
curl -LO https://github.com/FelipeMarchantVargas/sd-lab3-bd-legada/releases/download/v1.0/<variante>-bd_1.0.tar.gz
```

```bash
docker load -i <variante>-bd_1.0.tar.gz
```

## Consultas

Foro de Aula, o felipe.marchantv@usm.cl / nelson.sepulvedav@usm.cl.
