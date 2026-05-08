# ANERSA Admin Panel — Especificación Técnica

> **Este documento es la fuente de verdad del proyecto.**  
> Si eres un agente de IA leyendo esto, aquí encontrarás todo lo que necesitas saber para entender, mantener y extender el proyecto sin ambigüedad.

---

## 1. Visión General del Proyecto

### 1.1 Propósito

`anersa-invitaciones-padmin` es un **panel de administración Angular base, reutilizable**, construido con el framework de UI **Nebular**. Su diseño permite que cualquier agente de IA (o desarrollador) pueda conectarle módulos de negocio sin tocar la infraestructura base.

La filosofía es: **base sólida una vez, módulos de negocio encima siempre**.

### 1.2 Casos de Uso Objetivo

- Panel de administración para el sistema de invitaciones de ANERSA
- Plantilla base para cualquier sistema de administración interno que requiera ANERSA
- El código base (sin módulos de negocio) debe ser completamente funcional: login, navegación, tema persistente, estructura modular lista para escalar

### 1.3 Stack Tecnológico — Versiones Exactas

| Tecnología | Versión | Propósito |
|---|---|---|
| Angular | `15.2.10` | Framework principal |
| TypeScript | `4.9.x` | Lenguaje |
| RxJS | `7.8.x` | Programación reactiva |
| Nebular Theme | `11.0.1` | Componentes UI, temas, layout |
| Nebular Auth | `11.0.1` | Módulo de autenticación (base, no se usa la estrategia de Nebular directamente) |
| Nebular Security | `11.0.1` | Control de acceso basado en roles (ACL) |
| Nebular Eva Icons | `11.0.1` | Iconografía (`nb-icon`) |
| Eva Icons | `1.1.3` | Fuente de íconos SVG usada por Nebular |
| Bootstrap | `4.3.1` | Solo el **grid system** (no estilos de Bootstrap, solo SCSS mixins y grid) |
| Ionicons | `2.0.1` | Íconos legacy (usados en footer) |
| Pace JS | `1.x` | Barra de progreso de carga de página |
| @angular/cdk | `15.x` | Angular Component Dev Kit (requerido por Nebular) |

### 1.4 Comandos Esenciales

```bash
# Instalar dependencias
npm install

# Servidor de desarrollo (http://localhost:4200)
ng serve

# Build de producción
ng build --configuration production

# Generar componente (ejemplo)
ng generate component pages/mi-modulo/mi-componente
```

---

## 2. Arquitectura de Módulos

### 2.1 Diagrama de Módulos

```
AppModule (root)
├── CoreModule.forRoot()        → Servicios singleton (auth, layout, mock data)
├── ThemeModule.forRoot()       → Componentes UI compartidos + 4 temas Nebular
├── NbSidebarModule.forRoot()   → Sidebar singleton de Nebular
├── NbMenuModule.forRoot()      → Menú singleton de Nebular
├── NbEvaIconsModule            → Íconos Eva
└── AppRoutingModule
    ├── /auth  (lazy)  → AuthModule
    │   └── /auth/login → LoginComponent
    ├── /pages (lazy)  → PagesModule [AuthGuard]
    │   ├── /pages/dashboard → DashboardModule (lazy)
    │   └── [aquí se agregan módulos de negocio futuros]
    └── /        → redirect /pages
    └── **       → redirect /pages
```

### 2.2 Descripción de Cada Módulo

#### `AppModule` — Módulo Raíz
- **Archivo:** `src/app/app.module.ts`
- **Propósito:** Punto de entrada. Importa módulos globales singleton y configura Nebular.
- **No declara componentes de página** — solo `AppComponent`.
- `AppComponent` es el bootstrap component: restaura el tema guardado en `localStorage` en `ngOnInit`.

#### `CoreModule` — Capa de Servicios Singleton
- **Archivo:** `src/app/@core/core.module.ts`
- **Patrón:** `forRoot()` con `throwIfAlreadyLoaded()` — garantiza instancia única.
- **Provee:**
  - `AuthService` — autenticación mock (reemplazable por JWT/OAuth2)
  - `MockDataModule` — módulo de datos mock para desarrollo
  - `LayoutService` — eventos de cambio de tamaño de layout
