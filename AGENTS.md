# 🛡️ ARMY MAKER — Reglas de Agente y Protocolo de Documento Vivo (`AGENTS.md`)

## 🚨 1. PROTOCOLO OBLIGATORIO DE DOCUMENTO VIVO (LEER PRIMERO)

1. **Punto de Partida en Cada Conversación:**
   - El archivo [`PROJECT_MANUAL.md`](./PROJECT_MANUAL.md) es el **Documento Maestro Vivo** de este proyecto.
   - **Al iniciar cualquier conversación o antes de modificar código**, debes leer [`PROJECT_MANUAL.md`](./PROJECT_MANUAL.md) para tener el contexto técnico completo (arquitectura, flujo de datos, esquema de base de datos, estado de las 7 facciones y limitaciones conocidas).
2. **Registro Obligatorio de Cambios (Living Document):**
   - **Ningún cambio en el código está completo si no se actualiza [`PROJECT_MANUAL.md`](./PROJECT_MANUAL.md).**
   - Siempre que implementes una nueva funcionalidad, corrección de bug, refactorización o cambio de datos:
     1. Actualiza la sección técnica correspondiente de [`PROJECT_MANUAL.md`](./PROJECT_MANUAL.md) (Secciones 1 a 7) si cambió algún archivo, flujo, regla o estado.
     2. Añade una nueva entrada fechada al principio de la **Sección 8 (`## 8. Registro Vivo de Cambios (Changelog)`)** de [`PROJECT_MANUAL.md`](./PROJECT_MANUAL.md) detallando los archivos tocados y el cambio técnico realizado.
     3. Ejecuta `npm test` (`node scripts/verify_easyarmy.js`) si tocaste facciones o el motor, y sube los cambios a GitHub (`git add`, `git commit`, `git push`) cuando corresponda.

---

## ⚡ 2. Resumen Técnico Rápido (Mapa Mental Inmediato)

- **Proyecto:** Constructor de listas de ejército para **Bolt Action: Tercera Edición (v3)** con paridad 100% respecto a EasyArmy (`wg.easyarmy.com`).
- **Filosofía:** **Zero-Build Runtime SPA & PWA Standalone**. Se ejecuta directamente desde `index.html` usando **React 18 UMD**, **Babel Standalone**, **Tailwind CSS Play CDN**, **`html2pdf.js`** y **Supabase JS v2**.
- **Servidor Local:** `node server.js` (`npm run dev` o `start.bat`), escucha en los puertos `5173` y `3000` (sin requerir `vite`) y arranca automáticamente el demonio `scripts/auto_sync.js` (que hace auto-commit y push a GitHub tras 8s de inactividad).

### Mapa de Archivos Críticos
- [`index.html`](./index.html): Shell principal, paleta Tailwind militar (`zinc-50..950`, `amber-50..600`, `tacticalRed: '#733028'`), registro de Service Worker y orden de carga de scripts.
- [`sw.js`](./sw.js): Service Worker (`CACHE = 'army-maker-v16'`). Estrategia *Network-First* para `index.html`, `config.js`, `/js/` y `/css/`; *Stale-While-Revalidate* para `factions/*` e imágenes.
- [`css/styles.css`](./css/styles.css): Safe-areas PWA, scrollbars y reglas `@media print` A4 retrato (`@page { size: A4 portrait; margin: 8mm 8mm 10mm 8mm; }`, cromos `.unit-card-cromo` de `90mm × 50mm`).
- [`js/engine.js`](./js/engine.js): Motor puro JS (compatible con navegador y Node.js vía `_root`). Contiene `getUnitDefinition`, `getUnitBaseMen`, `calculateUnitPoints`, `getUnitSummary`, `findUnitWeaponRow`, `buildUnitRosterTable` y `validateUnitConfiguration`.
- [`js/storage.js`](./js/storage.js): Cliente Supabase (`window.supabaseClient`), listas locales (`boltaction.saved_lists.v1`) y hook `useAuth()`.
- [`js/icons.js`](./js/icons.js): Iconos SVG inline, `<HelmetLogo />`, `<UnitTacticalIcon />` y `<PlatoonTacticalIcon />` (símbolos OTAN).
- [`js/app.js`](./js/app.js): Componente raíz `<App />` (estado de facción, pelotones, auto-limpieza de pelotones vacíos, *Drag-to-Copy* sobre pestañas y exportación PDF con `html2pdf.js`).
- [`js/components/Slot.js`](./js/components/Slot.js): Casilla de pelotón, buscador de unidades con filtro de obligatoriedad y *Drag-to-Copy* (HTML5 + pulsación larga táctil de `220ms`).
- [`js/components/UnitEditModal.js`](./js/components/UnitEditModal.js): Modal a dos columnas para configurar unidad, opciones, tabla de armas, ilustración y bloqueo por errores de validación.
- [`js/components/Modals.js`](./js/components/Modals.js): `<ModalShell />`, `<AuthModal />`, `<SavedListRow />`, `<MyListsModal />` y `<ExportModal />`.
- [`js/components/ArmyRosterPrintView.js`](./js/components/ArmyRosterPrintView.js): Vista de impresión A4, glosario contextual de reglas activas y **Tarjetas de Unidad (`<UnitTacticalCard />` de `90mm × 50mm`)** con descuento automático de cargador LMG (`Cargador LMG`, 0 disparos).
- [`factions/`](./factions/): 7 facciones activas registradas en `window.EASYARMY_FACTIONS` (`germany`, `soviet_union`, `usa`, `great_britain`, `japan`, `italy`, `france` — 821 unidades y 2.089 opciones).
- [`supabase/schema.sql`](./supabase/schema.sql): Tabla `public.army_lists` (con RLS por `auth.uid() = user_id`, límite de 512 KB en `data` y trigger de máximo 50 listas por usuario).

### Reglas Inquebrantables de Código
1. **Sin `import`/`export` en `js/` ni `factions/`**: Todos los símbolos compartidos entre archivos del frontend deben exponerse en `window` (o `_root` en `js/engine.js`).
2. **Evitar colisiones de ámbito en Babel**: Al desestructurar hooks de React al inicio de un archivo JSX, usar `var { useState, useEffect, useRef, useMemo } = React;` (nunca `const`).
3. **Nomenclatura interna en inglés**: No traducir nombres de unidades, armas u opciones en los datasets, ya que `buildUnitRosterTable` y `validateUnitConfiguration` operan mediante expresiones regulares sobre la nomenclatura oficial de EasyArmy.
4. **Cromos de impresión A4**: Respetar estrictamente el tamaño físico `90mm × 50mm` de `.unit-card-cromo` y la caja de imagen `26mm × 26mm` con fondo blanco.
