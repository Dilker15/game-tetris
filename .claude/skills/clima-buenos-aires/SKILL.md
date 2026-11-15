---
name: clima-buenos-aires
description: Consulta el clima actual de Buenos Aires, Argentina usando la API gratuita wttr.in (sin API key). Usa esta skill siempre que el usuario pregunte por el clima, el tiempo, la temperatura, si va a llover, o cómo está el día en Buenos Aires (o diga simplemente "clima", "tiempo" o "weather" sin especificar ciudad, asumiendo que se refiere a Buenos Aires salvo que indique otra ciudad).
---

# Clima de Buenos Aires

Consulta el clima actual en Buenos Aires, Argentina a través de la API pública y gratuita [wttr.in](https://wttr.in), que no requiere API key ni autenticación.

## Cómo obtener los datos

Ejecutá este comando con la herramienta Bash:

```bash
curl -s 'wttr.in/Buenos+Aires?format=j1'
```

Esto devuelve un JSON. Los campos relevantes están en `current_condition[0]`:

| Campo JSON | Significado |
|---|---|
| `temp_C` | Temperatura actual (°C) |
| `FeelsLikeC` | Sensación térmica (°C) |
| `weatherDesc[0].value` | Condición (ej. "Clear", "Partly cloudy", "Rain") |
| `humidity` | Humedad (%) |
| `windspeedKmph` | Velocidad del viento (km/h) |
| `winddir16Point` | Dirección del viento (ej. "SSW") |

Si `curl` no está disponible o la llamada falla (sin conexión, timeout, servicio caído), usá el fallback de una sola línea, mucho más liviano:

```bash
curl -s 'wttr.in/Buenos+Aires?format=3'
```

Esto devuelve un texto corto tipo `Buenos Aires: ⛅ +23°C`, suficiente si el JSON no funciona.

## Cómo presentar la respuesta

Traducí la condición climática al español (Clear → Despejado, Partly cloudy → Parcialmente nublado, Rain → Lluvia, Overcast → Nublado, etc.) y mostrá algo breve y legible, por ejemplo:

```
Clima en Buenos Aires ahora:
- Condición: Despejado
- Temperatura: 23°C (sensación térmica: 23°C)
- Humedad: 58%
- Viento: 10 km/h SSW
```

No hace falta mostrar el JSON crudo ni explicar el comando curl al usuario — solo el resumen final. Si el usuario pide un pronóstico a futuro (no solo el clima actual), el mismo JSON de `j1` incluye un array `weather` con los próximos días (`weather[].date`, `.maxtempC`, `.mintempC`, `.hourly[]`), así que podés usarlo sin hacer una llamada adicional.