- **NUNCA importar en módulos lazy** — solo en `AppModule`.

#### `ThemeModule` — Módulo de UI Compartido
- **Archivo:** `src/app/@theme/theme.module.ts`
- **Patrón:** `forRoot()` para configurar temas Nebular; importación normal para usar componentes.
- **Declara:** `HeaderComponent`, `FooterComponent`, `OneColumnLayoutComponent`
- **Re-exporta:** Todos los módulos de Nebular necesarios para que cualquier módulo de página los use sin re-importar.
- **Contiene:** Configuración de los 4 temas y todos los estilos SCSS.

#### `AuthModule` — Módulo de Autenticación
- **Archivo:** `src/app/auth/auth.module.ts`
- **Cargado lazy** desde `AppRoutingModule`.
- **Contiene:** `LoginComponent` únicamente.
- **Sin layout** — la pantalla de login no usa `OneColumnLayout`, tiene su propio diseño standalone.

#### `PagesModule` — Shell del Panel de Administración
- **Archivo:** `src/app/pages/pages.module.ts`
- **Cargado lazy** desde `AppRoutingModule`, **protegido por `AuthGuard`**.
- **Contiene:** `PagesComponent` (shell con sidebar + layout), `DashboardModule` y futuros módulos de negocio.
- `PagesComponent` inyecta el menú desde `MENU_ITEMS` y lo pasa a `<nb-menu>`.

#### `DashboardModule` — Página Principal
- **Archivo:** `src/app/pages/dashboard/dashboard.module.ts`
- **Cargado lazy** desde `PagesRoutingModule`.
- **Contiene:** `DashboardComponent` con tarjetas de estadísticas estáticas.
- **Es la plantilla de referencia para crear nuevos módulos de negocio.**

---

## 3. Estructura de Archivos Completa

```
ANERSE-invitaciones/
├── TECHNICAL_SPEC.md               ← Este archivo
├── angular.json                    ← Configuración Angular CLI
├── package.json                    ← Dependencias npm
├── tsconfig.json                   ← TypeScript config raíz
├── tsconfig.app.json               ← TypeScript config de la app
└── src/
    ├── index.html                  ← HTML raíz, selector <anersa-app>
    ├── main.ts                     ← Bootstrap de AppModule
    ├── polyfills.ts                ← Polyfills (zone.js mínimo)
    ├── environments/
    │   ├── environment.ts          ← Dev: production: false
    │   └── environment.prod.ts     ← Prod: production: true
    └── app/
        ├── app.module.ts           ← Módulo raíz
        ├── app.component.ts        ← Bootstrap component (restaura tema)
        ├── app-routing.module.ts   ← Rutas raíz (lazy loads)
        │
        ├── @theme/                 ← Módulo UI (layout, componentes, estilos)
        │   ├── theme.module.ts
        │   ├── components/
        │   │   ├── header/
        │   │   │   ├── header.component.ts
        │   │   │   ├── header.component.html
        │   │   │   └── header.component.scss
        │   │   └── footer/
        │   │       └── footer.component.ts
        │   ├── layouts/
        │   │   └── one-column/
        │   │       ├── one-column.layout.ts
        │   │       └── one-column.layout.scss
        │   └── styles/
        │       ├── styles.scss         ← Entry point de estilos globales
        │       ├── themes.scss         ← Registro de los 4 temas
        │       ├── _layout.scss        ← Overrides de layout responsive
        │       ├── _overrides.scss     ← Overrides de componentes Nebular
        │       ├── pace.theme.scss     ← Estilos de la barra de progreso
        │       ├── theme.default.ts    ← Variables tema Light
        │       ├── theme.dark.ts       ← Variables tema Dark
        │       ├── theme.cosmic.ts     ← Variables tema Cosmic
        │       └── theme.corporate.ts  ← Variables tema Corporate
        │
        ├── @core/                  ← Módulo de servicios singleton
        │   ├── core.module.ts
        │   ├── module-import-guard.ts
        │   ├── services/
        │   │   └── auth.service.ts     ← Mock auth (login/logout/isAuthenticated)
        │   ├── guards/
        │   │   └── auth.guard.ts       ← CanActivate: redirige a /auth/login si no autenticado
        │   ├── utils/
        │   │   └── layout.service.ts   ← Observable de cambios de tamaño del layout
        │   ├── data/
        │   │   └── users.ts            ← Abstract class UserData
        │   └── mock/
        │       ├── users.service.ts    ← Mock implementation de UserData
        │       └── mock-data.module.ts ← Módulo que provee los mocks
        │
        ├── auth/                   ← Módulo de autenticación (lazy)
        │   ├── auth.module.ts
        │   ├── auth-routing.module.ts
        │   └── login/
        │       ├── login.component.ts
        │       ├── login.component.html
        │       └── login.component.scss
        │
        └── pages/                  ← Shell del panel (lazy, protegido por AuthGuard)
            ├── pages.module.ts
            ├── pages-routing.module.ts
            ├── pages.component.ts
            ├── pages.component.scss
            ├── pages-menu.ts           ← AQUÍ se configuran los ítems del menú lateral
            └── dashboard/
                ├── dashboard.module.ts
                ├── dashboard.component.ts
                ├── dashboard.component.html
                └── dashboard.component.scss
```

