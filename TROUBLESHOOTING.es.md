# Diagnóstico de errores

[English](TROUBLESHOOTING.md) · [Inicio](README.es.md) · [Informe técnico](RECOVERY-REPORT.es.md)

Un error es una pista, no un diagnóstico. Empieza por el mensaje exacto y la fase que falló; comprueba la capa correspondiente antes de cambiar nada.

No repitas intentos a ciegas. Si los datos irremplazables están en riesgo, considera acudir a un especialista. Nunca elijas Erase ni omitas las confirmaciones de seguridad.

| Síntoma | Interpretación y comprobaciones |
| --- | --- |
| Error 100 / install_splat / Failed preallocation due to space | Examina el registro real. Este caso afectaba a la preasignación de Cryptex. No todos los errores 100 tienen esta causa. Sigue la guía original de 3SENN solo si coinciden las circunstancias. |
| iBEC enviado, sin reconexión | Comprueba auto-attach de USB/IP, lsusb y los registros del kernel. En este caso se observó una reconexión tardía. Se amplió localmente la espera, pero no se validó de forma independiente su necesidad. |
| Device failed to enter Restore Mode | El error genérico no demuestra un fallo de personalización de imágenes. Comprueba la enumeración USB en Windows y los eventos usbipd para 05ac:12ac. |
| CR_REMOVE_VETOED / PNP_VetoOutstandingOpen | Una aplicación o servicio de Windows puede mantener un recurso abierto. Cierra las aplicaciones de gestión Apple y examina los procesos; no desinstales controladores. |
| Unable to send NORData con mensaje de 65536 bytes frente al máximo de 65535 | Síntomas compatibles con el problema ZLP de USB/IP. Revisa la corrección experimental del PR #292 de usbmuxd, compila por separado y verifica que se ejecuta el daemon parcheado. |
| Unable to exit Restore Mode | No repitas una actualización a ciegas. Comprueba que no hay restauración activa, diagnostica el estado y consulta las indicaciones de recuperación antes de reiniciar. |
| DONE pero el teléfono todavía muestra progreso | La instalación desde el ordenador ha terminado; el arranque del dispositivo puede seguir en curso. No interrumpas mientras avance. |
| Hello tras actualizar | No demuestra pérdida ni conservación de datos. Confirma el acceso normal a los datos existentes antes de declarar el éxito. Detente si aparece una opción inesperada de borrado o configuración como nuevo. |

## Comprobaciones útiles

- Windows: usbipd list, usbipd state y registros de eventos de aplicaciones relevantes.
- WSL: lsusb, dmesg, pgrep -a idevicerestore, pgrep -a usbmuxd.
- Verifica rutas de ejecutables, versiones de bibliotecas y qué daemon está activo antes de modificar procesos.
- Nunca finalices una restauración activa ni desconectes el dispositivo durante la instalación.

## Privacidad antes de compartir diagnósticos

Elimina identificadores únicos (serie, ECID, UDID, IMEI), nonces, tickets de firma, nombres de usuario, rutas personales, nombres de equipo, direcciones IP y datos de cuenta. Comparte preferiblemente extractos breves con el error relevante. No publiques registros completos ni archivos de firmware.

## Referencias

- [Método Cryptex original](https://github.com/3SENN/iphone-stuck-in-recovery-full-storage)
- [usbmuxd PR #292](https://github.com/libimobiledevice/usbmuxd/pull/292)
- [usbmuxd incidencia #247](https://github.com/libimobiledevice/usbmuxd/issues/247)
- [usbipd-win](https://github.com/dorssel/usbipd-win)

Gracias a los desarrolladores originales y a quienes investigaron estos problemas. Si compartes un hallazgo, incluye el error relevante y las versiones de software, explica qué cambió y distingue las observaciones de las suposiciones.
