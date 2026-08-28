# Duck Studio Suite

Suite integrada de producción musical web del universo DUCK. Prototipo interactivo que organiza una idea rítmica como una sesión tangible: secuenciador de 16 pasos, piano roll, playlist, mezclador, laboratorio vocal y telemetría de sesión. Las interacciones son locales y demostrativas (no se procesa ni exporta audio real).

## Características

- **Secuenciador de pasos** con 16 pasos por patrón, mute/solo y fader por pista.
- **Piano roll**, **Playlist** y **Mixer** como vistas de la misma sesión.
- **Laboratorio vocal** con armado de toma y lista de capturas.
- Transporte con BPM ajustable, metrónomo visual y osciloscopio de demostración.
- Atajos de teclado: `Espacio` reproducir/pausar, `R` armar captura vocal, `1–5` cambiar vista.

## Stack

- **Frontend:** React 19 + TypeScript + Vite 7
- **UI:** Tailwind CSS 4 y componentes Radix UI / shadcn
- **Servidor:** Express 4 (sirve el build estático y SPA fallback)
- **Gestor de paquetes:** pnpm (10.4.1, ver `packageManager`)

## Requisitos

- Node.js ≥ 20
- pnpm ≥ 10

## Instalación

```bash
pnpm install --frozen-lockfile
```

## Scripts

| Comando            | Descripción                                                  |
| ------------------ | ------------------------------------------------------------ |
| `pnpm dev`         | Arranca el dev server de Vite en `http://localhost:3000`     |
| `pnpm build`       | Genera el cliente en `dist/public` y el servidor en `dist/`  |
| `pnpm start`       | Sirve el build de producción (`NODE_ENV=production`)         |
| `pnpm preview`     | Previsualiza el build con Vite                               |
| `pnpm check`       | Typecheck con `tsc --noEmit`                                 |
| `pnpm format`      | Formatea el código con Prettier                              |

## Variables de entorno

Las variables se cargan desde la raíz del proyecto (`envDir`). Todas son opcionales:

| Variable                     | Descripción                                                   |
| ---------------------------- | ------------------------------------------------------------- |
| `VITE_ANALYTICS_ENDPOINT`    | Base URL de umami analytics. Si no se define, no se emite el tag. |
| `VITE_ANALYTICS_WEBSITE_ID`  | ID del sitio en umami.                                        |
| `VITE_APP_ID` / `VITE_OAUTH_PORTAL_URL` | Login OAuth (usado por `client/src/const.ts`).        |
| `VITE_FRONTEND_FORGE_API_KEY` / `VITE_FRONTEND_FORGE_API_URL` | Google Maps (componente `Map.tsx`).          |

## Estructura

```
client/src/     Aplicación React (componentes, páginas, hooks, estilos)
shared/         Constantes compartidas entre cliente y servidor
server/         Servidor Express para servir el build
dist/           Build de producción (cliente en dist/public, servidor en dist/)
```

## Estado

Proyecto del ecosistema DUCK (ver duck-ecosystem, duck-2026, duck-lab). En fase de prototipo; el audio real y la exportación están fuera de alcance en esta versión estática.