---

## 4. Sistema de Autenticación

### 4.1 Implementación Actual (Mock)

La autenticación es completamente **mock** — no realiza ninguna llamada HTTP. Está diseñada para ser reemplazada por una implementación real sin modificar nada fuera de `auth.service.ts`.

**Archivo:** `src/app/@core/services/auth.service.ts`

**Credenciales de prueba:**
| Campo | Valor |
|---|---|
| Email | `admin@anersa.com` |
| Contraseña | `admin123` |
| Nombre de usuario | `Admin ANERSA` |

**API del servicio:**

```typescript
class AuthService {
  login(email: string, password: string): Observable<{ success: boolean; error?: string }>
  logout(): void
  isAuthenticated(): boolean
  getUsername(): string
  getToken(): string | null
}
```

**Almacenamiento en localStorage:**

| Key | Valor | Descripción |
|---|---|---|
| `anersa-auth-token` | string (mock token) | Token de sesión. Su existencia determina si el usuario está autenticado. |
| `anersa-user` | string (nombre de usuario) | Nombre visible en el header. |

### 4.2 Flujo de Autenticación

```
Login page → AuthService.login(email, pass)
  ├── Éxito → guardar en localStorage → router.navigate(['/pages/dashboard'])
  └── Error → mostrar mensaje "Credenciales incorrectas"

AuthGuard → AuthService.isAuthenticated()
  ├── true  → dejar pasar la ruta /pages/**
  └── false → router.navigate(['/auth/login'])

Header Logout → AuthService.logout()
  └── limpiar localStorage → router.navigate(['/auth/login'])
```

### 4.3 Cómo Reemplazar Mock Auth por Auth Real (JWT)

1. Instalar: `npm install @nebular/auth` (ya instalado)
2. En `auth.service.ts`, reemplazar la validación estática por una llamada HTTP:
   ```typescript
   login(email: string, password: string): Observable<...> {
     return this.http.post<AuthResponse>('/api/auth/login', { email, password })
       .pipe(tap(response => {
         localStorage.setItem('anersa-auth-token', response.token);
         localStorage.setItem('anersa-user', response.username);
       }));
   }
   ```
3. El `AuthGuard`, `HeaderComponent` y `LoginComponent` **no requieren cambios**.

---

## 5. Sistema de Temas

### 5.1 Temas Disponibles

| Nombre | Display | Base Nebular |
|---|---|---|
| `default` | Light | `default` |
| `dark` | Dark | `dark` |
| `cosmic` | Cosmic | `cosmic` |
| `corporate` | Corporate | `corporate` |

**Tema predeterminado:** `default` (Light)

### 5.2 Persistencia

El tema seleccionado se persiste en `localStorage`:

| Key | Valores posibles | Default |
|---|---|---|
| `anersa-theme` | `default`, `dark`, `cosmic`, `corporate` | `default` |

**Restauración:** `AppComponent.ngOnInit()` lee `anersa-theme` de `localStorage` y llama `NbThemeService.changeTheme()` antes del primer render.

**Cambio:** `HeaderComponent` → `<nb-select>` → `changeTheme(themeName)` → `NbThemeService.changeTheme(themeName)` → guarda en `localStorage`.

