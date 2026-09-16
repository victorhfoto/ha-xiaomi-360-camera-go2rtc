# Solución de problemas

[← Volver a la guía](../README.md)

Para ver qué está pasando, abre **Ajustes → Complementos → go2rtc (master) → Registro**.

Si necesitas más detalle de `ffmpeg`, añade esto a `go2rtc.yaml`, reinicia el complemento y, cuando termines, quítalo:

```yaml
log:
  exec: debug
```

---

## El login falla con `context deadline exceeded`

```text
ERROR: Get "https://sts.api.io.mi.com/sts?...": context deadline exceeded
(Client.Timeout exceeded while awaiting headers)
```

El login llega a enviar el código por email, pero el último paso (`sts.api.io.mi.com`) no responde a tiempo.

1. Espera un minuto y repite todo el login, código incluido.
2. Si tienes AdGuard Home o Pi-hole, busca `mi.com` en el registro de consultas y comprueba que no se bloquea nada.
3. Comprueba que el DNS de Home Assistant funciona: **Ajustes → Sistema → Red**. Configura un DNS secundario para que un fallo puntual del principal no corte la conexión.

## Después del login no aparece ninguna cámara

La región del desplegable no es la de tu cuenta. Por defecto está en **China**. En Europa elige **Europe**, que en la URL aparece como `:de@`.

## El stream original funciona pero el H.264 da `exit status 183`

Revisa en la pestaña **Config** que la línea sea exactamente:

```yaml
entrada_h264:
  - ffmpeg:entrada#video=h264
```

Si aparece solo `ffmpeg:entrada`, la parte `#video=h264` se ha perdido. Suele pasar al crear el stream desde la API HTTP sin escapar el `#`.

## En el registro aparecen `Could not find ref with POC` o `Error constructing the frame RPS`

Es normal al arrancar la conversión a H.264. `ffmpeg` empieza a leer a mitad de un grupo de fotogramas H.265 y descarta los incompletos hasta el siguiente fotograma clave. Si después aparece `run rtsp launch=...`, todo va bien.

## Home Assistant da error o tiempo de espera al guardar la Generic Camera

Home Assistant prueba el stream al guardar. El stream H.264 tarda unos 3–6 s en arrancar, y la primera conexión P2P con la cámara puede tardar más.

1. Comprueba que `preload` está configurado para el stream original.
2. Vuelve a guardar.
3. Si sigue fallando, guarda con `rtsp://127.0.0.1:8554/entrada` y cambia a `entrada_h264` después desde **Configurar**.

## Se ve en el iPhone pero no en una tablet Android o en el PC

Es un problema de códec. La cámara emite en H.265 y muchos navegadores no lo decodifican. Usa el stream `entrada_h264` como *Fuente de stream* en la Generic Camera.

## go2rtc no conecta con la cámara (`i/o timeout`, `EOF`, `401`, `permit deny`)

- Comprueba que Home Assistant llega a la IP de la cámara. Revisa VLANs, firewall y el **aislamiento de clientes** del Wi-Fi.
- Asegúrate de que la cámara tiene IP fija. Si cambia, actualiza `<IP_CAMARA>` en la URL.
- Busca tu modelo en la [lista de cámaras conocidas](https://github.com/AlexxIT/go2rtc/issues/1982). Algunos modelos antiguos (TUTK) solo funcionan con la versión *master* o *dev*, y otros todavía no están soportados.
- Prueba una calidad menor añadiendo `&subtype=sd` a la URL `xiaomi://`.

## La tablet no muestra el apartado Medios

Si usas Custom Sidebar, revisa la excepción del usuario de la tablet (`user:` o `is_admin: false`). Luego recarga la página o reinicia la tablet, porque la configuración se lee al cargar el frontend.

## La CPU sube mientras se ve la cámara

Es la conversión a H.264 por software. Tienes estas opciones:

- Usar el stream original (H.265) en los dispositivos que lo soporten.
- Reducir la resolución de la conversión, por ejemplo con `ffmpeg:entrada#video=h264#width=1280`.
- Usar la variante **go2rtc (hardware)** para aprovechar la aceleración por hardware (Intel/AMD VAAPI). Así se transcodifica con la GPU. Consulta la documentación de go2rtc sobre `#hardware`.
