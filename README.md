# Video Player HLS - FlutterFlow Custom Widget

Widget custom para **FlutterFlow** que reproduce video **HLS/m3u8** en vivo usando el paquete [Chewie](https://pub.dev/packages/chewie).

## Características

- Reproducción de streams HLS/m3u8 en vivo
- Autoplay al iniciar
- Controles custom:
  - Play / Pause (centro)
  - Mute / Unmute (esquina inferior derecha)
  - Fullscreen (esquina inferior derecha)
  - Badge **LIVE** (esquina inferior izquierda)
- Pantalla de carga con fondo negro y texto "Cargando..."
- Wakelock: mantiene la pantalla encendida durante la reproducción
- Detección de visibilidad: pausa el video cuando no es visible
- Soporte de orientación landscape en pantalla completa

## Dependencias (pub.dev)

Agregar en **FlutterFlow > Settings > Project Dependencies**:

| Paquete | Versión |
|---------|---------|
| chewie | ^1.8.5 |
| video_player | ^2.9.2 |
| visibility_detector | ^0.4.0+2 |
| wakelock_plus | ^1.2.8 |

## Parámetros del Widget

| Parámetro | Tipo | Default | Descripción |
|-----------|------|---------|-------------|
| videoPath | String | URL de ejemplo | URL del stream HLS/m3u8 |
| autoPlay | bool | true | Iniciar reproducción automáticamente |
| looping | bool | true | Repetir el video |
| allowFullScreen | bool | true | Permitir pantalla completa |

## Cómo usar en FlutterFlow

1. Ir a **Custom Code > Custom Widgets**
2. Crear un nuevo widget
3. Copiar el contenido de `chewie_demo.dart`
4. Agregar las dependencias en **Settings > Project Dependencies**
5. Agregar los parámetros del widget (videoPath, autoPlay, looping, allowFullScreen)
6. Compilar el APK

## Captura

El widget muestra controles tipo streaming en vivo con badge LIVE rojo, botón play/pause central, y controles de mute y fullscreen en la barra inferior.
