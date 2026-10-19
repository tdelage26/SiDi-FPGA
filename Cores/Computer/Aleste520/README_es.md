# Aleste 520EX para placas FPGA de la familia MiST

El **Aleste 520EX** es un ordenador fabricado en Rusia a principios de los
90 (Patisonic / Foton): un compatible Amstrad CPC 6128 que añade un modo
propio con MSX-DOS, gráficos de alta resolución de 16 colores, un mapper de
memoria de 1 MB, reloj en tiempo real y más. Este core lo recrea en placas de
la familia MiST. Se basa en **Aleste LX** de hww
(github.com/hww/aleste-lx-fpga-sv).

[English version](README.md)

| Placa | FPGA | Proyecto | Audio |
|---|---|---|---|
| Poseidon | Cyclone IV GX EP4CGX150 | `poseidon/aleste_poseidon.qpf` | I2S |
| MiST | Cyclone III EP3C25 | `mist/aleste_mist.qpf` | sigma-delta |
| SiDi | Cyclone IV E EP4CE22 | `sidi/aleste_sidi.qpf` | sigma-delta |
| Calypso (CYC1000) | Cyclone 10 LP 10CL025 | `calypso/aleste_calypso.qpf` | I2S |

## Características

- **Dos máquinas en una**, a elegir desde el OSD (*CPC boot*):
  - **Modo AMSTRAD**: compatible con el CPC 6128, BASIC 1.1 y AMSDOS. Carga
    de cintas desde ficheros CDT/TZX o desde un magnetófono por la entrada de
    audio de la placa (no en la MiST), discos de AMSDOS (DSK/EDSK) y ratones
    del CPC (Kempston, SYMBiFACE II, Multiplay, AMX).
  - **Modo Aleste**: el BOOTMENU y el SETUP del propio 520EX, y **MSX-DOS 1**
    con la BIOS del 520EX: modos de 640x200 y 320x200 en 16 colores con paleta
    de 64, mapper de 1 MB, ratón serie y dos disqueteras.
- **Z80** a 4 MHz, con el modo turbo del 520EX (8 MHz) y 16 MHz.
- **Vídeo**: Gate Array, CRTC 6845 (UM6845R) y los modos de alta resolución
  del 520EX. Salida VGA (doblador de líneas, scanlines) o RGB de 15 kHz;
  centrado horizontal y vertical.
- **Sonido**: AY-3-8912 (mono o estéreo ABC), tarjeta **MagicSound** (cuatro
  canales, DMA 8237 y temporizador 8254) y sonido de la cinta.
- **Controlador de disquete uPD765** con dos disqueteras.
- **Reloj MC146818**, puesto en hora desde el reloj del firmware de MiST, y
  **CMOS** (ajustes del SETUP) guardado en la tarjeta SD.
- **Teclado** con la matriz del 520EX, incluidas las teclas rusas y F1-F5,
  HELP e INS.

## Tarjeta SD

Copia el contenido de `sd/` a la raíz de la tarjeta SD (detalles en
[sd/README.md](sd/README.md)):

- `ALESTE.ROM`: ROM del sistema; se carga sola.
- `ALESTE.HD2`: CMOS (ajustes del SETUP); se monta solo.
- `dsk/ALESTDOS_TIME.dsk`: disco de arranque de MSX-DOS con `TIME.COM` y dos
  disqueteras. Está en una subcarpeta para que no se monte solo: móntalo desde
  el OSD, o cópialo a la raíz como `ALESTE.HD0`.

## Herramientas

Todas en `tools/` (Python 3; cada una se documenta en su cabecera):

| Herramienta | Para qué |
|---|---|
| `make_msxdisk.py` | Imágenes de MSX-DOS desde una carpeta (con los ficheros de sistema de un disco de arranque); listar, añadir y extraer ficheros de imágenes existentes |
| `make_amsdisk.py` | Imágenes de AMSDOS (modo AMSTRAD) desde una carpeta, repartidas en varios discos manteniendo juntos los ficheros de cada programa |
| `make_cdt.py` | Cintas de prueba (CDT/TZX) |
| `make_rom.py` | `ALESTE.ROM` a partir de la ROM del 520EX (`AL512.BIN`) |
| `msxrom2aleste.py` | Convierte cartuchos de MSX al 520EX con la "GameBios" del Aleste (ver abajo) |
| `raw2edsk.py`, `bdos_2drives.py`, `dos_preload.py` | Conversión de imágenes de disco, parche de la BDOS para dos disqueteras, ayudas para la simulación |
| `gen_rtl.py`, `gen_u765.py` | Generan el RTL parcheado a partir de Aleste LX y del controlador u765 |
| `check_quartus.py`, `aleste_wbs.py`, `gen_font.py` | Comprobación de compatibilidad con Quartus, guiones de bus, fuente de la ROM de diagnóstico |
| `diag/` | ROM de diagnóstico (ensamblador Z80, `pasmo`) |
| `gb2/` | Cargador genérico para juegos de MSX2 con la GameBios v2 (ensamblador Z80) y bancos de prueba en un emulador de Z80 |

`msx/TIME.COM` (con su fuente) pone en hora el reloj desde MSX-DOS.

### Cartuchos de MSX en el 520EX

La comunidad del Aleste convertía juegos de MSX con la **GameBios**, una
página 0 de MSX de sustitución con un emulador del VDP por software que iba
dentro del cargador de cada juego. `msxrom2aleste.py` automatiza
conversiones del mismo modo: el motor de 16 KB de Konami de 1985 (King's
Valley, Yie Ar Kung-Fu) con la primera GameBios, y **Metal Gear** (MSX2) con
la segunda y un cargador propio. La GameBios se toma del software del Aleste
que aportes tú (disco `msx-games`); este repositorio no incluye ningún juego
ni ninguna GameBios. El análisis y el desensamblado de la GameBios están en
`doc/`.

