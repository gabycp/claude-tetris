---
name: clima-local
description: Consulta el clima actual (temperatura, condición, viento, humedad) detectando la ubicación automáticamente por IP, usando APIs públicas gratuitas sin necesidad de API key. Úsalo cuando el usuario pregunte por el clima, temperatura, si va a llover, si necesita paraguas/abrigo, o pida un pronóstico, sin especificar una herramienta distinta.
---

# Clima local

Este skill obtiene el clima actual del usuario detectando su ubicación
automáticamente a partir de su IP pública, sin pedirle que indique una
ciudad y sin requerir ninguna API key.

## Pasos

1. **Fuente principal — wttr.in**: ejecuta

   ```bash
   curl -s 'https://wttr.in/?format=j1'
   ```

   wttr.in detecta la ubicación automáticamente a partir de la IP de origen
   de la petición. La respuesta JSON trae, entre otros campos:
   - `current_condition[0].temp_C` / `FeelsLikeC`
   - `current_condition[0].weatherDesc[0].value`
   - `current_condition[0].humidity`
   - `current_condition[0].windspeedKmph`
   - `nearest_area[0].areaName[0].value` (ciudad detectada)

   Si solo necesitas una respuesta rápida en una línea (sin parsear JSON),
   usa en su lugar:

   ```bash
   curl -s 'https://wttr.in/?format=3'
   ```

   que devuelve algo como `Santiago: ⛅️ +14°C`.

2. **Fallback — si wttr.in no responde o da timeout** (usa `curl -s --max-time 8`
   para no colgarte esperando):

   a. Geolocaliza por IP:

      ```bash
      curl -s https://ipinfo.io/json
      ```

      Extrae `city`, `region`, `country` y `loc` (viene como `"lat,lon"`).

   b. Con esas coordenadas, consulta Open-Meteo (sin API key):

      ```bash
      curl -s "https://api.open-meteo.com/v1/forecast?latitude=<LAT>&longitude=<LON>&current_weather=true"
      ```

      Devuelve `current_weather.temperature`, `.windspeed` y `.weathercode`
      (código WMO; tradúcelo a una descripción legible, ej. 0 = despejado,
      1-3 = parcialmente nublado, 45/48 = niebla, 51-67 = llovizna/lluvia,
      71-86 = nieve, 95-99 = tormenta).

3. **Presenta el resultado** de forma breve y directa: ciudad detectada,
   condición, temperatura (y sensación térmica si está disponible). No
   muestres el JSON crudo al usuario salvo que lo pida explícitamente.

## Notas

- No requiere autenticación ni claves de API.
- Si el usuario pide el clima de otra ciudad distinta a la detectada por IP,
  pásala como parámetro: `curl -s 'https://wttr.in/Buenos+Aires?format=j1'`
  o resuelve sus coordenadas antes de llamar a Open-Meteo.
- Ambos servicios son de terceros: si ambos fallan, informa al usuario que
  no se pudo obtener el clima en este momento en lugar de inventar datos.
