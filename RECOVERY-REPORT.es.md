# Informe técnico de recuperación

[English](RECOVERY-REPORT.md) · [Inicio](README.es.md) · [Diagnóstico](TROUBLESHOOTING.es.md)

De los errores repetidos de recuperación a un arranque exitoso: este informe recoge qué cambió entre intentos, qué se observó y qué sigue siendo incierto.

## Entorno

Configuración histórica: Windows 11; Ubuntu 26.04 en WSL2; usbipd-win 5.3.0; iPhone 15, 128 GB, iPhone15,4 / d37ap; iOS 27.2.1, compilación 24B156. Son identificadores de producto no únicos; no se incluyen identificadores particulares del dispositivo.

- idevicerestore, base 60192e9, con el [parche Cryptex original](https://github.com/3SENN/iphone-stuck-in-recovery-full-storage) y una modificación local del tiempo de espera de iBEC.
- libirecovery, base d767ca5, con gestión local de referencias hotplug y trazas de diagnóstico.
- libtatsu cdbca26.
- usbmuxd 1.1.1-73-g4afca63, [PR #292](https://github.com/libimobiledevice/usbmuxd/pull/292).

No se aisló la necesidad de cada modificación local. Se trata de un informe histórico, no de una guía completa de compilación desde una instalación limpia.

## Evolución de los fallos

| Intentos | Observación |
| --- | --- |
| 1–2 | TSS completado; agotado el tiempo de reconexión de iBEC. |
| 3 | Reconexión iBEC observada después del límite original de 60 segundos. Espera local ampliada a 600 segundos. |
| 4–5 | Enviados iBEC y componentes de arranque; no se capturó Restore Mode. |
| 6 | Añadido AutoBind; Windows informó PNP_VetoOutstandingOpen para la interfaz Restore Mode. |
| 7 | Tras detener procesos Apple que interferían, se capturó Restore Mode; NORData falló por un mensaje de transporte demasiado grande. |
| 8 | Falló el reinicio por software para salir de Restore Mode. El dispositivo volvió posteriormente a Recovery mediante un reinicio manual. |
| 9 | usbmuxd parcheado activo; instalación Upgrade completada y datos existentes accesibles tras arrancar. |

Fallo original de almacenamiento (extractos de diagnóstico):
```text
Failed preallocation due to space
No hope in this succeeding, bailing.
FAILURE:100
```

La preasignación fallida fue de 6329204736 bytes para Cryptex1,SystemOS.

Fallo de transporte:
```text
Unable to send NORData
message was too large (65536 bytes, max = 65535)
```

## Referencia de comandos históricos

Los comandos son ilustrativos. Revisa los requisitos originales, comprueba la firma y compatibilidad del firmware y sustituye los marcadores localmente. Nunca compartas el ECID ni registros completos. Update escribe firmware y puede ocasionar pérdida de datos. No uses opciones de borrado ni omitas las comprobaciones de seguridad.

En PowerShell como administrador, usa el BUSID real de usbipd list. Las reglas afectan a los tipos de dispositivo coincidentes y deben retirarse después.

```powershell
usbipd policy add --effect allow --operation AutoBind --hardware-id 05ac:1281
usbipd policy add --effect allow --operation AutoBind --hardware-id 05ac:12ac
# Replace YOUR_BUSID before running:
usbipd bind --busid YOUR_BUSID
usbipd attach --wsl --busid YOUR_BUSID --auto-attach
```

Mantén auto-attach activo durante la actualización. Si Windows informa de un veto por un recurso abierto, cierra las aplicaciones de gestión de Apple. Una vez comprobado que no hay ninguna otra operación Apple activa, la intervención histórica fue:

```powershell
Stop-Process -Name AppleMobileDeviceProcess,iPodService,iTunesHelper -Force -ErrorAction SilentlyContinue
```

También se detuvo temporalmente Bonjour, aunque no se demostró que fuera necesario. No desinstales controladores.

En Ubuntu, sigue los requisitos de compilación originales de usbmuxd. Compila la corrección por separado, sin instalarla globalmente:

```bash
cd ~
git clone https://github.com/libimobiledevice/usbmuxd.git usbmuxd-fix
cd usbmuxd-fix
git fetch origin pull/292/head:fix-usbipd-zlp
git switch --detach 4afca633b8460416e39dab3fc0e8ae10123cfec6
./autogen.sh --prefix="$HOME/usbmuxd-fix/install"
make -j4
./src/usbmuxd --version
ldd ./src/usbmuxd
```

Comprueba las dependencias y revisa el código. Sin ninguna restauración activa, detén el daemon anterior de forma ordenada e inicia el ejecutable parcheado en su propia terminal:

```bash
sudo "$HOME/usbmuxd-fix/src/usbmuxd" -f -v -v
```

Comprueba que solo se ejecuta un daemon. Usa lsusb e irecovery -q localmente para confirmar Recovery; no publiques su salida sin anonimizarla.

La guía original de Cryptex proporciona la herramienta de cálculo del tamaño. Sin pasarle un registro de restauración, indicó 6036 MiB reales y sugirió 6784 MiB (+748 MiB) para la compilación histórica; no es un valor universal. Sigue las instrucciones originales de cálculo para el dispositivo y firmware reales. Verifica la identidad Upgrade de BuildManifest y RestoreBehavior=Update. La estructura del comando exitoso fue:

```bash
ECID='YOUR_ECID'
IPSW='/path/to/compatible-signed-firmware.ipsw'
CRYPTEX_MIB='YOUR_CALCULATED_SIZE'
sudo env LD_LIBRARY_PATH="$HOME/libirecovery/install/lib" \
  IDEVICERESTORE_CRYPTEX_SIZE_MIB="$CRYPTEX_MIB" \
  "$HOME/idevicerestore/install/bin/idevicerestore" \
  -d -P --variant 'Customer Upgrade Install (IPSW)' \
  -i "$ECID" --logfile="$HOME/iphone-update.log" "$IPSW"
```

Esto presupone los prefijos de compilación locales históricos. Comprueba los marcadores y dependencias antes de ejecutar. Nunca pases -e, -y ni --ignore-errors. Una comprobación sin acción (-n) no simula toda la restauración.

## Resultado exitoso — ¡por fin DONE!

```text
Status: Restore Finished
DONE
progress: 7 1.000000
```

El dispositivo completó su propia barra de progreso posterior, mostró Hello y los datos existentes eran accesibles tras desbloquearlo. No se realizó una comparación forense independiente de cada archivo. Crea pronto una copia cifrada, conserva su contraseña y verifica la copia antes de eliminar datos.

## Limpieza

Solo cuando la actualización y el arranque hayan terminado: detén el reenvío USB temporal y el daemon parcheado, desconecta de WSL si procede, elimina únicamente las reglas AutoBind añadidas (consulta usbipd policy remove --help) y restablece los servicios que estaban activos cuando corresponda.

## Créditos

Consulta los [agradecimientos principales](README.es.md). El código y los parches originales siguen atribuidos a sus autores. Este informe aporta observaciones sobre la integración de sus soluciones en Windows/WSL2.
