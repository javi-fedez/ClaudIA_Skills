# Skill: Producción de Vídeo con Remotion

ClaudIA puede diseñar, escribir y renderizar vídeos programáticos usando Remotion — vídeos generados con código React/TypeScript. El MCP `remotion` da acceso a búsqueda semántica en toda la documentación oficial.

---

## Herramienta MCP disponible

| Herramienta | MCP | Función |
|---|---|---|
| `remotion-documentation` | `remotion` | Búsqueda semántica en la documentación oficial de Remotion |

**Uso del MCP:** Antes de escribir cualquier código Remotion, consultar la documentación para obtener la API exacta, evitar deprecaciones y encontrar ejemplos reales. Llamar con una query descriptiva en inglés.

---

## ¿Qué es Remotion?

Framework para crear vídeos con React. Cada frame es un componente React renderizado en un instante de tiempo concreto (`useCurrentFrame()`). El resultado se exporta como MP4, GIF, WebM o secuencia de imágenes.

**Casos de uso:**
- Vídeos de presentación de empresa / producto
- Tutoriales animados con texto y gráficos
- Vídeos de datos (charts animados, estadísticas)
- Intros/outros para YouTube o redes
- Vídeos personalizados en masa (mail merge en vídeo)

---

## Flujo de trabajo completo

### 1. Consultar documentación antes de codificar
```
[Usar MCP remotion-documentation]
Query: "how to use interpolate for fade animations"
Query: "spring animation bouncing"
Query: "AbsoluteFill component usage"
Query: "Audio component sync with video"
```

### 2. Estructura de un proyecto Remotion
```bash
# Crear proyecto nuevo
npx create-video@latest
cd mi-video
npm install

# Estructura resultante:
# src/
#   Root.tsx          ← registro de composiciones
#   MyVideo.tsx       ← componente principal
#   index.ts          ← entry point
# remotion.config.ts  ← configuración
# package.json
```

### 3. Componente básico de vídeo
```tsx
// src/MyVideo.tsx
import { AbsoluteFill, useCurrentFrame, interpolate, spring, useVideoConfig } from 'remotion';

export const MyVideo = () => {
  const frame = useCurrentFrame();
  const { fps, durationInFrames } = useVideoConfig();

  // Fade in durante los primeros 30 frames
  const opacity = interpolate(frame, [0, 30], [0, 1], {
    extrapolateLeft: 'clamp',
    extrapolateRight: 'clamp',
  });

  // Animación spring para escala
  const scale = spring({ frame, fps, config: { damping: 10 } });

  return (
    <AbsoluteFill style={{ backgroundColor: '#0f172a', justifyContent: 'center', alignItems: 'center' }}>
      <h1 style={{ color: 'white', fontSize: 80, opacity, transform: `scale(${scale})` }}>
        Hola Mundo
      </h1>
    </AbsoluteFill>
  );
};
```

### 4. Registrar la composición en Root.tsx
```tsx
// src/Root.tsx
import { Composition } from 'remotion';
import { MyVideo } from './MyVideo';

export const RemotionRoot = () => (
  <>
    <Composition
      id="MyVideo"
      component={MyVideo}
      durationInFrames={150}   // 5 segundos a 30fps
      fps={30}
      width={1920}
      height={1080}
    />
  </>
);
```

### 5. Previsualizar en el Studio
```bash
npx remotion studio
# Abre http://localhost:3000 con preview interactivo
```

### 6. Renderizar a MP4
```bash
npx remotion render MyVideo output/video.mp4
# Con opciones:
npx remotion render MyVideo output/video.mp4 --codec=h264 --crf=18
```

---

## APIs clave (consultar MCP para detalles actualizados)

| API | Uso |
|---|---|
| `useCurrentFrame()` | Frame actual (0 → durationInFrames-1) |
| `useVideoConfig()` | `fps`, `width`, `height`, `durationInFrames` |
| `interpolate(frame, [in], [out])` | Mapear frames a valores (opacidad, posición, escala) |
| `spring({ frame, fps, config })` | Animación física tipo muelle |
| `sequence` / `<Sequence>` | Desfasar componentes en el tiempo |
| `<Series>` | Reproducir escenas en serie automáticamente |
| `<Audio src={} />` | Añadir audio sincronizado |
| `<Video src={} />` | Incrustar vídeo dentro del vídeo |
| `<Img src={} />` | Imagen estática optimizada |
| `<OffthreadVideo>` | Vídeo en render offthread (más rápido) |
| `getInputProps()` | Pasar props al renderizar (para personalización en masa) |
| `continueRender` / `delayRender` | Esperar recursos async antes de renderizar |

---

## Patrones de consulta al MCP

Usar queries específicas para obtener resultados útiles:

```
"interpolate easing functions list"
"how to render with lambda serverless"
"Lottie animation in Remotion"
"Three.js 3D in Remotion"
"dynamic duration based on content"
"loop video seamlessly"
"gif output render command"
"@remotion/google-fonts usage"
"subtitles captions transcription"
"remotion data visualization charts"
"input props schema validation"
"remotion player embed in React app"
```

---

## Render en masa (personalización por registro)

```bash
# Pasar datos dinámicos al render
npx remotion render MyVideo output/video-javi.mp4 \
  --props='{"nombre": "Javi", "empresa": "AdoptaUnia"}'
```

```tsx
// En el componente
import { getInputProps } from 'remotion';
const { nombre, empresa } = getInputProps();
```

---

## Render serverless con Lambda (para escala)

```bash
npm install @remotion/lambda
npx remotion lambda functions deploy
npx remotion lambda render <function-url> MyVideo
```
> Consultar MCP: `"remotion lambda deploy setup aws"`

---

## Ejemplo de prompt para ClaudIA

```
Crea un vídeo de 10 segundos en Remotion para presentar AdoptaUnia.
Debe mostrar:
1. Logo (fade in) durante 2s
2. Texto "Adopta un perro, cambia una vida" con animación spring (3s)
3. Grid de 4 fotos de perros apareciendo progresivamente (4s)
4. Call to action "adoptaunia.com" con fondo azul (1s)
Resolución 1920x1080, 30fps, exportar a MP4.
Antes de codificar, consulta el MCP de Remotion para la API de Sequence y spring.
```