## Cómo funciona

El core reutiliza Aleste LX como submódulo de git: `tools/gen_rtl.py` genera
a partir de él los ficheros de `rtl/`, con los cambios necesarios para estas
placas y para el 520EX (cada cambio está descrito en el script). El hardware
propio del 520EX (vídeo del CPC, teclado, reloj, MagicSound, ratones, cinta)
está en `rtl/cpc/`. La memoria está en la SDRAM de la placa; en la Calypso
(8 MB) el mapa de memoria se recoloca.

## Compilar

```
git clone --recursive https://github.com/shaeon/aleste520ex.git
```

Abre el proyecto de tu placa en Quartus Prime 22.1 Lite y compila; el `.rbf`
sale en `output_files/`. Los bancos de prueba de Verilator están en `sim/`
(`make`, `make fase2`...). La documentación de `doc/` está en español;
[doc/00-guia-de-uso.md](doc/00-guia-de-uso.md) es la guía de uso.

## Créditos

Este core se apoya en el trabajo de mucha gente. Gracias a todos.

**Código usado en el core**

- **Aleste LX** (la base del core: Z80, MMU, vídeo, árbitro de memoria y
  más): hww (github.com/hww/aleste-lx-fpga-sv). Licencia Apache 2.0. Se usa
  como submódulo; los ficheros generados a partir de él indican su origen.
- **Módulos de MiST** (`user_io`, `data_io`, vídeo, OSD, doblador de
  líneas): Till Harbaum, Gyorgy Szombathelyi (gyurco) y la comunidad de MiST.
  GPL v3 o posterior.
- **u765**, controlador de disquete uPD765, y **tzxplayer**: Gyorgy
  Szombathelyi (gyurco), de sus cores de Amstrad y Spectrum para MiST. GPL.
- **CRTC UM6845R** y los módulos de ratón **Kempston, SYMBiFACE II, Multiplay
  y AMX**: Sorgelig, adaptados a MiST por gyurco. GPL.
- DAC sigma-delta (SiDi): basado en la nota de aplicación XAPP154 de Xilinx.

**Referencias**

- **El archivo del Aleste 520EX de hww** (github.com/hww/Aleste520EX): ROM,
  esquemas, manuales, software y los cargadores con la GameBios.
- Las páginas de **CPCWiki** sobre el Aleste 520EX y el driver `al520ex` de
  **MAME**.
- El desensamblado comentado de **Metal Gear** de Manuel Pazos
  (github.com/GuillianSeed/MetalGear).
- Las herramientas de disco de fMSX `rddsk`/`wrdsk` (Arnold Metselaar) y
  `cpcxfs`, como referencia para las herramientas de disco; la base de datos
  de software de openMSX, para identificar ROM.
- Los pines de las placas, tomados del core del NewBrain
  (github.com/shaeon/newbrain).

**Herramientas**

- Los bancos de prueba y los análisis de la GameBios usan el paquete de
  Python `z80` de Ivan Kosarev (licencia MIT, no incluido). Las fuentes de
  Z80 se ensamblan con `pasmo`.

## Agradecimientos

A **hww**, por Aleste LX y por conservar y publicar el material del Aleste
520EX, y a quienes diseñaron y programaron el Aleste 520EX.

Al **Retro-Wiki FPGA-dev Team**, en especial a **Ron, Manuel (teiram), Somhi,
Roderick, Rampa, Kyp y Benito**, por su ayuda y su paciencia. Una mención
especial para **Ron**.

A **Jaime González (alt)**, posiblemente el único poseedor de un Aleste
520EX en España, por su ayuda incondicional.

## Asistencia de IA y recursos

Partes de este core se desarrollaron con la ayuda de Claude (Anthropic), usado
para código, bancos de prueba, cosimulación y documentación. El acceso a
Claude lo proporcionó Advanced Computer Trading, S.L. (actsl.com), que
autoriza además la publicación de este trabajo con la licencia indicada
abajo. Todas las decisiones de diseño, la integración, las pruebas en
hardware real y la revisión final las hizo el autor, Shaeon (Carlos Palmero).

## Licencia

Copyright (C) 2026 Shaeon (Carlos Palmero).

Este programa es software libre: puedes redistribuirlo y/o modificarlo según
los términos de la **Licencia Pública General de GNU** publicada por la Free
Software Foundation, ya sea la **versión 3** de la Licencia o (a tu elección)
cualquier versión posterior. Ver [LICENSE](LICENSE).

Hace falta la versión 3 porque algunos de los módulos incluidos (los de
MiST) tienen licencia GPL v3 o posterior. Los ficheros de terceros conservan
sus avisos de copyright y sus licencias, compatibles con la GPL v3 (Aleste
LX, Licencia Apache 2.0).

La ROM del 520EX que va en `sd/ALESTE.ROM` procede del material publicado
por hww y se incluye por comodidad; pertenece a sus respectivos dueños.

## Exención de responsabilidad

Este proyecto se entrega **"tal cual", sin garantía de ningún tipo**, ni
expresa ni implícita, incluidas, entre otras, las garantías de
comerciabilidad y de idoneidad para un fin concreto. En ningún caso los
autores o colaboradores serán responsables de ninguna reclamación, daño u otra
responsabilidad derivada del uso de este software, del hardware en el que se
ejecuta o de cualquier equipo conectado. Úsalo bajo tu propia responsabilidad.

Aleste, Amstrad, MSX y los nombres de los juegos mencionados son marcas de sus
respectivos propietarios. Este proyecto no está afiliado a ellos ni cuenta con
su respaldo.
