# AeroDist – Base de Datos Legada (PostgreSQL)

**Laboratorio 3 – Sistemas Distribuidos 2026-2 – UTFSM**

Esta es la imagen de la **Base de Datos Legada** de la sección 3.3 del enunciado. **No deben construirla**: solo cargarla en la **MV1**, levantarla y conectarse a ella desde el **Cliente Emisor** y el **Extractor Histórico**.

---

## 1. Obtener la imagen en la MV1

La imagen es pública en GitHub Container Registry:

```bash
docker pull ghcr.io/felipemarchantvargas/aerodist-bd:1.0
```

No hace falta ejecutar este comando aparte: `docker compose up` la descarga automáticamente la primera vez.

**Si el `pull` falla** (por ejemplo, por problemas con el registry), descarguen el respaldo desde el Release y cárguenlo:

```bash
curl -LO https://github.com/FelipeMarchantVargas/sd-lab3-bd-legada/releases/download/v1.0/aerodist-bd_1.0.tar.gz
docker load -i aerodist-bd_1.0.tar.gz
```

En ambos casos debe aparecer `ghcr.io/felipemarchantvargas/aerodist-bd:1.0` en `docker images`.

## 2. Levantarla

Copien el bloque `bd-legada` de `docker-compose.yml` al compose de su MV1 (junto con el volumen `bd-legada-datos`) y ejecuten `make docker-VM1`, o pruébenla sola con:

```bash
docker compose up -d
```

La primera vez la base se crea y se puebla automáticamente (unos segundos). Cuando el contenedor queda **`healthy`** (`docker ps`), los datos ya están cargados. Mientras tanto, sus entidades deben **reintentar** la conexión (sección 6 del enunciado).

> Los datos se generan solo la primera vez que se crea el volumen. Para partir de cero: `docker compose down -v`.

## 3. Datos de conexión

| Parámetro  | Valor                    |
|------------|--------------------------|
| Host       | `bd-legada` dentro del compose de la MV1 (o la IP de la MV1 si publican el puerto) |
| Puerto     | `5432`                   |
| Base       | `aerodist`                 |
| Usuario    | `aerodist_lector`          |
| Contraseña | `lector_lab3_2026`       |

- El usuario es de **solo lectura**: solo permite `SELECT`. Cualquier `INSERT`, `UPDATE`, `DELETE`, `TRUNCATE` o `CREATE` es rechazado por la base.
- **No escriban estos valores en el código.** Van en el `.env` de la MV1 (ver `.env.example`). En la evaluación pueden ser otros.

Con `lib/pq` o `pgx`, la cadena de conexión tiene esta forma:

```
host=<DB_HOST> port=<DB_PORT> user=<DB_USER> password=<DB_PASSWORD> dbname=<DB_NAME> sslmode=disable
```

## 4. Esquema definitivo

Este esquema reemplaza al esquema referencial del Listado 1 del enunciado.

```sql
CREATE TABLE telemetria_tiempo_real (
    id                SERIAL PRIMARY KEY,
    id_vuelo          VARCHAR(20),
    aerolinea         VARCHAR(30),   -- GopherAir, KubeJet, PacketWings, LatencyAir
    estado            VARCHAR(20),   -- En Camino, Aterrizado, Retrasado, Cancelado
    timestamp_emision TIMESTAMP
);

CREATE TABLE telemetria_historica (
    -- mismas columnas que telemetria_tiempo_real
);
```

| Tabla | Uso | Volumen |
|---|---|---|
| `telemetria_tiempo_real` | La lee el **Cliente Emisor**, recorriéndola en orden por `id`. | Cientos de registros |
| `telemetria_historica` | La lee el **Extractor Histórico**, paginando (`LIMIT/OFFSET` o rangos de `id`). | Cientos de miles |

- `id` es correlativo y `timestamp_emision` crece con `id`.
- Un mismo `id_vuelo` puede aparecer en varios registros (distintos eventos de un mismo vuelo).
- Formato del ID canónico: `GA0421` (prefijo según aerolinea: `GA`, `KJ`, `PW`, `LY`).

## 5. Datos con ruido

Como indica el enunciado, la base contiene inconsistencias **intencionales**. Su tratamiento es responsabilidad de las **Procesadoras**; el Emisor y el Extractor publican los registros **tal cual** los leen:

- IDs en minúsculas (`ga0421`).
- Estados con distinto formato (`aterrizado`, `ATERRIZADO`).
- Campos vacíos (`''`) o `NULL`, en cualquier columna salvo `id`.
- Estados no reconocidos, que no pertenecen a la lista canónica.

> En la evaluación se usará una base con **contenido y volumen distintos**, con el mismo esquema. No asuman una cantidad fija de registros, IDs ni proporciones de ruido. Para saber cuántos registros hay, usen `COUNT(*)`.

## 6. Preguntas frecuentes

- **¿Puedo crear índices o vistas?** No. El usuario no tiene permisos y la regla del enunciado lo prohíbe.
- **¿Puedo usar el usuario administrador?** No. Sus entidades deben usar el usuario lector.
- **La conexión falla al levantar la MV.** Es esperable mientras la base se inicializa. Implementen reintentos.

Consultas: foro de Aula o felipe.marchantv@usm.cl / nelson.sepulvedav@usm.cl.
