# Recuperación de iPhone con almacenamiento lleno — Windows 11 y WSL2

[English](README.md) · [Informe técnico (EN)](RECOVERY-REPORT.md) · [Diagnóstico (EN)](TROUBLESHOOTING.md)

Caso técnico anonimizado: un iPhone atrapado en Recovery tras una actualización fallida por falta de espacio consiguió completar una instalación Update sin solicitar borrado de fábrica. Los datos existentes eran accesibles tras arrancar.

**Advertencia:** procedimiento experimental. Incluso Update puede causar pérdida de datos. Si son irremplazables, considera un especialista antes de escribir firmware. No elijas Erase ni omitas las confirmaciones de seguridad. Comprueba la compatibilidad y firma vigente del IPSW.

## Agradecimientos

La recuperación fue posible gracias a los autores de los proyectos originales:

- [3SENN](https://github.com/3SENN/iphone-stuck-in-recovery-full-storage): solución original de Cryptex.
- [hamagharbi, PR #292 de usbmuxd](https://github.com/libimobiledevice/usbmuxd/pull/292): corrección de transporte USB/IP.
- [idevicerestore](https://github.com/libimobiledevice/idevicerestore), [libirecovery](https://github.com/libimobiledevice/libirecovery), [libtatsu](https://github.com/libimobiledevice/libtatsu), [usbmuxd](https://github.com/libimobiledevice/usbmuxd).
- [usbipd-win](https://github.com/dorssel/usbipd-win), [WSL](https://github.com/microsoft/WSL), [Ubuntu](https://ubuntu.com/wsl).
- [libimobiledevice](https://github.com/libimobiledevice/libimobiledevice), [libusbmuxd](https://github.com/libimobiledevice/libusbmuxd), [libplist](https://github.com/libimobiledevice/libplist), [libimobiledevice-glue](https://github.com/libimobiledevice/libimobiledevice-glue), [libusb](https://github.com/libusb/libusb).
- Colaboradores de [usbmuxd #247](https://github.com/libimobiledevice/usbmuxd/issues/247) y [usbipd-win #924](https://github.com/dorssel/usbipd-win/issues/924).

Gracias a todos los desarrolladores, mantenedores y usuarios que compartieron sus investigaciones. Este proyecto documenta la combinación de soluciones existentes; no reivindica la autoría de los parches.

## Procedimiento general

1. Seguir la guía original de 3SENN para compilar idevicerestore con el parche Cryptex y sus dependencias.
2. Configurar WSL2 y usbipd-win para mantener la conexión durante los cambios de modo USB (Recovery y Restore Mode).
3. Diagnosticar posibles bloqueos de Windows. Cerrar las aplicaciones Apple que impidan capturar Restore Mode.
4. Si aparece el error NORData acompañado del límite 65535 bytes, utilizar la versión corregida de usbmuxd indicada en el informe técnico.
5. Calcular el tamaño Cryptex según el firmware real. En el caso documentado: 6036 MiB reales, 6784 MiB declarados. No reutilizar estos valores indiscriminadamente.
6. Seleccionar Customer Upgrade Install (IPSW), con RestoreBehavior=Update. Nunca utilizar -e, -y ni --ignore-errors.
7. Esperar a que finalice el proceso y el arranque posterior. Comprobar los datos y realizar una copia cifrada.

## Limitaciones

Caso único, sin reproducción independiente desde una instalación limpia. No se publican identificadores únicos, datos privados ni logs completos. Los modelos y versiones de software son referencias técnicas compartidas. No se garantiza la recuperación ni se acredita la necesidad individual de todos los cambios locales.