### 5.3 Estructura de Archivos de Tema

- `@theme/styles/themes.scss` — Registra los 4 temas con `nb-register-theme()` (variables custom de layout/cards)
- `@theme/styles/theme.*.ts` — Archivos TypeScript con variables de colores para uso en componentes que necesitan valores dinámicos en runtime (estos no son SCSS, son objetos JS importados por componentes Angular)

### 5.4 Cómo Agregar un Nuevo Tema

1. En `@theme/styles/themes.scss`, agregar un nuevo bloque `nb-register-theme((...), nombre, base-theme)`.
2. En `AppModule`, agregar el nombre al array `themes` de `NbThemeModule.forRoot({ name: ..., themes: [...] })`.
3. En `HeaderComponent`, agregar el objeto `{ value: 'nombre', name: 'Display Name' }` al array `themes`.

---

## 6. Sistema de Menú Lateral

### 6.1 Archivo de Configuración

**Archivo:** `src/app/pages/pages-menu.ts`

Este es el **único archivo que se debe modificar** para agregar, quitar o reorganizar ítems del menú lateral.

### 6.2 Estructura de un Ítem de Menú

```typescript
import { NbMenuItem } from '@nebular/theme';

export const MENU_ITEMS: NbMenuItem[] = [
  {
    title: 'Dashboard',           // Texto visible en el menú
    icon: 'home-outline',         // Nombre del ícono de Eva Icons
    link: '/pages/dashboard',     // Ruta Angular (routerLink)
    home: true,                   // true = página de inicio (resaltada por defecto)
  },
  {
    title: 'SECCIÓN',             // Separador de grupo (solo texto, sin link)
    group: true,
  },
  {
    title: 'Mi Módulo',
    icon: 'briefcase-outline',
    children: [                   // Submenú desplegable
      {
        title: 'Listado',
        link: '/pages/mi-modulo/lista',
      },
      {
        title: 'Crear',
        link: '/pages/mi-modulo/crear',
      },
    ],
  },
];
```

### 6.3 Íconos Disponibles

