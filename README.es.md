# Recuperación de iPhone con almacenamiento lleno — Windows 11 y WSL2

[English](README.md) · [Informe técnico (EN)](RECOVERY-REPORT.md) · [Diagnóstico (EN)](TROUBLESHOOTING.md)

**Nueve intentos, varios obstáculos de comunicación USB y, por fin: ¡`DONE`!** 🎉

Un iPhone atrapado en Recovery tras una actualización fallida por falta de espacio consiguió completar una instalación Update desde Windows 11 mediante WSL2, sin restablecimiento de fábrica. Los datos existentes eran accesibles tras arrancar.

Si te encuentras en una situación parecida, este caso puede servirte de punto de partida: identifica el error real, resuelve un problema cada vez y aprovecha el conocimiento de quienes ya han investigado el fallo. No hay garantías de recuperación, pero cada error puede aportar información útil.

**Advertencia:** procedimiento experimental. Incluso Update puede causar pérdida de datos. Si son irremplazables, considera un especialista antes de escribir firmware. No elijas Erase ni omitas las confirmaciones de seguridad. Comprueba la compatibilidad y firma vigente del IPSW.

## ¿Es este caso relevante para ti?

| Síntoma | Por dónde empezar |
| --- | --- |
| Error 100 / `install_splat` / `Failed preallocation due to space` | [Método original de recuperación Cryptex](https://github.com/3SENN/iphone-stuck-in-recovery-full-storage) |
| Problemas de reconexión iBEC o captura de Restore Mode en Windows | [Diagnóstico de errores](TROUBLESHOOTING.md) |
| Fallo NORData con un mensaje de 65536 bytes que supera el límite de 65535 | [Informe técnico y solución alternativa USB/IP](RECOVERY-REPORT.md) |

Otros errores de restauración pueden tener causas diferentes. Comprueba el registro real antes de intentar actualizar el firmware.

## Agradecimientos — gracias a la comunidad de código abierto

La recuperación fue posible gracias a los autores de los proyectos originales:

- [3SENN](https://github.com/3SENN/iphone-stuck-in-recovery-full-storage): solución original de Cryptex.
- [hamagharbi, PR #292 de usbmuxd](https://github.com/libimobiledevice/usbmuxd/pull/292): corrección de transporte USB/IP.
- [idevicerestore](https://github.com/libimobiledevice/idevicerestore), [libirecovery](https://github.com/libimobiledevice/libirecovery), [libtatsu](https://github.com/libimobiledevice/libtatsu), [usbmuxd](https://github.com/libimobiledevice/usbmuxd).
- [usbipd-win](https://github.com/dorssel/usbipd-win), [WSL](https://github.com/microsoft/WSL), [Ubuntu](https://ubuntu.com/wsl).
- [libimobiledevice](https://github.com/libimobiledevice/libimobiledevice), [libusbmuxd](https://github.com/libimobiledevice/libusbmuxd), [libplist](https://github.com/libimobiledevice/libplist), [libimobiledevice-glue](https://github.com/libimobiledevice/libimobiledevice-glue), [libusb](https://github.com/libusb/libusb).
- Colaboradores de [usbmuxd #247](https://github.com/libimobiledevice/usbmuxd/issues/247) y [usbipd-win #924](https://github.com/dorssel/usbipd-win/issues/924).

Gracias a todos los desarrolladores, mantenedores y usuarios que compartieron sus investigaciones. Este proyecto documenta la combinación de soluciones existentes; no reivindica la autoría de los parches. Lo mejor de compartir esta experiencia es que quizá pueda ayudar a otra persona a recuperar sus datos.

## Procedimiento general

Empieza por la [guía original de Cryptex](https://github.com/3SENN/iphone-stuck-in-recovery-full-storage). El [informe técnico](RECOVERY-REPORT.md) contiene los comandos históricos y la [guía de diagnóstico](TROUBLESHOOTING.md) recoge los errores de WSL2. Estos documentos complementan la guía original, no la sustituyen.

1. Seguir la guía original de 3SENN para compilar idevicerestore con el parche Cryptex y sus dependencias.
2. Configurar WSL2 y usbipd-win para mantener la conexión durante los cambios de modo USB (Recovery y Restore Mode).
3. Diagnosticar posibles bloqueos de Windows. Cerrar las aplicaciones Apple que impidan capturar Restore Mode.
4. Si aparece el error NORData acompañado del límite 65535 bytes, revisar la corrección experimental de usbmuxd indicada en el informe técnico.
5. Calcular el tamaño Cryptex según el firmware real. En el caso documentado: 6036 MiB reales, 6784 MiB declarados. No reutilizar estos valores indiscriminadamente.
6. Seleccionar Customer Upgrade Install (IPSW), con RestoreBehavior=Update. Nunca utilizar -e, -y ni --ignore-errors.
7. Esperar a que finalice el proceso y el arranque posterior. Comprobar los datos y realizar una copia cifrada.

## Compartir el resultado

El caso se comunicó a [3SENN (incidencia #36)](https://github.com/3SENN/iphone-stuck-in-recovery-full-storage/issues/36) y en el [PR #292 de usbmuxd](https://github.com/libimobiledevice/usbmuxd/pull/292#issuecomment-7202018614). Son informes de un caso exitoso, no avales de los mantenedores. Si compartes tu propia experiencia, anonimiza los datos técnicos.

## Limitaciones

Caso único, sin reproducción independiente desde una instalación limpia. No se publican identificadores únicos, datos privados ni logs completos. Los modelos y versiones de software son referencias técnicas compartidas. No se garantiza la recuperación ni se acredita la necesidad individual de todos los cambios locales.

Si consigues llegar a `DONE`, recuerda que el siguiente objetivo es una copia de seguridad verificada. ¡Gracias por contribuir a que el conocimiento siga circulando!

## ¿Te ha resultado útil?

La documentación es de acceso gratuito. Si te ha ayudado, una estrella en GitHub, compartir un resultado anonimizado o recomendar la guía a quien la necesite son excelentes formas de contribuir. Recuerda también agradecer y apoyar a los desarrolladores originales que hicieron posible la recuperación.
