# Informe de Auditoría: duck-studio-suite

- **Fecha:** 2026-08-27
- **Stack detectado:** React 19 + TypeScript + Vite 7 + Tailwind CSS 4 + Radix UI/shadcn + Express 4 + pnpm
- **Commits analizados:** 1 (ab8fc75) — proyecto generado en un solo commit "docs: add README"
- **Veredicto:** mejorable

## Lo mejor del repo (mínimo 3)

1. **Build y typecheck limpios.** `pnpm check` (tsc --noEmit, strict) y `pnpm build` (1618 módulos) terminan sin errores; el lockfile de pnpm está fielmente sincronizado (`--frozen-lockfile` instaló sin conflictos).
2. **Diseño con intención real.** ideas.md define un lenguaje visual coherente ("Laboratorio de señal nocturna") que Home.tsx aplica de forma consistente: 5 vistas de sesión, secuenciador de 16 pasos, mute/solo/fader, medidores y osciloscopio en vivo, todo con propósitos accesibles (`aria-label`, `role`, teclas de acceso).
3. **Código cliente muy modular y limpio.** Componentes Radix/shadcn bien encapsulados, hooks reutilizables con buena higiene (`usePersistFn`, `useComposition` con manejo del IME de Safari, `useIsMobile`), ThemeProvider con toggling opt-in y ErrorBoundary con recuperación.
4. **Higiene de secretos impecable.** Escaneo con patrones de claves (OpenAI/OpenRouter/Anthropic/GitLab/GitHub PAT/AWS/GCP/private keys), passwords y `api_key/token/secret` hardcodeados: 0 matches. No hay `.env`, `.pem` ni `.key` versionados, y `.gitignore` cubre el runtime data habitual.

## Hallazgos CRÍTICOS

Ninguno. No se detectaron secretos reales ni vectores RCE/inyección SQL (no hay SQL ni evaluación de input de usuario; `firewall` del servidor es sólo una fallback SPA sobre archivos estáticos).

## Hallazgos ALTOS

1. **Script de analítica roto en producción (FIJADO).** `client/index.html:14` usaba `%VITE_ANALYTICS_ENDPOINT%/umami` y `%VITE_ANALYTICS_WEBSITE_ID%`, placeholders no definidos que Vite emitía literales en el HTML final → `<script src=".../umami">` roto y 2 warnings por cada build. Se añadió un guard en `vite.config.ts` (`vitePluginAnalyticsGuard`) que **elimina el tag si las variables no están configuradas** y lo sustituye por valores reales si sí lo están. Verificado: el HTML de `dist/public` ya no contiene `umami` ni `%VITE_ANALYTICS_%`.
2. **Assets `/manus-storage/*` irresolubles al auto-alojar (NO TOCADO).** El favicon (`client/index.html:8`), el fondo reticulado (`client/src/index.css:16`) y 3 `<img>` de Home.tsx (líneas 235, 248, 264) apuntan a `/manus-storage/…`, que sólo resuelve en desarrollo a través del proxy de la plataforma (`vitePluginStorageProxy`) o en el hosting de Manus. En `pnpm start` (servidor Express plano) dan 404. Requiere assets reales locales o deploy en la infraestructura Manus; no eliminado por regla de auditoría.

## Hallazgos MEDIOS

1. **Herramientas Manus incrustadas (dev-only, NO TOCADO).** `vite.config.ts` incluye `vite-plugin-manus-runtime`, `vitePluginManusDebugCollector` y `vitePluginStorageProxy`. El collector inyecta `client/public/__manus__/debug-collector.js` (intercepta console, fetch/XHR y eventos de UI y los reenvía a `/__manus__/logs`; sanitiza password/token/secret) **sólo en desarrollo**. El archivo se copia igualmente a `dist/public/__manus__/` (no se carga en producción). Útil para el agente que generó el proyecto, no para la app. Recomendación: aislar de la config de producción.
2. **Código muerto de plantilla (NO TOCADO).** `components/ManusDialog.tsx` (login "Login with Manus"), `components/Map.tsx` (Google Maps consumiendo `VITE_FRONTEND_FORGE_API_KEY`) y `const.ts` (`getLoginUrl`, `COOKIE_NAME`) no se importan desde ninguna ruta del router. Dead code heredado de la plantilla.
3. **README insuficiente (MEJORADO).** Tenía 10 líneas sin instalar/ejecutar/probar. Se expandió con características, stack, requisitos, scripts, variables de entorno y estructura.
4. **`.gitignore` sin `.manus-logs/` (FIJADO).** El collector escribe logs de runtime en `.manus-logs/`; el directorio no estaba ignorado y podía terminarse commiteando. Añadido.
5. **Sin CI ni tests.** No hay `.github/workflows/` y `vitest` está instalado pero no hay test files (“FASE 3” sugiere añadir CI mínimo y tests de humo).
6. **Alias muerto `@assets`** en `vite.config.ts` apunta a `attached_assets/` inexistente.
7. **devDependency `add`** (paquete sin utilidad real) — candidato a remover con `pnpm remove add`.

## Añadido por el auditor

Rama `agent/auditoria-2026-08-27` sobre `main` con 4 commits: 3 de la auditoría + este informe.

- `bfe3d73` fix: drop unconfigured umami placeholder tag from production HTML
- `0b35c59` chore: ignore Manus debug collector logs
- `2c5cbd8` docs: document setup, scripts and env vars in README
- (commit del informe AUDIT.md)

## Próximos pasos recomendados

1. Decidir el destino del tooling Manus: mover debug-collector/plugins a dev-only explícito o eliminarlos si la app deja de desarrollarse en la plataforma.
2. Sustituir los assets `/manus-storage/*` por locales (favicon, grid de fondo, imágenes de Home) para auto-hosting correcto.
3. Añadir CI mínimo (`.github/workflows/ci.yml` con `pnpm install --frozen-lockfile`, `pnpm check`, `pnpm build`) y tests de humo (el scaffold ya trae vitest).
4. Limpiar dead code: `Map.tsx`, `ManusDialog.tsx`, `const.ts`, alias `@assets`, dep `add`.
5. Si se usa OAuth/umami, documentar o popular `.env.example` con los nombres de las variables.

## No tocado (pero anotado)

- `server/index.ts` — sirve estáticos + SPA fallback; correcto. El fallback `app.get("*")` sólo falla si `index.html` falta.
- `client/public/__manus__/debug-collector.js` — no eliminado (regla de auditoría); se recomienda excluirlo de `dist` en el build de producción.
- `client/src/components/Map.tsx`, `ManusDialog.tsx`, `client/src/const.ts` — dead code, no eliminados.
- `pnpm-lock.yaml`, `client/index.css`, componentes UI, `Home.tsx` — sin cambios.