Usar íconos de [Eva Icons](https://akveo.github.io/eva-icons/) — siempre con el sufijo `-outline` o `-fill`.  
Ejemplos: `home-outline`, `people-outline`, `settings-outline`, `briefcase-outline`, `clipboard-outline`, `list-outline`.

---

## 7. Layout y Componentes de UI

### 7.1 Layout Principal

El panel de administración usa `OneColumnLayout`:

```html
<nb-layout windowMode>
  <nb-layout-header fixed>
    <anersa-header></anersa-header>      <!-- Logo + hamburguesa + theme switcher + user/logout -->
  </nb-layout-header>

  <nb-sidebar class="menu-sidebar" tag="menu-sidebar" responsive>
    <nb-menu [items]="menu"></nb-menu>   <!-- Menú dinámico desde pages-menu.ts -->
  </nb-sidebar>

  <nb-layout-column>
    <router-outlet></router-outlet>      <!-- Aquí se renderizan las páginas -->
  </nb-layout-column>

  <nb-layout-footer fixed>
    <anersa-footer></anersa-footer>      <!-- Footer simple -->
  </nb-layout-footer>
</nb-layout>
```

### 7.2 Header Component

**Selector:** `anersa-header` | **Archivo:** `@theme/components/header/header.component.ts`

**Responsabilidades:**
- Toggle del sidebar (`NbSidebarService.toggle(true, 'menu-sidebar')`)
- Mostrar el logo "ANERSA"
- Theme Switcher (`<nb-select>` → `NbThemeService.changeTheme()` + `localStorage`)
- Mostrar nombre del usuario logueado (`AuthService.getUsername()`)
- Botón de Logout (`AuthService.logout()`)

**Subscripciones (RxJS):**
- `NbThemeService.onThemeChange()` — para sincronizar el selector con el tema activo
- `NbMediaBreakpointsService` — para comportamiento responsive

### 7.3 Componentes Nebular Usados

| Componente Nebular | Tag HTML | Uso |
|---|---|---|
| `NbLayoutModule` | `<nb-layout>`, `<nb-layout-header>`, etc. | Estructura del layout |
| `NbSidebarModule` | `<nb-sidebar>` | Barra lateral |
| `NbMenuModule` | `<nb-menu>` | Menú de navegación |
| `NbCardModule` | `<nb-card>`, `<nb-card-header>`, `<nb-card-body>` | Tarjetas de contenido |
| `NbButtonModule` | `<button nbButton>` | Botones |
| `NbInputModule` | `<input nbInput>` | Campos de formulario |
| `NbSelectModule` | `<nb-select>`, `<nb-option>` | Selector desplegable |
| `NbActionsModule` | `<nb-actions>`, `<nb-action>` | Botones de acción en header |
| `NbUserModule` | `<nb-user>` | Avatar + nombre de usuario |
| `NbContextMenuModule` | `[nbContextMenu]` | Menú contextual (user dropdown) |
| `NbIconModule` | `<nb-icon>` | Renderizar íconos SVG |
| `NbCheckboxModule` | `<nb-checkbox>` | Checkboxes |
| `NbAlertModule` | `<nb-alert>` | Mensajes de alerta |
| `NbSpinnerModule` | `[nbSpinner]` | Indicador de carga |

---

## 8. Patrones de Código — Convenciones Obligatorias

### 8.1 Nomenclatura de Archivos

| Tipo | Patrón | Ejemplo |
|---|---|---|
| Componente | `nombre.component.ts` | `login.component.ts` |
| Módulo | `nombre.module.ts` | `dashboard.module.ts` |
| Servicio | `nombre.service.ts` | `auth.service.ts` |
| Guard | `nombre.guard.ts` | `auth.guard.ts` |
| Constante/Config | `nombre-items.ts` | `pages-menu.ts` |
| Layout | `nombre.layout.ts` | `one-column.layout.ts` |
| Interfaz/Abstracta | `nombre.ts` en `/data/` | `users.ts` |

### 8.2 Patrón de Limpieza de Subscripciones RxJS

**SIEMPRE usar este patrón en componentes con subscripciones:**

```typescript
import { Subject } from 'rxjs';
import { takeUntil } from 'rxjs/operators';

export class MiComponent implements OnInit, OnDestroy {
  private destroy$ = new Subject<void>();

  ngOnInit() {
    miObservable.pipe(takeUntil(this.destroy$)).subscribe(data => { ... });
  }

  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

### 8.3 Patrón de Datos: Abstract Class + Mock Service

Para cualquier fuente de datos en el panel:

```typescript
// 1. Interfaz/abstracta en @core/data/
export abstract class MiData {
  abstract getItems(): Observable<MiItem[]>;
}

// 2. Mock en @core/mock/
export class MiService extends MiData {
  getItems(): Observable<MiItem[]> {
    return of(MOCK_ITEMS);  // datos estáticos
  }
}

// 3. Provider en CoreModule / MockDataModule
{ provide: MiData, useClass: MiService }

// 4. Inyección en componente
constructor(private miData: MiData) {}
```

### 8.4 Módulos: forRoot() para Singletons

```typescript
@NgModule({ ... })
export class CoreModule {
  constructor(@Optional() @SkipSelf() parentModule: CoreModule) {
    throwIfAlreadyLoaded(parentModule, 'CoreModule');
  }
  static forRoot(): ModuleWithProviders<CoreModule> {
    return { ngModule: CoreModule, providers: [...] };
  }
}
```

---

## 9. Cómo Agregar un Nuevo Módulo de Negocio

Esta es la guía para que un agente de IA o desarrollador agregue funcionalidad sin romper la base.

### 9.1 Pasos

**1. Crear el módulo con lazy loading:**

```
src/app/pages/mi-modulo/
├── mi-modulo.module.ts
├── mi-modulo-routing.module.ts
├── lista/
│   ├── lista.component.ts
│   ├── lista.component.html
│   └── lista.component.scss
└── detalle/
    ├── detalle.component.ts
    ├── detalle.component.html
    └── detalle.component.scss
```

**2. Registrar la ruta en `PagesRoutingModule`:**

```typescript
// src/app/pages/pages-routing.module.ts
{
  path: 'mi-modulo',
  loadChildren: () => import('./mi-modulo/mi-modulo.module').then(m => m.MiModuloModule),
}
```

**3. Agregar al menú en `pages-menu.ts`:**

```typescript
{
  title: 'Mi Módulo',
  icon: 'briefcase-outline',
  link: '/pages/mi-modulo/lista',
}
```

**4. Importar `ThemeModule` en el nuevo módulo para usar componentes Nebular:**

```typescript
@NgModule({
  imports: [CommonModule, ThemeModule, MiModuloRoutingModule],
  declarations: [ListaComponent, DetalleComponent],
})
export class MiModuloModule {}
```

### 9.2 Checklist para Nuevo Módulo

- [ ] Crear carpeta en `src/app/pages/mi-modulo/`
- [ ] Crear `mi-modulo.module.ts` con `ThemeModule` importado
- [ ] Crear `mi-modulo-routing.module.ts`
- [ ] Crear componentes necesarios
- [ ] Agregar ruta lazy en `pages-routing.module.ts`
- [ ] Agregar ítem en `pages-menu.ts`
- [ ] Si necesita datos: crear abstract class en `@core/data/` + mock en `@core/mock/` + provider en `CoreModule`

---

## 10. Variables de Entorno

**`src/environments/environment.ts`** (desarrollo):
```typescript
export const environment = {
  production: false,
  apiUrl: 'http://localhost:3000/api',  // URL del backend cuando se conecte uno real
};
```

**`src/environments/environment.prod.ts`** (producción):
```typescript
export const environment = {
  production: true,
  apiUrl: 'https://api.anersa.com',    // URL del backend de producción
};
```

---

## 11. Estado Actual del Proyecto y Roadmap

### 11.1 Incluido en la Base (v1.0)

| Feature | Estado | Notas |
|---|---|---|
| Login con mock auth (email + password) | ✅ | Credenciales: admin@anersa.com / admin123 |
| AuthGuard para rutas protegidas | ✅ | Redirige a /auth/login |
| Layout OneColumn (header + sidebar + footer) | ✅ | |
| Sidebar dinámico via pages-menu.ts | ✅ | |
| Theme Switcher (4 temas) | ✅ | |
| Persistencia de tema en localStorage | ✅ | Key: anersa-theme |
| Nombre de usuario en header | ✅ | Leído de localStorage |
| Botón Logout en header | ✅ | |
| Dashboard con tarjetas estáticas | ✅ | Datos de ejemplo de invitaciones |

### 11.2 Excluido de la Base (se agrega como módulos)

| Feature | Por qué excluido |
|---|---|
| Gráficos (ECharts, Chart.js, ngx-charts) | Depende del módulo de negocio |
| Tablas CRUD (ng2-smart-table) | Depende del módulo de negocio |
| Mapas (Leaflet, Google Maps) | Depende del módulo de negocio |
| Editores enriquecidos (TinyMCE, CKEditor) | Depende del módulo de negocio |
| Auth real (JWT/OAuth2) | Reemplazar auth.service.ts cuando haya backend |
| Tests unitarios | Agregar cuando se implemente lógica de negocio |
| Layouts Two/Three columns | OneColumn es suficiente para panel admin estándar |
| Internacionalización (i18n) | Agregar si se requiere multi-idioma |

---

## 12. Decisiones de Diseño y Justificaciones

| Decisión | Alternativa considerada | Razón |
|---|---|---|
| Mock auth en lugar de Nebular NbDummyAuthStrategy | Usar NbDummyAuthStrategy del DEMO | Se quiere control total sobre el flujo sin dependencia del sistema auth de Nebular que puede cambiar en v12+ |
| Solo layout OneColumn | Dos/Tres columnas como en DEMO | El panel admin estándar no necesita múltiples columnas de layout |
| `Subject + takeUntil` para cleanup | `Subscription.unsubscribe()` | Patrón más limpio y consistente con el DEMO de referencia |
| `localStorage` para tema y sesión | `sessionStorage` o cookies | Persistencia entre cierres de pestaña sin complejidad de cookies |
| Bootstrap solo grid | Bootstrap completo | Nebular provee todos los estilos UI; Bootstrap solo se usa para el grid responsive system |

---

*Última actualización: Mayo 2026 — Fase 0 y 1 completadas*
