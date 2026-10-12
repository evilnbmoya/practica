# SPEC — Sistema de Cine

Guía de implementación del TP N.º 1 de Programación IV (UTN FRA, comisión 141).

| Dato | Valor |
|---|---|
| Versión | 2 · 06/10/2026 |
| Nombre del cine | Cineverso |
| Entrega | 13/10/2026 (funciones nuevas hasta el 11/10; el 12/10 es para probar la app desplegada y cerrar el README) |
| Qué construir | *Sistema de Cine — Documento de Requerimientos* (revisado por el docente) |
| Cómo construirlo | Clases 1 a 8 de la cursada (referencia definitiva) y el proyecto de clase `taskflow` (`progra IV/taskflow`) |
| Stack | Angular 22 · Supabase (Auth, Postgres, RLS, Realtime, Storage) · PWA · Vercel |

Regla de precedencia: el documento de requerimientos manda sobre el **qué**; las clases (y después `taskflow`), sobre el **cómo**; la convención de comentarios de la sección 3.1, sobre ambos.

Cada decisión lleva su línea **Por qué:**. Los identificadores RF, RN, RNF y S son los del documento de requerimientos. Los D-xx son decisiones de este SPEC ante puntos que el documento no resuelve (ver sección 13).

---

## 1. Resumen y alcance

### 1.1 Qué es

Una PWA instalable para un cine de un solo edificio con varias salas idénticas. Vende entradas, candy y combos con o sin cuenta, emite un único QR por compra, valida ese QR en la puerta y en el candy, y le da al administrador el catálogo, la programación, los precios, los reportes y la auditoría.

Las reglas críticas (salas, butacas, QR y dinero) viven en Postgres: restricciones, funciones RPC transaccionales y triggers. Angular solo las anticipa para dar buena experiencia (sección 5 del documento de requerimientos, RNF-11).

### 1.2 Roles

El rol vive en `perfiles.rol` y se controla en dos capas: guards funcionales en Angular y RLS en Supabase (sección 3 del documento, RNF-06, RNF-10).

| Rol | Cómo existe en el sistema | Puede |
|---|---|---|
| Visitante | Sin sesión (rol `anon` de Postgres). Al elegir butacas se le crea una **sesión anónima** de Supabase (rol `authenticated` con `is_anonymous = true`) | Ver cartelera, fichas y reseñas. Comprar pagando, sin beneficios (S-04) |
| Cliente | Cuenta propia (email y contraseña, Google o GitHub) con `rol = 'cliente'` | Lo anterior, más cupones, puntos, canjes, crédito, cancelación, reseñas, alertas y Mis películas |
| Empleado | Cuenta creada por un admin con `rol = 'empleado'` (RF-06) | Validar QR o código manual en sala y candy |
| Administrador | Cuenta creada por otro admin; el primero se crea con SQL (sección 7.1) | Catálogo, salas, funciones, precios, promociones, reportes, auditoría y cuentas internas |

Empleado y administrador **no compran** (tabla de la sección 3 del documento). La base lo impide en `retener_butacas` y `confirmar_compra`.

### 1.3 Módulos dentro del alcance

Usuarios y cuenta (RF-01 a RF-07, RF-86) · Películas y cartelera (RF-08 a RF-15) · Salas, butacas y funciones (RF-16 a RF-25) · Mapa en tiempo real (RF-26 a RF-31, RF-90) · Compra (RF-32 a RF-38) · Precios y cupones (RF-39 a RF-45) · Candy y combos (RF-46 a RF-50) · Comprobante, QR y validación (RF-51 a RF-60) · Experiencia del cliente (RF-61 a RF-76, RF-87 a RF-89) · Backoffice, reportes y auditoría (RF-77 a RF-85).

### 1.4 Fuera de alcance

Del documento (sección 8): mapa del edificio con la sala de la entrada, cobro real con pasarela, devolución de dinero, envío de emails (comprobantes y alertas) y venta de candy sin entrada o en un punto de venta presencial.

Técnicos, decididos en este SPEC:

- Vincular la sesión anónima con una cuenta nueva. Las compras anónimas quedan en el comprobante del dispositivo (S-04).
- CAPTCHA para el inicio de sesión anónimo. Alcanza con el límite por IP que trae Supabase.
- SMTP propio. Se usa el de Supabase y se desactiva la confirmación de email (D-16). La recuperación de contraseña (RF-02) se implementa completa, pero ese SMTP solo envía a las direcciones del equipo del proyecto y tiene un límite bajo por hora. En la demo se prueba con una dirección del equipo, y la limitación queda escrita en el README.
- La CLI de Supabase y Supabase local: la base se arma pegando scripts en el SQL Editor (sección 7.6).
- Edge Functions: solo `enviar-push`, que envía el push (D-25). Las cuentas internas se siguen creando sin ellas (D-23).
- Push con la app cerrada: la cátedra lo exigió el 09/10, así que F16 deja de ser opcional (D-25).

---

## 2. Convenciones heredadas del proyecto de clase

La referencia definitiva del "cómo" son las **clases 1 a 8**. Donde difieren del código de `taskflow`, mandan las clases. Criterio acordado: en Angular, todo como en clase; la base queda como está diseñada. La tabla de decisiones técnica por técnica está en la sección 13.4.

### 2.1 Qué se copia y dónde se ve

| # | Convención | Archivo de `taskflow` que la muestra |
|---|---|---|
| C-01 | Componentes standalone y arranque con `bootstrapApplication(App, appConfig)` | `src/main.ts` |
| C-02 | `appConfig` con `provideBrowserGlobalErrorListeners()` y `provideRouter(routes, withComponentInputBinding())`: los parámetros de ruta llegan como inputs | `src/app/app.config.ts`; uso en `tasks/task-detail/task-detail.ts:20` (`taskId = input.required({ transform: numberAttribute })`) |
| C-03 | Una carpeta por componente con `nombre.ts`, `nombre.html`, `nombre.css` y `nombre.spec.ts`; clase sin sufijo `Component`; selector `app-` en kebab-case; `imports`, `templateUrl` y `styleUrl` en el decorador | `tasks/task-card/` |
| C-04 | Signal inputs y outputs: `input.required<T>()`, `input(valor)`, `transform: numberAttribute`, `alias`, `output<T>()` y `emit` desde un método del componente | `tasks/task-card/task-card.ts:14-30` |
| C-05 | Control de formulario propio con `model()`, usado con `[formField]` | `tasks/priority-picker/priority-picker.ts:10`, `tasks/task-form/task-form.html:35` |
| C-06 | Store singleton con signals públicos de solo lectura, métodos `async` que llaman a la Api y después hacen `update` del signal, y componentes que derivan con `computed` | `services/task-store.service.ts`, `tasks/task-board/task-board.ts:22`, `tasks/task-list/task-list.ts:18` |
| C-07 | Servicio de Api delgado, sin estado: `const { data, error } = await supabase…; if (error) throw error; return data as T` | `tasks/task-api.service.ts` |
| C-08 | Cliente de Supabase como constante de módulo creada con `createClient` | `services/supabase.client.ts` |
| C-09 | Modelos con `interface`, tipos `Draft = Omit<…, 'id'>` y `Patch = Partial<…>`, factory del borrador vacío, uniones de literales, etiquetas en `Record<Union, string>` y `switch` exhaustivo con `never` | `tasks/task.model.ts`, `tasks/task-status.ts:25` |
| C-10 | Sesión como `signal<Session \| null>` y una promesa `initialized` que se resuelve con el primer evento de auth | `services/auth.service.ts:12-29` |
| C-11 | Guards funcionales `async` que esperan la inicialización y devuelven `router.parseUrl(...)` para redirigir | `guard/auth-guard.ts` |
| C-12 | Signal Forms: `form(signal, schema, { submission: { action } })`, `[formRoot]`, `[formField]`, errores con `@for (e of campo().errors(); track e.kind)` cuando `touched() && !valid()`, signals `saving` y `submitError`, y `role="alert"` en el error de envío | `tasks/task-form/task-form.ts:53-100`, `task-form.html` |
| C-13 | Control de flujo integrado: `@if (…; as x)`, `@for` con `track` y `@empty`, `@let` | `task-board.html`, `task-detail.html`, `task-form.html` |
| C-14 | Realtime: `supabase.channel().on('postgres_changes', …).subscribe()`, un `switch` sobre `eventType` y `supabase.removeChannel()` para limpiar | `services/task-store.service.ts:127-196` |
| C-15 | Storage: ruta por entidad, nombre de archivo normalizado y `upload(..., { upsert: true })` | `services/attachments.ts:11-30` |
| C-16 | CSS puro por componente con `:host`, sin framework ni preprocesador | `*.css` |
| C-17 | Vitest con globals (`vi.fn()`), `TestBed.configureTestingModule({ imports: [Componente] })`, mocks con `{ provide: X, useValue: {...} }` y `fixture.componentRef.setInput(...)` | `app.spec.ts`, `task-card.spec.ts`, `task-detail.spec.ts` |
| C-18 | Formato: Prettier con `printWidth: 100` y `singleQuote`; EditorConfig con 2 espacios | `.prettierrc`, `.editorconfig` |
| C-19 | `async/await` con `try/finally` para apagar el estado de guardado | `task-form.ts:62-97` |
| C-20 | Interfaz en español | todas las plantillas |

### 2.2 Diferencias con el proyecto de clase

| En `taskflow` | Dónde | En este proyecto | Motivo |
|---|---|---|---|
| La propiedad de la URL se llama `supebaseUrl` | `src/environments/environments.ts:3` | `supabaseUrl` | Coincide con el nombre del servicio |
| Hay dos carpetas de environments; la de `src/app/environments/` no se importa | `src/environments/`, `src/app/environments/` | Una sola carpeta, `src/environments/` | Sin duplicados |
| `angular.json` no tiene `fileReplacements` (lo indica el README de clase, línea 95) | `angular.json` | `environment.ts` para desarrollo y `environment.prod.ts` para producción, con `fileReplacements` en la configuración de producción y el campo `supabaseAnonKey`, como Clase 6, lámina 20 | RNF-24 pide configuración por entorno |
| Los adjuntos del usuario se leen con `getPublicUrl` | `services/attachments.ts:27` | Los buckets públicos guardan solo pósters e imágenes de productos, y escribe solo el admin. Los comprobantes se generan en el cliente | RNF-13; Storage no guarda datos personales |
| `AuthService` inyecta `TaskStore` y le informa el usuario | `services/auth.service.ts:8,31` | Cada store lee `SesionStore.usuarioId()` | La sesión no depende del dominio |
| Un `<a>` contiene una tarjeta que tiene botones | `tasks/task-list/task-list.html:10` | El enlace va dentro de la tarjeta, separado de los botones | El HTML no admite elementos interactivos anidados (RNF-20) |
| Las acciones de la tarjeta llaman al store sin `await` | `tasks/task-board/task-board.ts:37,41` | `await` con `try/catch` y el error en un signal | El usuario ve si la acción falló |
| `UserModel.id` es `number` | `tasks/task.model.ts:14` | `string` para los ids de usuario | Supabase usa uuid |
| Algunas specs crean el componente con el store real | `task-board.spec.ts`, `task-list.spec.ts` | Mocks con `useValue`, como en `task-detail.spec.ts` | Las pruebas no dependen de la red |
| Algunos archivos usan 4 espacios y comillas dobles | `auth.service.ts`, `supabase.client.ts`, `task-api.service.ts` | 2 espacios y comillas simples | Es lo que configuran `.editorconfig` y `.prettierrc` |
| Cada componente define sus colores en hex | `*.css` | Tokens en `styles.css`; los componentes usan `var(--…)` | La identidad se cambia en un solo lugar (RNF-17) |
| `errorMessage` está dentro del formulario | `task-form.ts:42` | `shared/utils/errores.ts` con `mensajeError(error)` y los códigos de la base | Lo usan todos los formularios |
| Los comentarios tienen formatos distintos | varios | La convención de la sección 3.1 | Un solo formato en todo el código |
| — | `tsconfig.json` de `tp1-sistema-cine` | Se agregan `strict` y `strictTemplates` en la fase 0 | RNF-08 |

### 2.3 Lo que no tiene ejemplo en el proyecto de clase

Verificado buscando en el código de `taskflow`: no hay `.rpc(`, `@switch`, `@Pipe`, `@Directive`, `loadComponent` ni `loadChildren`, service worker, manifest, `signInWithPassword`, `signUp` ni `signInAnonymously`, roles, `functions.invoke`, validadores propios, archivos `.sql` ni pg_cron. La única coincidencia de "role" es `role="alert"`.

| Técnica | Solución propuesta, en el estilo del resto | Sección |
|---|---|---|
| Roles | Columna `perfiles.rol` y funciones SQL `es_admin()` y `es_staff()` en las políticas; en Angular, `adminGuard`, `empleadoGuard` y `clienteGuard`, como el `adminGuard` de Clase 7, lámina 7 | 5.2, 5.11, 8.6 |
| Carga diferida | `loadComponent` con `canMatch` en cada ruta de `/cuenta`, `/empleado` y `/admin`, y en el checkout (Clase 3, lámina 6; Clase 7, lámina 7) | 4 |
| RPC | `supabase.rpc('nombre', params)` dentro de los servicios de Api, con el mismo `if (error) throw error` de C-07 | 6, 8.3 |
| Scripts SQL | Dos scripts en `supabase/sql/` (`01_base.sql` y `tp1-sistema-cine.sql`), pegados en el SQL Editor como en Clase 6, lámina 5 | 5.12, 7.6 |
| PWA y avisos | `ng add @angular/pwa` y `SwUpdate` (Clase 8), más un aviso del sistema con la app abierta | 9 |
| Pipes, directivas y validador | `@Pipe` puro, `@Directive` con `host` y signal inputs, y una regla de Signal Forms construida sobre `validate()` | 8.7 a 8.9 |
| `@switch` | Resultado de validación, estado de butaca y paso del checkout | 8.5 |
| Email, contraseña y sesión anónima | Métodos de `AuthApi` sobre `supabase.auth` | 7.1 |
| Cuentas internas | Segundo cliente de Supabase sin sesión y la RPC `asignar_rol` | 7.4 |
| Tareas programadas | pg_cron | 6.4 |
| Estado del checkout | `CompraStore` y `ButacasStore` en `root` con `reiniciar()`, que el `Checkout` llama en `ngOnDestroy` | 8.4 |
| Servicio de pago | Un `PagoService` común que llama a la RPC; cambiar de pasarela es cambiar esa clase | 8.3 |
| Guards de salida y resolver | `pendingChangesGuard` (`canDeactivate`) y `peliculaResolver` (`resolve`), como Clase 8, láminas 6 y 7 | 4, 8.6 |

### 2.4 Nombres

Reglas:

- **Carpetas de organización en inglés**, como en el material de clase (`ng generate directive shared/highlight`, Clase 8, lámina 3). `shared/` tiene `api/`, `stores/`, `services/`, `models/`, `guard/`, `pipes/`, `directives/`, `validators/`, `components/`, `rules/` y `utils/`.
- **Dominio en español:**
  - las áreas: `publico/`, `cliente/`, `empleado/` y `admin/`;
  - los componentes, como `pelicula-detalle/` y `mapa-butacas/`;
  - las clases, las tablas y las columnas (en `snake_case`);
  - los nombres que exige la cátedra: `duracion`, `precioArs`, `edad`, `appHighlightButaca`, `appRestrictEdad` y `butacasContiguas`.
- **Excepción:** el login es `publico/login/` (componente `Login`, ruta `/login`), como en clase.
- **Nombres de archivo:** los que genera el CLI 22.2, verificados con `--dry-run`.
- **Modelos:** usan los mismos nombres de columna que las tablas.

**Por qué los nombres de columna:** Así `data as PeliculaModel` (C-07) sigue siendo cierto sin una capa de mapeo.

| Artefacto | Comando | Archivo | Clase o símbolo | Selector o nombre |
|---|---|---|---|---|
| Componente | `ng g c publico/pelicula-detalle` | `publico/pelicula-detalle/pelicula-detalle.ts` | `PeliculaDetalle` | `app-pelicula-detalle` |
| Store | `ng g s shared/stores/cartelera-store` | `shared/stores/cartelera-store.ts` | `CarteleraStore` | — |
| Api | `ng g s shared/api/pelicula-api` | `shared/api/pelicula-api.ts` | `PeliculaApi` | — |
| Servicio | `ng g s shared/services/pago-service` | `shared/services/pago-service.ts` | `PagoService` | — |
| Modelo | `ng g interface shared/models/pelicula model` | `shared/models/pelicula.model.ts` | `PeliculaModel`, `PeliculaDraft` | — |
| Guard | `ng g guard shared/guard/admin` | `shared/guard/admin-guard.ts` | `adminGuard` | — |
| Resolver | `ng g resolver shared/guard/pelicula` | `shared/guard/pelicula-resolver.ts` | `peliculaResolver` | — |
| Pipe | `ng g pipe shared/pipes/duracion` | `shared/pipes/duracion-pipe.ts` | `DuracionPipe` | `duracion` |
| Directiva | `ng g d shared/directives/highlight-butaca` | `shared/directives/highlight-butaca.ts` | `HighlightButaca` | `[appHighlightButaca]` |
| Validador | — (a mano) | `shared/validators/butacas-contiguas.ts` | `butacasContiguas` | — |
| Regla pura | — (a mano) | `shared/rules/precio.ts` y su `precio.spec.ts` | `precioEntrada`, `calcularResumen` | — |
| Utilidad | — (a mano) | `shared/utils/errores.ts` | `mensajeError` | — |
| Tabla SQL | — | — | `peliculas`, `peliculas_generos` (plural) | — |
| Función SQL | — | — | `confirmar_compra` (verbo en infinitivo, como la acción que pide el usuario) | — |
| Script SQL | — | `supabase/sql/tp1-sistema-cine.sql` | `01_base.sql`, el script único y, si hacen falta cambios, scripts numerados desde `02` (sección 5.12) | — |

Más detalles:

- **Servicios:** el CLI 22.2 genera `@Service()` por defecto. Se usa `@Injectable({ providedIn: 'root' })`, que es lo que muestra el código de clase (Clase 5, lámina 5; Clase 6, láminas 7 y 15) y que la Clase 1, lámina 6, da como válido. Se agrega `--injectable` al generar, o se ajusta el decorador a mano.
- **Specs:** `angular.json` tiene `skipTests: true` para componentes, directivas, pipes, servicios, guards y resolvers, porque las pruebas son solo de funciones puras (sección 10).

---

## 3. Arquitectura y estructura de carpetas

### 3.1 Convención de comentarios

Esta regla manda sobre lo que se ve en el proyecto de clase.

1. **Como máximo una línea** de comentario antes de cada función o bloque. Nunca dos líneas de comentario seguidas.
2. Describe **qué hace** el bloque, en español, con el verbo en **tercera persona del singular**: "sube", "valida", "calcula". No "subimos", "se suben" ni "subir".
3. Empieza en **minúscula** y **no lleva punto final**.
4. Si una función tiene varios pasos en secuencia, cada paso lleva su línea: `// paso 1: ...`, `// paso 2: ...`.
5. Prohibidos: JSDoc y bloques `/** */`, comentarios al final de una línea de código, código comentado y separadores decorativos.
6. Vale igual para **TypeScript** (`//`), **SQL** (`--`), **HTML** (`<!-- -->`) y **CSS** (`/* */`, en una sola línea).
7. Los comentarios que generó el CLI en los archivos de configuración (`tsconfig*.json`, `.vscode/*.json`, `.gitignore`) quedan como están.

Ejemplos correctos:

```ts
// sube el archivo y devuelve la URL pública
async subirPoster(peliculaId: number, archivo: File): Promise<string> {
  const ruta = `${peliculaId}/${nombreSeguro(archivo.name)}`;
  const { error } = await supabase.storage.from('posters').upload(ruta, archivo, { upsert: true });
  if (error) throw error;
  return supabase.storage.from('posters').getPublicUrl(ruta).data.publicUrl;
}

// confirma la compra y guarda el comprobante en el dispositivo
async confirmar(tarjeta: DatosTarjeta): Promise<void> {
  // paso 1: envía la compra y el pago a la base
  const resultado = await this.pago.pagar(this.solicitud(tarjeta));
  // paso 2: guarda el comprobante para verlo sin conexión
  if (resultado.estado === 'aprobado') this.comprobantes.guardar(await this.api.comprobante(resultado.compra_id));
  this.resultado.set(resultado);
}
```


```html
<!-- muestra la leyenda de tipos de butaca siempre visible -->
<ul class="leyenda">...</ul>
```

```css
/* resalta las butacas vip con borde y estrella */
.butaca--vip { border: 2px solid var(--butaca-vip); }
```

Ejemplos incorrectos, y qué tienen de malo:

| Comentario | Falla |
|---|---|
| `// Calcula el total.` | Mayúscula inicial y punto final |
| `// calculamos el total` / `// se calcula el total` / `// calcular el total` | Persona o forma verbal |
| `const max = 10; // máximo de butacas` | Comentario al final de la línea |
| `/** Calcula el total */` | JSDoc |
| `// ===== Precios =====` | Separador decorativo |
| `// const viejo = calcularViejo();` | Código comentado |
| Dos líneas `//` seguidas | Más de una línea |

En `taskflow` hay 13 comentarios en TypeScript y 13 en CSS, más los que generó el CLI. Solo uno cumple la regla: `services/attachments.ts:10`. No se toma ninguno de los demás como modelo.

### 3.2 Capas

```text
Componentes (plantillas con @if/@for/@switch, Signal Forms)
   │  inyectan
   ▼
Stores (signals, computed, effect)   ◄── reglas puras de TypeScript (anticipan las RN)
   │  llaman
   ▼
Servicios de Api (delgados, sin estado)
   │  usan
   ▼
supabase.client.ts ──► Postgres: RLS + restricciones + RPC + triggers + pg_cron (garantizan las RN)
                       Realtime · Storage
```

| Decisión | Por qué |
|---|---|
| Tres capas en Angular: componentes, stores y Api (RNF-07) | Es la división de `taskflow` (C-06, C-07); cada capa tiene una sola responsabilidad y se prueba con mocks |
| Las reglas de negocio están dos veces: en TypeScript puro (`shared/rules/`) para anticipar, y en SQL para garantizar | El documento lo pide (sección 5): la interfaz anticipa y la base asegura aunque dos usuarios operen a la vez. Las funciones puras se prueban con Vitest (RNF-23) |
| Las pantallas del admin sin estado compartido usan su Api directamente, con signals locales | Como `TaskDetail` con `AttachmentsService`; un store por cada ABM sería código sin beneficio |
| Todos los stores con `providedIn: 'root'`, como en clase. `CompraStore` y `ButacasStore` tienen un `reiniciar()` que el `Checkout` llama en `ngOnDestroy` | Es el patrón de Clase 5, lámina 5, y Clase 6, lámina 13. Al salir del checkout se liberan las butacas y se cierra el canal, y la próxima compra arranca vacía |
| App sin zone.js (el valor por defecto de Angular 22, que este proyecto ya trae) | Todo lo que cambia la vista es un signal, así que no hace falta zone.js |

### 3.3 Árbol propuesto

```text
tp1-sistema-cine/
├── README.md
├── ngsw-config.json
├── angular.json
├── package.json
├── tsconfig.json
├── public/
│   ├── favicon.ico
│   ├── manifest.webmanifest
│   └── icons/
└── src/
    ├── index.html
    ├── main.ts
    ├── styles.css
    ├── environments/
    │   ├── environment.ts
    │   └── environment.prod.ts
    └── app/
        ├── app.ts · app.html · app.css
        ├── app.config.ts
        ├── app.routes.ts
        ├── shared/
        │   ├── api/
        │   ├── stores/
        │   ├── services/
        │   │   └── supabase.client.ts
        │   ├── models/
        │   ├── guard/
        │   ├── pipes/
        │   ├── directives/
        │   ├── validators/
        │   ├── components/
        │   ├── rules/
        │   └── utils/
        ├── publico/
        │   ├── inicio/
        │   ├── proximamente/
        │   ├── pelicula-detalle/
        │   ├── login/
        │   ├── registro/
        │   ├── recuperar-clave/
        │   ├── restablecer-clave/
        │   ├── privacidad/
        │   ├── comprobantes/
        │   └── compra/
        │       ├── checkout/
        │       ├── paso-funcion/
        │       ├── paso-butacas/
        │       ├── paso-candy/
        │       ├── paso-pago/
        │       └── paso-comprobante/
        ├── cliente/
        │   ├── perfil/
        │   ├── completar-perfil/
        │   ├── mis-compras/
        │   ├── detalle-compra/
        │   ├── mis-peliculas/
        │   └── notificaciones/
        ├── empleado/
        │   ├── validacion/
        │   ├── escaner-qr/
        │   └── resultado-validacion/
        └── admin/
            ├── peliculas-admin/
            ├── pelicula-form/
            ├── generos-admin/
            ├── salas-admin/
            ├── sala-butacas/
            ├── funciones-admin/
            ├── programar-funciones/
            ├── funcion-form/
            ├── precios-admin/
            ├── categorias-admin/
            ├── productos-admin/
            ├── combos-admin/
            ├── cupones-admin/
            ├── recompensas-admin/
            ├── cuentas-admin/
            ├── reportes/
            └── auditoria/
```

Los scripts siguen el orden de la sección 5.12. Una corrección toma el siguiente número libre.

| Carpeta | Contenido | Por qué |
|---|---|---|
| `shared/` | Todo lo que usa más de un área: cliente de Supabase, modelos, Api, stores, reglas, guards y resolver, pipes, directivas, validador, componentes reutilizables y utilidades | Lo que es de un área no se mezcla con lo que es de todos. El nombre es el de clase (Clase 8, lámina 3) |
| `shared/models/` | Un archivo `*.model.ts` por entidad (RNF-08) | Como `task.model.ts`: el tipo de la fila más las etiquetas |
| `shared/api/` | Un servicio `*-api.ts` por agregado | C-07: acceso a datos separado (RNF-07, S-08) |
| `shared/stores/` | Los cinco stores de la sección 8.4 | RNF-02 pide stores por dominio: cartelera, compra, butacas y sesión |
| `shared/services/` | Servicios que no son acceso a datos: pago, avisos del sistema y comprobantes locales | Separa la infraestructura de las Api |
| `shared/rules/` | Funciones puras con su spec | Son lo único que se prueba con Vitest, sin TestBed (RNF-23) |
| `shared/guard/` | Un archivo por guard y el resolver, con los nombres del CLI | Como `auth-guard.ts` y `pending-changes` de clase |
| `publico/` | Lo que ve cualquiera, checkout incluido | Visitante y cliente compran por el mismo flujo (RF-33) |
| `cliente/`, `empleado/`, `admin/` | Un área por rol. Cada pantalla se carga a demanda con `loadComponent` y se protege con el guard de su rol en `canMatch`. El menú de cada área va en `app.html`, según el rol | RNF-06; Clase 3, lámina 6, y Clase 7, lámina 7 |
| `supabase/sql/` | `01_base.sql` y `tp1-sistema-cine.sql`, que se pegan en el SQL Editor. Es una carpeta local, excluida en `.git/info/exclude` | La base vive en Supabase y los scripts no forman parte de la entrega |

---

## 4. Rutas

Todas las rutas están en un solo `app.routes.ts`, como en clase:

- `Inicio`, `Comprobantes` y `Privacidad` se cargan con `component`. `PeliculaDetalle` y `Proximamente` pasaron a `loadComponent` en F2: con `component`, el inicial llegaba a 514 kB; así quedó en 508 kB, y con los filtros de F3 en 515 kB. Las de acceso (`Login`, `Registro`, `RecuperarClave` y `RestablecerClave`) usan `loadComponent`, para que Signal Forms no entre en la carga inicial: en F1, con `component`, el inicial llegaba a 566 kB, y así quedó en 487 kB.
- Las pantallas de las áreas y el checkout se cargan con `loadComponent`, y cada ruta de área lleva el guard de su rol en `canMatch` (Clase 3, lámina 6; Clase 7, lámina 7).
- No hay `loadChildren` ni archivos `*.routes.ts` por área.
- `tasks/new` va antes de `tasks/:taskId` (Clase 4, lámina 6), y por lo mismo `peliculas/nueva` va antes de `peliculas/:peliculaId`.

**Por qué la carga diferida:** Es por **rendimiento**. Cada usuario descarga el código de las pantallas que usa, y la carga inicial queda chica.

**Por qué `canMatch`:** Se evalúa antes de descargar el chunk, así que quien no tiene el rol no lo baja (Clase 7, lámina 7). El service worker mantiene esto, porque solo precachea lo inicial (sección 9).

**Qué protege los datos:** El guard no es seguridad (Clase 6, lámina 17; Clase 7, lámina 5): ordena la navegación, y el código de Angular llega al navegador. Los datos los protegen el **RLS** y las **RPC** (secciones 5.11 y 6).

**El menú de cada área va en `app.html`**, con `@if` según `sesion.rol()`. El cliente ve "Mi cuenta", el empleado ve "Validar" y el admin ve su menú: Películas, Géneros, Salas, Funciones, Precios, Candy, Cupones, Recompensas, Cuentas, Reportes, Auditoría y también "Validar", porque `empleadoGuard` lo deja pasar (desde F6).

`/comprobantes` se carga con el bundle inicial, junto con lo que necesita para funcionar sin conexión (sección 9).

| Ruta | Componente | Carga | Guards y resolver | Requerimientos |
|---|---|---|---|---|
| `/` | `Inicio` | `component` | — | RF-13, RF-14 |
| `/proximamente` | `Proximamente` | `loadComponent` | — | RF-68, RF-69 |
| `/peliculas/:peliculaId` | `PeliculaDetalle` | `loadComponent` | `resolve: { pelicula: peliculaResolver }` | RF-15, RF-62 |
| `/login` | `Login` | `loadComponent` | `canActivate: [invitadoGuard]` | RF-02 |
| `/registro` | `Registro` | `loadComponent` | `canActivate: [invitadoGuard]`, `canDeactivate: [pendingChangesGuard]` | RF-01 |
| `/recuperar-clave` | `RecuperarClave` | `loadComponent` | `canActivate: [invitadoGuard]` | RF-02 |
| `/restablecer-clave` | `RestablecerClave` | `loadComponent` | — (llega con la sesión de recuperación) | RF-02 |
| `/comprobantes` | `Comprobantes` | `component` | — | RNF-15, S-04 |
| `/privacidad` | `Privacidad` | `component` | — | RNF-22. Google pide la URL de la política de privacidad para pasar el login a producción |
| `/compra/:peliculaId` (query opcional `funcionId`) | `Checkout` | `loadComponent` | `canActivate: [puedeComprarGuard]`, `canDeactivate: [pendingChangesGuard]` | RF-32 a RF-38 |
| `/cuenta` | `Perfil` | `loadComponent` | `canMatch: [clienteGuard]`, `canDeactivate: [pendingChangesGuard]` | RF-04, RF-66, RF-75 |
| `/cuenta/completar-perfil` | `CompletarPerfil` | `loadComponent` | `canMatch: [clienteGuard]`, `canDeactivate: [pendingChangesGuard]` | RF-86 |
| `/cuenta/compras` | `MisCompras` | `loadComponent` | `canMatch: [clienteGuard]` | RF-05 |
| `/cuenta/compras/:compraId` | `DetalleCompra` | `loadComponent` | `canMatch: [clienteGuard]` | RF-05, RF-51, RF-73 |
| `/cuenta/peliculas` | `MisPeliculas` | `loadComponent` | `canMatch: [clienteGuard]` | RF-72 |
| `/cuenta/notificaciones` | `Notificaciones` | `loadComponent` | `canMatch: [clienteGuard]` | RF-69, RF-87 a RF-89 |
| `/empleado` | `Validacion` | `loadComponent` | `canMatch: [empleadoGuard]` | RF-55 a RF-59 |
| `/admin` | — | — | `redirectTo: 'admin/peliculas'`, `pathMatch: 'full'` | RF-77 |
| `/admin/peliculas` | `PeliculasAdmin` | `loadComponent` | `canMatch: [adminGuard]` | RF-08 a RF-11 |
| `/admin/peliculas/nueva` | `PeliculaForm` | `loadComponent` | `canMatch: [adminGuard]`, `canDeactivate: [pendingChangesGuard]` | RF-08 |
| `/admin/peliculas/:peliculaId` | `PeliculaForm` | `loadComponent` | `canMatch: [adminGuard]`, `canDeactivate: [pendingChangesGuard]` | RF-08, RF-70 |
| `/admin/generos` | `GenerosAdmin` | `loadComponent` | `canMatch: [adminGuard]`, `canDeactivate: [pendingChangesGuard]` | RF-09 |
| `/admin/salas` | `SalasAdmin` | `loadComponent` | `canMatch: [adminGuard]`, `canDeactivate: [pendingChangesGuard]` | RF-16 |
| `/admin/salas/:salaId` | `SalaButacas` | `loadComponent` | `canMatch: [adminGuard]`, `canDeactivate: [pendingChangesGuard]` | RF-20 |
| `/admin/funciones` | `FuncionesAdmin` | `loadComponent` | `canMatch: [adminGuard]` | RF-21, RF-25 |
| `/admin/funciones/programar` | `ProgramarFunciones` | `loadComponent` | `canMatch: [adminGuard]`, `canDeactivate: [pendingChangesGuard]` | RF-22 a RF-24, RF-78 |
| `/admin/funciones/:funcionId` | `FuncionForm` | `loadComponent` | `canMatch: [adminGuard]`, `canDeactivate: [pendingChangesGuard]` | RF-21 |
| `/admin/precios` | `PreciosAdmin` | `loadComponent` | `canMatch: [adminGuard]`, `canDeactivate: [pendingChangesGuard]` | RF-39, RF-41 |
| `/admin/categorias` | `CategoriasAdmin` | `loadComponent` | `canMatch: [adminGuard]`, `canDeactivate: [pendingChangesGuard]` | RF-46 |
| `/admin/productos` | `ProductosAdmin` | `loadComponent` | `canMatch: [adminGuard]`, `canDeactivate: [pendingChangesGuard]` | RF-46 |
| `/admin/combos` | `CombosAdmin` | `loadComponent` | `canMatch: [adminGuard]`, `canDeactivate: [pendingChangesGuard]` | RF-48 |
| `/admin/cupones` | `CuponesAdmin` | `loadComponent` | `canMatch: [adminGuard]`, `canDeactivate: [pendingChangesGuard]` | RF-40, RF-42 |
| `/admin/recompensas` | `RecompensasAdmin` | `loadComponent` | `canMatch: [adminGuard]`, `canDeactivate: [pendingChangesGuard]` | RF-65 |
| `/admin/cuentas` | `CuentasAdmin` | `loadComponent` | `canMatch: [adminGuard]`, `canDeactivate: [pendingChangesGuard]` | RF-06 |
| `/admin/reportes` | `Reportes` | `loadComponent` | `canMatch: [adminGuard]` | RF-79 a RF-82 |
| `/admin/auditoria` | `Auditoria` | `loadComponent` | `canMatch: [adminGuard]` | RF-83 a RF-85 |
| `**` | — | — | `redirectTo: ''`, como Clase 8, lámina 8 | — |

Guards y resolver, uno por archivo en `shared/guard/` (sección 8.6):

| Guard | Tipo | Qué hace |
|---|---|---|
| `invitadoGuard` | `CanActivateFn` | Deja pasar sin sesión o con sesión anónima; a un usuario registrado lo manda a `/` |
| `clienteGuard`, `empleadoGuard`, `adminGuard` | `CanMatchFn` | Sin sesión registrada, devuelven `router.parseUrl('/login')`. Con otro rol, `router.parseUrl('/')`. `empleadoGuard` también deja pasar al admin, que puede validar (tabla de roles del documento) |
| `puedeComprarGuard` | `CanActivateFn` | El visitante y el cliente con perfil completo pasan. El cliente con perfil incompleto va a `/cuenta/completar-perfil?volver=…` (RF-86). Empleado y admin van a `/` con un aviso |
| `pendingChangesGuard` | `CanDeactivateFn<ConCambiosPendientes>` | Si `componente.cambiosPendientes()` es verdadero, pregunta con `confirm(componente.mensajeSalida())`, como Clase 8, lámina 6. Lo usan los formularios (que están sucios si el formulario cambió) y el checkout (que está sucio si hay butacas retenidas: "Tenés butacas retenidas. Si salís, se liberan. ¿Salir igual?") |
| `peliculaResolver` | `ResolveFn<PeliculaCartelera \| null>` | Trae la película de la ruta antes de mostrar la ficha. Con `withComponentInputBinding`, llega al componente como `pelicula = input<PeliculaCartelera \| null>()` (Clase 8, lámina 7) |

**Las ABM chicas** (géneros, categorías, productos, combos, cupones y recompensas) tienen lista y formulario en el mismo componente. Para dos o tres campos, una ruta de formulario aparte no suma nada.

Películas sí tiene ruta propia para el formulario, como `tasks/new` y `tasks/:id/edit` en clase, porque es más grande.

---

## 5. Modelo de datos

Convenciones:

- Tablas en `public`, en plural y `snake_case`.
- Ids `bigint generated always as identity`, salvo `compras.id` (uuid aleatorio) y los ids de usuario (uuid de `auth.users`).
- Montos en `numeric(12,2)`.
- Fechas con hora en `timestamptz`. Para mostrarlas y calcular días se usa la hora de Buenos Aires (RNF-21).

### 5.1 Tipos, configuración y precios

**Script `01_base.sql`**, aplicado en F0, con técnicas vistas en clase: `create table` con `check`, `insert` y RLS (Clase 5, lámina 10; Clase 6, lámina 5).

Los valores cerrados se modelan con `text` + `check (... in (...))`, como `status` en la tabla `tasks` de clase. En `01_base.sql`, `precios_formato.formato` ya usa esa forma.

**Decisión cerrada (sección 13.4):** en todos los scripts los valores cerrados van como `text` + `check`. El DDL de esta sección muestra el diseño con tipos como `public.formato` o `public.rol_usuario`. Al escribir cada script, cada uno de esos tipos se convierte en `text` con el `check` de sus valores, por ejemplo `rol text not null default 'cliente' check (rol in ('cliente', 'empleado', 'admin'))`. Las extensiones (`btree_gist`, `pgcrypto` y `pg_cron`) se crean al principio de `tp1-sistema-cine.sql`.

```sql
create table public.configuracion (
  id int primary key default 1 check (id = 1),
  recargo_vip numeric(12,2) not null default 3000 check (recargo_vip >= 0),
  minutos_limpieza int not null default 30 check (minutos_limpieza >= 0),
  minutos_retencion int not null default 10 check (minutos_retencion between 1 and 60),
  max_butacas_compra int not null default 10 check (max_butacas_compra between 1 and 50),
  horas_limite_cancelacion int not null default 2 check (horas_limite_cancelacion >= 0),
  minutos_previos_acceso int not null default 60 check (minutos_previos_acceso >= 0),
  dias_ranking int not null default 30 check (dias_ranking > 0)
);

create table public.precios_formato (
  formato text primary key check (formato in ('2D', '3D', '4D', '5D')),
  precio_base numeric(12,2) not null check (precio_base > 0)
);
```

Datos iniciales, en el mismo script: una fila de `configuracion` y los cuatro precios base con valores de ejemplo, editables desde `/admin/precios`. Las dos tablas tienen RLS con una política de lectura para cualquiera. La escritura del admin está en `tp1-sistema-cine.sql`, que crea los roles.

| Decisión | Requerimientos | Por qué |
|---|---|---|
| Todos los valores numéricos del negocio en `configuracion` | RF-39, S-21, RN-05, RN-17, RN-18, RN-20, introducción de la sección 7 del documento | Cambiar un criterio no requiere tocar código |
| `configuracion` de una sola fila (`check (id = 1)`) | RF-39 | Es el modo más simple de guardar parámetros globales con tipos y restricciones |
| `btree_gist` | RN-02 | Lo necesita la restricción de exclusión que combina `sala_id with =` con un rango |

### 5.2 Usuarios

```sql
create table public.perfiles (
  id uuid primary key references auth.users (id) on delete cascade,
  email text,
  nombre text check (char_length(nombre) between 1 and 60),
  apellido text check (char_length(apellido) between 1 and 60),
  fecha_nacimiento date check (fecha_nacimiento > date '1900-01-01'),
  color_ojos text,
  dias_vacaciones int check (dias_vacaciones between 0 and 365),
  rol public.rol_usuario not null default 'cliente',
  creado_en timestamptz not null default now()
);

create table public.datos_sensibles (
  usuario_id uuid primary key references public.perfiles (id) on delete cascade,
  tipo_sangre text not null check (tipo_sangre in ('0-', '0+', 'A-', 'A+', 'B-', 'B+', 'AB-', 'AB+')),
  consentimiento_en timestamptz not null default now()
);
```

- El trigger `crear_perfil` (sección 6.3) crea la fila de `perfiles` con `rol = 'cliente'` para todo usuario **no anónimo**. Nadie se autorregistra como empleado o admin (RF-06).
- El perfil está **completo** cuando `nombre`, `apellido`, `fecha_nacimiento`, `color_ojos` y `dias_vacaciones` no son nulos. Lo exige `puedeComprarGuard` y lo vuelven a exigir `retener_butacas` y `confirmar_compra` (RF-86, RN-06).
- `color_ojos` sale de una lista cerrada con la opción "otro" (D-17).
- Que exista la fila en `datos_sensibles` significa que hubo consentimiento. Retirar el consentimiento es borrar la fila (S-16, RNF-22).
- **La fecha de nacimiento se carga una sola vez.** El trigger `proteger_fecha_nacimiento` (sección 6.3) deja completarla cuando está vacía: en el registro con email, o en `CompletarPerfil` si la cuenta es de Google o GitHub. Si ya tiene valor, rechaza cualquier cambio que llegue por la API con `FECHA_NACIMIENTO_INMUTABLE`. El perfil la muestra como solo lectura.
- **Corrección de un error de carga:** se hace desde el SQL Editor de Supabase, donde no hay JWT y por eso `auth.uid()` es nulo. El cambio queda en la auditoría.

| Decisión | Requerimientos | Por qué |
|---|---|---|
| Tipo de sangre en una tabla aparte | S-16, RNF-22 | RLS filtra filas, no columnas: en `perfiles`, el admin lo vería |
| El cliente edita su perfil con un `grant update (columnas)` | RF-04, RF-06 | RLS deja editar la fila propia; el grant por columna impide que el cliente cambie su `rol` |
| La fecha de nacimiento no se puede cambiar una vez cargada | RN-06, RN-10, RF-07 | Si se pudiera editar, un menor podría comprar una +18 o alguien de 40 usar el cupón de mayores de 50. El grant la incluye solo para la primera carga |

### 5.3 Catálogo

```sql
create table public.generos (
  id bigint generated always as identity primary key,
  nombre text not null unique check (char_length(nombre) between 1 and 40)
);

create table public.peliculas (
  id bigint generated always as identity primary key,
  nombre text not null check (char_length(nombre) between 1 and 120),
  sinopsis text not null default '' check (char_length(sinopsis) <= 2000),
  duracion_min int not null check (duracion_min between 1 and 600),
  fecha_estreno date not null,
  clasificacion public.clasificacion not null,
  estado public.estado_pelicula not null default 'borrador',
  poster_url text,
  preventa_activa boolean not null default false,
  preventa_porcentaje numeric(5,2) not null default 0 check (preventa_porcentaje between 0 and 100),
  preventa_dias int not null default 7 check (preventa_dias between 1 and 60),
  alertas_enviadas_en timestamptz,
  creado_en timestamptz not null default now()
);

create table public.peliculas_generos (
  pelicula_id bigint not null references public.peliculas (id) on delete cascade,
  genero_id bigint not null references public.generos (id) on delete restrict,
  primary key (pelicula_id, genero_id)
);
```

- RF-08 a RF-11: los datos de la película, géneros con relación muchos a muchos, la clasificación y el estado. El estado lo cambia el admin a mano (D-04).
- RF-70: las columnas `preventa_*`. La preventa se calcula con `en_preventa()` (sección 6.1) y **no** necesita una tarea programada para cerrarse (RF-71, RN-08).
- "Uno o más géneros" (RF-09) lo valida el formulario con Signal Forms. No es una regla crítica.

Vista para la cartelera. `security_invoker` hace que la vista respete el RLS de quien consulta.

```sql
create view public.peliculas_cartelera with (security_invoker = true) as
select
  p.*,
  coalesce((select array_agg(g.nombre order by g.nombre) from public.peliculas_generos pg
            join public.generos g on g.id = pg.genero_id where pg.pelicula_id = p.id), '{}') as generos,
  coalesce((select array_agg(distinct f.formato) from public.funciones f
            where f.pelicula_id = p.id and f.inicio > now()), '{}') as formatos,
  coalesce((select array_agg(distinct f.idioma) from public.funciones f
            where f.pelicula_id = p.id and f.inicio > now()), '{}') as idiomas,
  (select round(avg(r.estrellas), 1) from public.resenas r where r.pelicula_id = p.id) as promedio,
  (select count(*) from public.resenas r where r.pelicula_id = p.id) as cantidad_resenas,
  public.en_preventa(p.id, now()) as en_preventa
from public.peliculas p
where p.estado in ('proximamente', 'en_cartelera');
```

La vista se crea directamente en su versión final, en `tp1-sistema-cine.sql`. Hasta F12, `promedio` es nulo y `cantidad_resenas` vale 0, porque todavía no hay reseñas.

**Por qué:** La cartelera es chica. Traerla entera una vez y filtrarla en el cliente con `computed` (RF-14) es más simple que armar filtros de PostgREST sobre tablas relacionadas, y es lo que hace `TaskList` en clase.

### 5.4 Salas y butacas

```sql
create table public.salas (
  id bigint generated always as identity primary key,
  numero int not null unique check (numero > 0),
  nombre text,
  formatos public.formato[] not null default '{2D,3D,4D,5D}' check (cardinality(formatos) > 0),
  activa boolean not null default true
);

create table public.plantilla_butacas (
  fila char(1) not null check (fila between 'A' and 'T'),
  numero int not null check (numero > 0),
  bloque public.bloque not null,
  tipo public.tipo_butaca not null,
  primary key (fila, numero)
);

create table public.butacas (
  id bigint generated always as identity primary key,
  sala_id bigint not null references public.salas (id) on delete cascade,
  fila char(1) not null,
  numero int not null,
  bloque public.bloque not null,
  tipo public.tipo_butaca not null,
  fuera_de_servicio boolean not null default false,
  unique (sala_id, fila, numero)
);
```

La plantilla se carga en el mismo script: 18 filas de 28 butacas más J y K con 14, en total 532 (S-01, RF-17 a RF-19).

```sql
with filas as (
  select chr(64 + i) as fila,
         case when i in (10, 11) then 2 else 4 end as lateral,
         case when i in (10, 11) then 10 else 20 end as central,
         case when i in (10, 11) then 'accesible' when i >= 18 then 'vip' else 'estandar' end::public.tipo_butaca as tipo
  from generate_series(1, 20) as i
)
insert into public.plantilla_butacas (fila, numero, bloque, tipo)
select f.fila, n,
       case when n <= f.lateral then 'izquierdo'
            when n <= f.lateral + f.central then 'central'
            else 'derecho' end::public.bloque,
       f.tipo
from filas f
cross join lateral generate_series(1, 2 * f.lateral + f.central) as n;
```

- **Numeración:** cada fila se numera de izquierda a derecha, desde 1. En una fila estándar, del 1 al 4 es el bloque izquierdo, del 5 al 24 el central y del 25 al 28 el derecho. En J y K: 1-2, 3-12 y 13-14.
- **Butacas por sala:** el trigger `crear_butacas_sala` copia la plantilla cada vez que se da de alta una sala. Desde ese momento el admin puede cambiar el tipo de cada butaca o marcarla fuera de servicio (RF-20). Marcar una butaca fuera de servicio afecta solo las ventas nuevas (D-07).
- **Sala:** `numero` es obligatorio porque la asignación elige "la de menor número" (RN-03); `nombre` es opcional (D-06). Los formatos se declaran por sala (S-03).

| Decisión | Por qué |
|---|---|
| La plantilla es una tabla y no una constante en Angular | S-01 lo pide explícitamente: si se cambia la plantilla, no hay que tocar componentes |
| Una fila de `butacas` por sala, copiada de la plantilla | RF-20 pide distribución por sala; las ventas apuntan a butacas reales |

### 5.5 Funciones

```sql
create table public.funciones (
  id bigint generated always as identity primary key,
  pelicula_id bigint not null references public.peliculas (id) on delete restrict,
  sala_id bigint not null references public.salas (id) on delete restrict,
  inicio timestamptz not null,
  fin timestamptz not null,
  ocupacion tstzrange not null,
  formato public.formato not null,
  idioma public.idioma not null,
  creado_en timestamptz not null default now(),
  constraint funciones_sin_solapamiento exclude using gist (sala_id with =, ocupacion with &&)
);
create index funciones_pelicula_inicio on public.funciones (pelicula_id, inicio);
```

- El trigger `preparar_funcion` (BEFORE INSERT OR UPDATE) calcula `fin = inicio + duracion_min` (RF-21) y `ocupacion = [inicio, fin + minutos_limpieza)` (RN-01). También rechaza la fila si la sala está inactiva o no soporta el formato (RF-23, S-03).
- **Restricción de exclusión (RN-02):** dos funciones de la misma sala no pueden tener rangos de ocupación que se superpongan. La garantiza Postgres aunque dos administradores programen al mismo tiempo (S-19). El rango es semiabierto `[)`: una función que libera la sala a las 20:30 deja empezar otra a las 20:30.
- El trigger `proteger_funcion_vendida` impide borrar una función con butacas vendidas, o cambiarle `inicio`, `sala_id` o `pelicula_id` (RF-25).
- Formato e idioma se guardan por función (RF-12).
- Si se cambia `minutos_limpieza`, el valor nuevo rige para las funciones nuevas o modificadas; las existentes conservan su ocupación.

**Por qué el trigger y no una columna generada:** `timestamptz + interval` no es inmutable y la duración vive en otra tabla, así que Postgres no permite calcularlo en una columna generada.

### 5.6 Ocupación de butacas y compras

```sql
create table public.compras (
  id uuid primary key default gen_random_uuid(),
  usuario_id uuid not null references auth.users (id) on delete restrict,
  funcion_id bigint not null references public.funciones (id) on delete restrict,
  estado public.estado_compra not null default 'confirmada',
  codigo text not null unique,
  subtotal_entradas numeric(12,2) not null check (subtotal_entradas >= 0),
  subtotal_candy numeric(12,2) not null default 0 check (subtotal_candy >= 0),
  subtotal_combos numeric(12,2) not null default 0 check (subtotal_combos >= 0),
  cupon_id bigint references public.cupones (id),
  descuento numeric(12,2) not null default 0 check (descuento >= 0),
  total numeric(12,2) not null check (total >= 0),
  credito_usado numeric(12,2) not null default 0 check (credito_usado >= 0),
  pagado_con_tarjeta numeric(12,2) not null default 0 check (pagado_con_tarjeta >= 0),
  puntos_ganados int not null default 0,
  puntos_canjeados int not null default 0,
  tiene_candy boolean not null default false,
  declaracion_edad boolean not null default false,
  acceso_validado_en timestamptz,
  acceso_validado_por uuid references public.perfiles (id),
  candy_validado_en timestamptz,
  candy_validado_por uuid references public.perfiles (id),
  confirmada_en timestamptz not null default now(),
  cancelada_en timestamptz,
  check (total = subtotal_entradas + subtotal_candy + subtotal_combos - descuento),
  check (total = credito_usado + pagado_con_tarjeta)
);

create table public.ocupaciones (
  funcion_id bigint not null references public.funciones (id) on delete cascade,
  butaca_id bigint not null references public.butacas (id) on delete restrict,
  estado public.estado_ocupacion not null,
  usuario_id uuid not null references auth.users (id) on delete cascade,
  vence_en timestamptz,
  compra_id uuid references public.compras (id) on delete cascade,
  primary key (funcion_id, butaca_id),
  check ((estado = 'retenida' and vence_en is not null and compra_id is null)
      or (estado = 'vendida' and compra_id is not null))
);

create table public.entradas (
  id bigint generated always as identity primary key,
  compra_id uuid not null references public.compras (id) on delete cascade,
  funcion_id bigint not null references public.funciones (id),
  butaca_id bigint not null references public.butacas (id),
  tipo public.tipo_butaca not null,
  precio numeric(12,2) not null check (precio >= 0),
  combo_id bigint references public.combos (id),
  recompensa_id bigint references public.recompensas (id),
  check (combo_id is null or recompensa_id is null)
);
```

Estas tablas se crean en su forma final, con todas sus FK: `tp1-sistema-cine.sql` crea antes las tablas a las que apuntan (`cupones`, `combos` y `recompensas`).

- **Unicidad de función y butaca (RN-04):** es la clave primaria `(funcion_id, butaca_id)` de `ocupaciones`. Una butaca tiene como máximo **una** fila por función, sea una retención o una venta. Con dos compras simultáneas, una gana el `insert` y la otra choca contra la clave.
- **Retención (RN-05, S-17):** una fila `retenida` con `vence_en`. La función que reserva reemplaza las retenciones vencidas, y un job de pg_cron las borra cada minuto para que Realtime avise al resto.
- **Venta:** la misma fila pasa a `vendida` con su `compra_id`. Si se cancela la compra, la fila se borra y la butaca queda libre (RN-18). `entradas` queda como historial.
- **Código del QR (RF-54, S-02):** `codigo` tiene 10 caracteres aleatorios de un alfabeto de 32 símbolos sin I, O, 0 ni 1 (unos 50 bits). No contiene el id de la compra ni datos personales. El QR codifica el mismo código que se imprime en texto como `ABCD-EFGH-JK`, así que hay un solo código (RF-54).
- **Estado visible (RF-05, D-05):** `cancelada` si se canceló; `usada` si se validó el acceso o la función ya terminó; `vigente` en cualquier otro caso. Lo calcula la vista `mis_compras` (sección 6.2.9).
- **Una compra corresponde a una sola función** (D-20).

| Decisión | Por qué |
|---|---|
| Una sola tabla `ocupaciones` para retenciones y ventas | El mapa lee y escucha por Realtime una sola tabla, y la clave primaria resuelve RN-04 sin índices parciales |
| `entradas` separada de `ocupaciones` | `ocupaciones` es el estado actual del mapa y se borra al cancelar; `entradas` es lo que se compró y no se borra |
| `compras.id` uuid y `codigo` aparte | El id no aparece nunca en el QR. El uuid tampoco se puede adivinar en una URL como `/cuenta/compras/:id` |

### 5.7 Candy y combos

```sql
create table public.categorias (
  id bigint generated always as identity primary key,
  nombre text not null unique
);

create table public.productos (
  id bigint generated always as identity primary key,
  categoria_id bigint not null references public.categorias (id) on delete restrict,
  nombre text not null,
  precio numeric(12,2) not null check (precio > 0),
  imagen_url text,
  disponible boolean not null default true
);

create table public.combos (
  id bigint generated always as identity primary key,
  nombre text not null,
  precio numeric(12,2) not null check (precio > 0),
  imagen_url text,
  activo boolean not null default true
);

create table public.combos_productos (
  combo_id bigint not null references public.combos (id) on delete cascade,
  producto_id bigint not null references public.productos (id) on delete restrict,
  cantidad int not null check (cantidad > 0),
  primary key (combo_id, producto_id)
);

create table public.compras_items (
  id bigint generated always as identity primary key,
  compra_id uuid not null references public.compras (id) on delete cascade,
  producto_id bigint references public.productos (id),
  combo_id bigint references public.combos (id),
  recompensa_id bigint references public.recompensas (id),
  cantidad int not null check (cantidad > 0),
  precio_unitario numeric(12,2) not null check (precio_unitario >= 0),
  check (num_nonnulls(producto_id, combo_id) = 1)
);
```

- **Combo (RN-15, S-12):** cada combo incluye una entrada 2D estándar. Por cada combo hay un ítem en `compras_items` con el precio fijo, y una entrada con `combo_id` cuyo `precio` es solo la **diferencia**: precio base del formato menos precio base 2D, más el recargo si la butaca es VIP. La preventa no se aplica al combo (D-02). Los combos cubren primero las butacas no VIP (D-12).
- **Subtotales (D-13):** cada compra guarda tres subtotales, que son las tres líneas del reporte de facturación.

  | Subtotal | Qué suma |
  |---|---|
  | `subtotal_entradas` | Las entradas sueltas y las diferencias de formato o VIP de las entradas cubiertas por un combo o una recompensa |
  | `subtotal_candy` | Los productos sueltos |
  | `subtotal_combos` | El precio fijo de los combos |

  El precio del combo no se reparte entre entrada y candy.
- **Candy sin entrada (RN-14, S-11):** no se puede, porque `confirmar_compra` exige al menos una butaca retenida.
- **Orden de creación:** `compras_items` se crea con todas sus FK, después de `recompensas`.

### 5.8 Cupones, puntos, crédito y recompensas

```sql
create table public.cupones (
  id bigint generated always as identity primary key,
  codigo text not null unique check (codigo = upper(codigo) and codigo ~ '^[A-Z0-9_-]{3,20}$'),
  tipo public.tipo_cupon not null default 'general',
  porcentaje numeric(5,2) not null check (porcentaje > 0 and porcentaje <= 100),
  vigente_desde date,
  vigente_hasta date,
  limite_usos int check (limite_usos > 0),
  usos int not null default 0,
  edad_minima int check (edad_minima between 1 and 120),
  activo boolean not null default true,
  check (vigente_desde is null or vigente_hasta is null or vigente_desde <= vigente_hasta),
  check (limite_usos is null or usos <= limite_usos),
  check ((tipo = 'edad') = (edad_minima is not null))
);
create unique index cupones_una_primera_compra on public.cupones (tipo) where tipo = 'primera_compra';

create table public.recompensas (
  id bigint generated always as identity primary key,
  nombre text not null,
  tipo text not null check (tipo in ('entrada', 'producto')),
  producto_id bigint references public.productos (id),
  costo_puntos int not null check (costo_puntos > 0),
  activa boolean not null default true,
  check ((tipo = 'producto') = (producto_id is not null))
);

create table public.puntos_movimientos (
  id bigint generated always as identity primary key,
  usuario_id uuid not null references public.perfiles (id) on delete cascade,
  compra_id uuid references public.compras (id),
  tipo text not null check (tipo in ('ganados', 'canje', 'anulacion', 'devolucion')),
  puntos int not null check (puntos <> 0),
  disponible_desde timestamptz not null default now(),
  creado_en timestamptz not null default now()
);

create table public.creditos_movimientos (
  id bigint generated always as identity primary key,
  usuario_id uuid not null references public.perfiles (id) on delete cascade,
  compra_id uuid references public.compras (id),
  tipo text not null check (tipo in ('cancelacion', 'uso')),
  monto numeric(12,2) not null check (monto <> 0),
  creado_en timestamptz not null default now()
);
```

- **Cupón de primera compra (RF-03, RF-41, RN-09):** es una sola fila sembrada (`BIENVENIDA`, 20 %) con el porcentaje editable. No se asigna a cada usuario: un cliente puede usarlo si no tiene ninguna compra, confirmada o cancelada (D-18). Se aplica el porcentaje vigente al momento de pagar.
- **Cupón por edad (RF-42, RN-10, S-06):** `tipo = 'edad'` con `edad_minima` (50 por defecto), en años cumplidos a la fecha de compra.
- **Límite de usos:** es global (D-03). Cancelar una compra no devuelve el uso del cupón (S-20).
- **Restricciones generales:** todos los cupones requieren cliente registrado (tabla de roles, S-04), y hay uno solo por compra (RN-11, S-23).
- **Puntos (RF-64, RN-12, RN-19, S-07):** se ganan `floor(total)`, donde `total` es el monto después del cupón e incluye la parte pagada con crédito. El movimiento `ganados` tiene `disponible_desde = funciones.inicio`.
  - Saldo disponible: `sum(puntos) where disponible_desde <= now()`.
  - Saldo pendiente: lo que tiene `disponible_desde > now()`.
  - No hace falta un proceso programado.
- **Canje (RF-45, RN-13):** genera un movimiento `canje` negativo al confirmar. Lo canjeado tiene precio 0, así que no suma puntos. La recompensa de entrada cubre una 2D estándar y se paga la diferencia (D-01).
- **Cancelación (RN-18):** genera un movimiento `anulacion` por los puntos pendientes, con el mismo `disponible_desde`, y uno `devolucion` por los canjeados.
- **Crédito (RF-44, RF-74 a RF-76):** se calcula como suma de movimientos. La cancelación lo genera y el uso en una compra lo consume.
- **Puntos no transferibles (RF-67):** no hay ninguna operación de transferencia, y los movimientos solo los escriben las RPC.

Vistas con `security_invoker = true`:

| Vista | Qué devuelve |
|---|---|
| `saldos_puntos` | `usuario_id`, `disponibles`, `pendientes` |
| `saldos_credito` | `usuario_id`, `saldo` |

**Por qué movimientos y no un campo saldo:** RF-66 y RF-75 piden los movimientos, y la cancelación se resuelve agregando filas en lugar de recalcular saldos.

### 5.9 Reseñas, alertas, notificaciones y push

```sql
create table public.resenas (
  id bigint generated always as identity primary key,
  pelicula_id bigint not null references public.peliculas (id) on delete cascade,
  usuario_id uuid not null references public.perfiles (id) on delete cascade,
  autor text not null,
  estrellas int not null check (estrellas between 1 and 5),
  comentario text not null default '' check (char_length(comentario) <= 280),
  creada_en timestamptz not null default now(),
  editada_en timestamptz,
  unique (pelicula_id, usuario_id)
);

create table public.alertas (
  usuario_id uuid not null references public.perfiles (id) on delete cascade,
  pelicula_id bigint not null references public.peliculas (id) on delete cascade,
  creada_en timestamptz not null default now(),
  primary key (usuario_id, pelicula_id)
);

create table public.notificaciones (
  id bigint generated always as identity primary key,
  usuario_id uuid not null references public.perfiles (id) on delete cascade,
  tipo text not null check (tipo in ('alerta', 'recordatorio_24', 'recordatorio_2', 'validacion_acceso', 'validacion_candy')),
  titulo text not null,
  cuerpo text not null,
  url text not null,
  compra_id uuid references public.compras (id) on delete cascade,
  pelicula_id bigint references public.peliculas (id) on delete cascade,
  leida boolean not null default false,
  creada_en timestamptz not null default now(),
  unique (compra_id, tipo)
);

create table public.suscripciones_push (
  id bigint generated always as identity primary key,
  usuario_id uuid not null references public.perfiles (id) on delete cascade,
  endpoint text not null unique,
  p256dh text not null,
  auth text not null,
  creada_en timestamptz not null default now()
);
```

- **Reseñas (RF-61 a RF-63, RN-21, S-14):**
  - Una por usuario y película: es la restricción `unique`.
  - Solo puede reseñar quien vio la película: la política de INSERT exige `vio_pelicula(pelicula_id)`.
  - `autor` lo completa un trigger con el nombre y la inicial del apellido (D-19), para no exponer `perfiles`.
- **Mis películas (RF-72, RN-22, S-13):** es la vista `mis_peliculas` (sección 6.2.9).
- **Notificaciones:** son la bandeja de la app (RNF-16, S-10). `unique (compra_id, tipo)` evita repetir recordatorios y avisos. Cada INSERT llega por Realtime a la bandeja y, con la app abierta, se muestra como aviso del sistema (sección 9).
- **Destinatarios:** solo clientes registrados, porque las FK apuntan a `perfiles` (D-08).
- **`suscripciones_push`:** una fila por dispositivo suscripto. La crea `03_push.sql` en F16 (D-25). El RLS deja leer solo las propias; las altas y bajas pasan por las RPC `guardar_suscripcion_push` (si el navegador era de otra cuenta, pasa a la actual) y `borrar_suscripcion_push`.

### 5.10 Auditoría

```sql
create table public.auditoria (
  id bigint generated always as identity primary key,
  usuario_id uuid references public.perfiles (id) on delete set null,
  accion text not null,
  entidad text not null,
  entidad_id text,
  valores_anteriores jsonb,
  valores_nuevos jsonb,
  creado_en timestamptz not null default now()
);
create index auditoria_creado_en on public.auditoria (creado_en desc);
```

- Cumple RF-83: quién (`usuario_id`; nulo cuando actúa el sistema), qué acción, sobre qué entidad, cuándo, y los valores anterior y nuevo.
- La escriben solo los triggers y las RPC, que son `security definer`. No hay política de INSERT para nadie (RN-24).
- El filtro del admin (RF-85) usa `usuario_id`, `accion` y `creado_en`.

### 5.11 Políticas RLS por rol

RLS está activado en **todas** las tablas (RNF-10).

- Las funciones `es_admin()`, `es_staff()`, `es_registrado()` y `puede_comprar()` son `security definer` (sección 6.1).
- En las políticas se escriben como `(select public.es_admin())` para que Postgres las evalúe una vez por consulta y no una vez por fila.
- Las vistas usan `security_invoker = true`.

**Por qué `security_invoker`:** Sin esa opción, una vista corre con los permisos de su dueño y se saltea el RLS.

S = select, I = insert, U = update, D = delete. "RPC" significa que nadie escribe directo: solo la función indicada.

| Tabla | Sin sesión (`anon`) | Anónimo (`authenticated`, `is_anonymous`) | Cliente | Empleado | Admin |
|---|---|---|---|---|---|
| `perfiles` | — | — | S y U propio (solo columnas permitidas) | S y U propio | S todos; el rol lo cambia la RPC `asignar_rol` |
| `datos_sensibles` | — | — | S I U D propio | S I U D propio | S I U D propio; no ve los de otros |
| `configuracion`, `precios_formato` | S | S | S | S | S, U |
| `generos`, `peliculas_generos` | S | S | S | S | S I U D |
| `peliculas` | S (sin borradores) | S (sin borradores) | S (sin borradores) | S (sin borradores) | S I U D |
| `plantilla_butacas` | S | S | S | S | S |
| `salas`, `butacas` | S | S | S | S | S I U D |
| `funciones` | S | S | S | S | S, D; I y U por RPC `programar_funciones` y `modificar_funcion` |
| `ocupaciones` | S | S | S | S | S; I U D por RPC `retener_butacas`, `liberar_butacas`, `confirmar_compra` y `cancelar_compra` |
| `compras` | — | S propias | S propias | — (lee por RPC `validar_qr`) | S todas |
| `entradas`, `compras_items` | — | S de compras propias | S de compras propias | — | S todas |
| `categorias`, `productos`, `combos`, `combos_productos`, `recompensas` | S | S | S | S | S I U D |
| `cupones` | — | — | — (por RPC `validar_cupon` y `cupones_sugeridos`) | — | S I U D |
| `puntos_movimientos`, `creditos_movimientos` | — | — | S propios | — | — |
| `resenas` | S | S | S, I (si `vio_pelicula`), U propia | S | S |
| `alertas` | — | — | S I D propias (solo de películas en `proximamente`) | — | — |
| `notificaciones` | — | — | S propias; U solo `leida` | S, U propias | S, U propias |
| `suscripciones_push` (F16) | — | — | S propias; alta y baja por RPC | S propias; alta y baja por RPC | S propias; alta y baja por RPC |
| `auditoria` | — | — | — | — | S |

Ejemplo del estilo de las políticas:

```sql
alter table public.peliculas enable row level security;

create policy "lee peliculas publicadas" on public.peliculas
  for select to anon, authenticated
  using (estado <> 'borrador' or (select public.es_admin()));

create policy "admin gestiona peliculas" on public.peliculas
  for all to authenticated
  using ((select public.es_admin()))
  with check ((select public.es_admin()));

create policy "titular gestiona sus datos sensibles" on public.datos_sensibles
  for all to authenticated
  using (usuario_id = (select auth.uid()))
  with check (usuario_id = (select auth.uid()));

revoke insert, update, delete on public.perfiles from anon, authenticated;
grant update (nombre, apellido, fecha_nacimiento, color_ojos, dias_vacaciones) on public.perfiles to authenticated;
```

El grant incluye `fecha_nacimiento` solo para la primera carga. Después, el trigger `proteger_fecha_nacimiento` rechaza cualquier cambio que llegue por la API (sección 5.2). Sin ese trigger, el cliente podría cambiar su edad y saltearse RN-06 y RN-10.

Además, toda función interna (`asignar_sala`, `generar_codigo`, `simular_pago`, `procesar_*` y las de los triggers) lleva `revoke execute on function … from public, anon, authenticated`.

**Por qué:** PostgREST publica como RPC toda función del esquema `public`, y por defecto cualquiera puede ejecutarla.

Las RPC que necesitan sesión tampoco las ejecuta `anon` (`02_permisos_anon.sql`). Sin sesión solo quedan `es_admin`, que usa la política "lee peliculas publicadas", y `top_peliculas`, que usa la portada. Las auxiliares (`en_preventa`, `precio_entrada`, `fecha_ba`, `edad`, `edad_minima` y `butacas_contiguas`) siguen abiertas, porque `en_preventa` la usa la vista `peliculas_cartelera` que lee el visitante.

### 5.12 Scripts SQL

La base se arma con **dos scripts** que se pegan en el SQL Editor (sección 7.6). Los dos viven en `supabase/sql/`, una carpeta local que no se sube al repo (`.git/info/exclude`).

| Script | Estado | Crea |
|---|---|---|
| `01_base.sql` | aplicado en F0 | `configuracion` y `precios_formato` con sus datos, RLS y la política de lectura |
| `tp1-sistema-cine.sql` | aplicado en F0 (07/10) | Todo lo demás, en su versión final: las extensiones, las 25 tablas con sus datos iniciales (la plantilla de 532 butacas y el cupón `BIENVENIDA`), las funciones auxiliares, los triggers, las RPC de F1 a F14, las vistas, el RLS de cada tabla, las políticas de Storage, la publicación de Realtime y los tres jobs de pg_cron. No incluye `suscripciones_push`, que es de la fase opcional F16 |
| `02_permisos_anon.sql` | escrito el 07/10, probado, falta aplicar | Quita a `anon` el permiso de ejecutar las 19 RPC que necesitan sesión y se lo da explícito a `authenticated`. Sin sesión solo quedan `es_admin` y `top_peliculas` |

Orden dentro de `tp1-sistema-cine.sql`: extensiones → tablas → funciones auxiliares → triggers → RPC → vistas → RLS y permisos → Storage → Realtime → pg_cron.

**Por qué ese orden:** Una FK exige que exista la tabla a la que apunta, y Postgres valida al crear una función SQL que existan las tablas y las funciones que usa.

**Por qué un solo script:** Se pidió un único archivo con todo lo que falta, en lugar de un script por fase. Así cada tabla nace con todas sus FK y cada función en su versión final: no hay `alter table` ni `create or replace function` entre fases. Angular usa cada parte cuando llega su fase.

**Cómo se probó:** se corrió entero, después de `01_base.sql`, en una Postgres en memoria (PGlite) con imitaciones de `auth`, `storage`, `cron` y la publicación de Realtime. Un recorrido de 151 controles pasó con cliente, anónimo, empleado y admin: perfiles y fecha inmutable, exclusión de horarios, retenciones, compra con cupón, candy, combo, crédito y canje, pago rechazado, validación del QR, cancelación, reportes, avisos, auditoría y RLS. Lo propio de Supabase (permisos sobre `auth.users`, `storage.objects`, `cron` y la publicación) se confirma al pegarlo, con las consultas de la sección 7.6.

Cambios después de aplicarlo:

- **Corrección o cambio de la base:** va en un script nuevo `NN_motivo.sql`, numerado desde `02`, que se aplica después de `tp1-sistema-cine.sql`.
- **Un script aplicado no se edita:** la base ya tiene ese estado. Si se editara el archivo, el archivo y la base dejarían de coincidir.

Ajustes respecto del diseño de la sección 6, decididos al escribir el script:

| Ajuste | Por qué |
|---|---|
| `comprobante` es `security definer` y controla de forma explícita que quien consulta sea el dueño o el admin | Con `security invoker`, el cliente no podía leer el código del cupón, porque `cupones` solo lo lee el admin |
| `confirmar_compra` lanza `CREDITO_INSUFICIENTE` si se pide más crédito que el saldo | Con `least(...)` se habría cobrado en silencio más a la tarjeta de lo que el cliente vio |
| Códigos nuevos: `RECOMPENSA_NO_DISPONIBLE`, `RECOMPENSAS_SUPERAN_ENTRADAS`, `FUNCION_INEXISTENTE`, `SECTOR_INVALIDO` y `PERIODO_INVALIDO`. El pago no aprobado devuelve `motivo` `PAGO_RECHAZADO` o `PAGO_ERROR` | Casos que el diseño no nombraba. `mensajeError()` los traduce en la fase que los usa |
| `procesar_alertas` exige en los dos casos una función futura | Una preventa abierta sin funciones todavía no tiene nada para vender |
| `proteger_funcion_vendida` impide borrar una función que tuvo cualquier compra, aunque esté cancelada | Las entradas de esa compra siguen apuntando a la función |
| `auditar_cambio` descarta los `update` que no cambian nada | Guardar un formulario sin tocarlo no ensucia la auditoría |
| El script crea también el perfil de los usuarios que ya existían | Si alguien se registró antes de pegarlo, igual tiene perfil |
| Storage suma la política "admin lee imagenes" | Subir con `upsert: true` necesita `select`, además de `insert` y `update` |

---

## 6. Lógica en la base

Reglas comunes a todas las funciones:

- Las que tocan datos de otros usuarios son `security definer` con `set search_path = ''` y nombres completos (`public.tabla`, `extensions.gen_random_bytes`).
- Cada RPC corre en **una sola transacción**: si algo falla, no queda nada a medias (RNF-11).
- Los errores se lanzan como `raise exception 'CODIGO' using detail = '…'`. Angular los traduce con `mensajeError()` (sección 8.3).

**Por qué `search_path` vacío:** Impide que alguien cree un objeto con el mismo nombre en otro esquema y lo haga ejecutar con los permisos del dueño de la función.

### 6.1 Funciones auxiliares

| Función | Firma | Qué hace | Requerimientos |
|---|---|---|---|
| `rol_actual` | `() returns rol_usuario` (definer, stable) | Rol del usuario autenticado, o `null` si es anónimo o no tiene sesión | RNF-09 |
| `es_admin`, `es_staff` | `() returns boolean` | `rol_actual() = 'admin'` y `rol_actual() in ('empleado', 'admin')` | RNF-06, RNF-10 |
| `es_registrado` | `() returns boolean` | `auth.uid()` no es nulo y el claim `is_anonymous` del JWT es falso | S-04 |
| `puede_comprar` | `() returns boolean` | Hay sesión y el usuario es anónimo o tiene `rol = 'cliente'` | Tabla de roles |
| `edad` | `(p_nacimiento date, p_fecha date) returns int` (immutable) | `extract(year from age(p_fecha, p_nacimiento))`, en años cumplidos | RF-07 |
| `edad_minima` | `(p_clasificacion) returns int` | ATP: 0 · +13: 13 · +18: 18 | RN-06 |
| `fecha_ba` | `(p_momento timestamptz) returns date` | `(p_momento at time zone 'America/Argentina/Buenos_Aires')::date` | RNF-21 |
| `en_preventa` | `(p_pelicula_id bigint, p_momento timestamptz) returns boolean` | `preventa_activa` y `p_momento` dentro de `[00:00 BA de (estreno − preventa_dias), 00:00 BA del estreno)` | RN-08, S-09, RF-71 |
| `precio_entrada` | `(p_funcion_id bigint, p_tipo tipo_butaca, p_momento timestamptz) returns numeric` | `round(precio_base × (1 − preventa%) + recargo_vip si es VIP, 2)` | RN-07, S-22 |
| `butacas_contiguas` | `(p_butacas bigint[]) returns text` | Devuelve `null` si las butacas son contiguas; si no, el código del motivo (ver abajo) | RN-25 |
| `generar_codigo` | `() returns text` (volatile) | 10 caracteres del alfabeto `ABCDEFGHJKLMNPQRSTUVWXYZ23456789` tomados de `extensions.gen_random_bytes(10)`. 32 divide a 256, así que no hay sesgo | RF-54, S-02 |
| `luhn_valido` | `(p_numero text) returns boolean` (immutable) | Algoritmo de Luhn sobre 13 a 19 dígitos | S-08 |
| `simular_pago` | `(p_pago jsonb, p_monto numeric) returns text` | Devuelve `'aprobado'`, `'rechazado'` o `'error'` (ver abajo). No guarda ni registra los datos de la tarjeta | RF-37, S-08 |
| `validar_venta` | `(p_funcion_id bigint) returns void` | La función empieza en el futuro (D-10). La película está `en_cartelera`, o `proximamente` con `en_preventa(now())`. Si el usuario está registrado: perfil completo y `edad(nacimiento, fecha_ba(inicio)) >= edad_minima(clasificacion)` | RN-06, RF-35 |
| `vio_pelicula` | `(p_pelicula_id bigint) returns boolean` (definer) | Existe una compra confirmada del usuario para esa película con acceso validado o con la función terminada | RN-21, RN-22 |
| `motivo_cupon` | `(p_cupon cupones, p_uid uuid) returns text` | Devuelve `null` o el motivo del rechazo (ver abajo) | RN-09, RN-10 |

- **`butacas_contiguas`**, códigos posibles: `BUTACAS_DISTINTA_FILA`, `BUTACAS_SEPARADAS_POR_PASILLO` o `BUTACAS_NO_CONSECUTIVAS`. Para el último compara `max(numero) − min(numero) + 1` con la cantidad de butacas.
- **`simular_pago`**: si el número no pasa Luhn, el vencimiento es anterior al mes actual o el CVV no tiene 3 o 4 dígitos, devuelve `'rechazado'`. Con datos válidos decide la tarjeta de prueba (D-22): `…0002` → `'rechazado'`, `…0119` → `'error'`, cualquier otra → `'aprobado'`.
- **`motivo_cupon`**, códigos posibles: `SOLO_REGISTRADOS`, `CUPON_INACTIVO`, `CUPON_VENCIDO`, `CUPON_AGOTADO`, `CUPON_SOLO_PRIMERA_COMPRA` o `CUPON_EDAD`.

Ejemplo:

```sql
create function public.en_preventa(p_pelicula_id bigint, p_momento timestamptz)
returns boolean language sql stable set search_path = '' as $$
  select p.preventa_activa
     and p_momento >= ((p.fecha_estreno - p.preventa_dias)::timestamp at time zone 'America/Argentina/Buenos_Aires')
     and p_momento < (p.fecha_estreno::timestamp at time zone 'America/Argentina/Buenos_Aires')
  from public.peliculas p
  where p.id = p_pelicula_id
$$;
```

### 6.2 RPC transaccionales

Todas se llaman desde Angular con `supabase.rpc(...)` dentro de su servicio de Api (sección 8.3).

#### 6.2.1 `asignar_sala` (interna)

| | |
|---|---|
| Firma | `asignar_sala(p_formato formato, p_inicio timestamptz, p_duracion_min int, p_excluir_funcion bigint default null) returns bigint` |
| Entradas | Formato, inicio, duración, y la función a ignorar cuando se está modificando una existente |
| Salida | El `id` de la sala elegida, o `null` |
| Lógica | Sala activa, compatible con el formato, sin ninguna función cuya `ocupacion` se superponga con `[inicio, inicio + duración + limpieza)`. Si hay varias, la de menor `numero` |
| Reglas | RN-01, RN-03, RF-23, S-19 |
| Errores | No lanza errores: devuelve `null` |

#### 6.2.2 `programar_funciones`

| | |
|---|---|
| Firma | `programar_funciones(p_pelicula_id bigint, p_formato formato, p_idioma idioma, p_hora time, p_fechas date[]) returns table (fecha date, funcion_id bigint, sala_numero int)` |
| Entradas | Película, formato, idioma, hora y las fechas ya expandidas por Angular a partir de los días de la semana y el rango (RF-22). Una función suelta (RF-21) es un arreglo de una sola fecha |
| Salida | Una fila por fecha. `funcion_id` y `sala_numero` son nulos si no hubo sala |
| Pasos | Paso 1: verifica `es_admin()`. Paso 2: por cada fecha calcula `inicio = (fecha + hora) at time zone 'America/Argentina/Buenos_Aires'`. Paso 3: llama a `asignar_sala`. Paso 4: inserta la función. Si la exclusión salta por un admin simultáneo, captura `exclusion_violation` y prueba otra vez con la sala siguiente |
| Reglas | RF-21 a RF-23, RN-01 a RN-03, S-19 |
| Errores | `SOLO_ADMIN`, `PELICULA_INEXISTENTE`, `SALA_INCOMPATIBLE` (lo lanza el trigger) |

La vista previa (RF-24) la calcula Angular con la regla pura `asignarSalas()` (sección 8.5), a partir de las salas y las funciones del rango. La RPC decide la asignación final y la pantalla muestra lo que realmente se creó.

**Por qué:** Así se cumple el patrón del documento: la interfaz anticipa y la base asegura. Además, la regla de asignación queda en TypeScript, donde Vitest la puede probar (RNF-23).

#### 6.2.3 `modificar_funcion`

| | |
|---|---|
| Firma | `modificar_funcion(p_funcion_id bigint, p_inicio timestamptz, p_formato formato, p_idioma idioma) returns jsonb` |
| Salida | `{ "funcion_id": 1, "sala_numero": 3 }` |
| Pasos | Paso 1: verifica `es_admin()` y que la función no tenga ventas. Paso 2: mantiene la sala si sigue siendo compatible y libre; si no, llama a `asignar_sala` excluyéndose a sí misma. Paso 3: actualiza la fila |
| Reglas | RF-21, RF-25, RN-01 a RN-03 |
| Errores | `SOLO_ADMIN`, `FUNCION_CON_VENTAS`, `SIN_SALA` |

#### 6.2.4 `retener_butacas`

| | |
|---|---|
| Firma | `retener_butacas(p_funcion_id bigint, p_butacas bigint[]) returns timestamptz` |
| Entradas | La función y la selección **completa** del usuario. Angular la llama cada vez que se toca una butaca |
| Salida | `vence_en`, común a toda la selección del usuario en esa función |
| Pasos | Paso 1: valida `puede_comprar()` y que `cardinality(p_butacas) <= max_butacas_compra`. Paso 2: llama a `validar_venta(p_funcion_id)`. Paso 3: borra las retenciones propias que ya no están en la selección. Paso 4: calcula el vencimiento como el mínimo `vence_en` vigente del usuario o, si no tiene ninguno, `now() + minutos_retencion` (D-11). Paso 5: inserta las butacas pidiendo `on conflict (funcion_id, butaca_id) do update … where ocupaciones.estado = 'retenida' and (ocupaciones.vence_en < now() or ocupaciones.usuario_id = excluded.usuario_id)`, excluyendo las fuera de servicio. Paso 6: cuenta las butacas propias retenidas y, si no son todas las pedidas, lanza `BUTACA_NO_DISPONIBLE`. La transacción se revierte entera y Angular vuelve a la selección anterior |
| Reglas | RN-04, RN-05, RN-06, RF-29, RF-38, S-17, S-21 |
| Errores | `NO_PUEDE_COMPRAR`, `MAXIMO_BUTACAS`, `FUNCION_NO_DISPONIBLE`, `PERFIL_INCOMPLETO`, `EDAD_INSUFICIENTE`, `BUTACA_NO_DISPONIBLE` |

La contigüidad **no** se valida al retener: el usuario todavía está eligiendo. Se valida en la interfaz con `butacasContiguas` y otra vez en `confirmar_compra` (RN-25).

#### 6.2.5 `liberar_butacas`

| | |
|---|---|
| Firma | `liberar_butacas(p_funcion_id bigint) returns void` |
| Lógica | Borra las retenciones del usuario en esa función. Angular la llama al abandonar el checkout o cuando vence el contador |
| Reglas | RF-29, RN-05 |
| Errores | Ninguno: si no hay retenciones, no hace nada |

#### 6.2.6 `confirmar_compra`

| | |
|---|---|
| Firma | `confirmar_compra(p_funcion_id bigint, p_productos jsonb default '[]', p_combos jsonb default '[]', p_recompensas jsonb default '[]', p_cupon text default null, p_credito numeric default 0, p_pago jsonb default null, p_declaracion_edad boolean default false) returns jsonb` |
| Entradas | `p_productos`: `[{ "producto_id": 1, "cantidad": 2 }]`. `p_combos`: `[{ "combo_id": 1, "cantidad": 1 }]`. `p_recompensas`: `[{ "recompensa_id": 3 }]`. `p_pago`: `{ "numero": "4242424242424242", "vencimiento": "12/27", "cvv": "123" }` |
| Salida | `{ "estado": "aprobado", "compra_id": "…", "codigo": "…" }`, o `{ "estado": "rechazado" \| "error", "motivo": "…" }` |
| Reglas | RN-04, RN-06, RN-07, RN-09 a RN-15, RN-19, RN-25, RF-32 a RF-38, RF-43 a RF-45, RF-47, RF-50, RF-53, RF-54, RF-64, S-04, S-08, S-11, S-12 |
| Errores | `NO_PUEDE_COMPRAR`, `PERFIL_INCOMPLETO`, `FUNCION_NO_DISPONIBLE`, `EDAD_INSUFICIENTE`, `DECLARACION_EDAD_REQUERIDA`, `RETENCION_VENCIDA`, `MAXIMO_BUTACAS`, `BUTACAS_DISTINTA_FILA`, `BUTACAS_SEPARADAS_POR_PASILLO`, `BUTACAS_NO_CONSECUTIVAS`, `PRODUCTO_NO_DISPONIBLE`, `COMBOS_SUPERAN_ENTRADAS`, `SOLO_REGISTRADOS`, `CUPON_INEXISTENTE` o el motivo de `motivo_cupon`, `CREDITO_INSUFICIENTE`, `PUNTOS_INSUFICIENTES`, `PAGO_REQUERIDO` |

Pasos:

1. Valida `puede_comprar()`. Si el usuario es registrado, bloquea su fila de `perfiles` con `for update`, lo que serializa sus operaciones de saldo.
2. Llama a `validar_venta`. Si el usuario es anónimo y la película es +13 o +18, exige `p_declaracion_edad = true` (RN-06, S-04).
3. Bloquea con `for update` las retenciones vigentes del usuario para la función. Exige entre 1 y `max_butacas_compra`, y que ninguna haya vencido (RN-05, RF-38).
4. Valida `butacas_contiguas` (RN-25).
5. Calcula el precio de cada entrada con `precio_entrada(..., now())` (RN-07, RN-08). Asigna los combos, y después las recompensas de entrada, a las butacas no VIP primero. Esas entradas pagan solo la diferencia (RN-15, D-01, D-12).
6. Calcula el candy y los combos:
   - Productos con `disponible`, que suman `subtotal_candy`.
   - Combos con `activo`, sin superar la cantidad de entradas, que suman `subtotal_combos` (D-13).
   - Recompensas de producto a precio 0.
   - RN-14 se cumple solo, porque siempre hay al menos una entrada.
7. Si `p_cupon` no es nulo, exige `es_registrado()`, bloquea la fila del cupón y valida `motivo_cupon`. Calcula `descuento = round(subtotal × porcentaje / 100, 2)` sobre entradas, candy y combos (RN-09 a RN-11).
8. Calcula `credito_usado = least(p_credito, saldo, total)` (RN-12, RF-44).
9. Exige puntos disponibles suficientes para los canjes (RN-13, RN-19).
10. Si `resto = total − credito_usado > 0`, llama a `simular_pago(p_pago, resto)`. Si no aprueba, **borra las retenciones** del usuario y devuelve `rechazado` o `error` sin lanzar excepción, para que la liberación quede guardada (RF-37).
11. Registra todo:
    - `compras`, con `codigo = generar_codigo()`.
    - `entradas` y `compras_items`.
    - `ocupaciones` pasa de `retenida` a `vendida`.
    - `cupones.usos + 1`.
    - Movimiento de crédito `uso`.
    - Puntos: `canje` negativo, y `ganados = floor(total)` con `disponible_desde = inicio`, solo para clientes registrados (RN-12, RN-19).
    - Auditoría de uso de cupón y de canje (RF-84).
12. Devuelve `aprobado` con `compra_id` y `codigo`.

La función se crea completa en `tp1-sistema-cine.sql`. Angular empieza a mandar los parámetros de candy, cupón, crédito y canje en las fases 8, 9, 10 y 13; hasta entonces van con su valor por defecto. En cada una de esas fases se repite la compra básica de F5 como prueba de regresión (sección 11).

**Por qué:** Angular no cambia de contrato fase a fase.

**Por qué el pago va en la RPC:** Aprobar en el servidor evita que alguien confirme sin pagar (S-08). Hacerlo en la misma transacción evita estados intermedios del tipo "pagó, pero no se registró la compra". El número y el CVV solo existen dentro de la llamada: no se guardan en ninguna tabla, no van a la auditoría y no aparecen en ningún mensaje de error.

#### 6.2.7 `validar_qr`

| | |
|---|---|
| Firma | `validar_qr(p_codigo text, p_sector sector) returns jsonb` |
| Entradas | El código leído del QR o tipeado (sin guiones, en mayúsculas o minúsculas) y el modo: `acceso` o `candy` |
| Salida | `{ "resultado": "valido" \| "ya_usado" \| "cancelado" \| "fuera_de_horario" \| "inexistente" \| "sin_candy", "validado_en": "…", "pendiente": "candy" \| "acceso" \| null, "pelicula": "…", "sala": 3, "inicio": "…", "butacas": [...], "productos": [...] }` |
| Reglas | RN-16, RN-17, RF-55 a RF-60, RF-89, S-02, S-18 |
| Errores | `SOLO_PERSONAL`. Las demás situaciones se informan en `resultado`, no como error |

Pasos:

1. Verifica `es_staff()`. Normaliza el código: `upper` y quita lo que no sea alfanumérico.
2. Bloquea la compra con `for update`. Si no existe, devuelve `inexistente`; si está cancelada, `cancelado`.
3. Si el sector ya se usó, devuelve `ya_usado` con la fecha y hora de esa validación y el sector que sigue pendiente. Si el modo es `candy` y la compra no tiene candy, devuelve `sin_candy` (D-14).
4. Controla la ventana horaria (RN-17, S-18):
   - Acceso: desde `inicio − minutos_previos_acceso` hasta `fin`.
   - Candy: desde las 00:00 BA del día de la función hasta `fin`.
   - Fuera de esa ventana devuelve `fuera_de_horario`.
5. Marca `*_validado_en = now()` y `*_validado_por = auth.uid()`. Escribe la auditoría con empleado, fecha, hora y sector (RF-60). Si el comprador es registrado, inserta una notificación `validacion_acceso` o `validacion_candy` (RF-89).
6. Devuelve `valido` con las butacas y su tipo, o los productos y combos a entregar.

El QR queda inválido cuando están consumidas todas las partes. Una compra sin candy se consume entera con el acceso (RF-59, S-02).

#### 6.2.8 `cancelar_compra`

| | |
|---|---|
| Firma | `cancelar_compra(p_compra_id uuid) returns jsonb` |
| Salida | `{ "credito": 12500.00 }` |
| Reglas | RN-18, RF-73 a RF-76, S-07, S-20 |
| Errores | `SOLO_REGISTRADOS`, `COMPRA_INEXISTENTE`, `COMPRA_YA_CANCELADA`, `FUERA_DE_PLAZO`, `COMPRA_USADA` |

Pasos:

1. Exige `es_registrado()` y que la compra sea del usuario. La bloquea con `for update`.
2. Exige `estado = 'confirmada'`, `now() <= inicio − horas_limite_cancelacion` y ningún sector validado.
3. Pone `estado = 'cancelada'` y `cancelada_en = now()`.
4. Borra sus `ocupaciones`: la butaca queda libre y Realtime avisa.
5. Acredita `total` (lo pagado, crédito incluido) como crédito.
6. Anula los puntos pendientes y devuelve los canjeados. El cupón **no** se restituye.
7. Escribe la auditoría (RF-84).

`cancelar_compra` se crea con el paso 6 incluido. En F10 la pantalla todavía no muestra puntos, y en F13 se repite la cancelación de F10 como regresión.

#### 6.2.9 Consultas de apoyo

| Función | Firma | Qué devuelve | Requerimientos |
|---|---|---|---|
| `top_peliculas` | `() returns table (pelicula_id bigint, entradas bigint)` (definer) | Hasta 3 películas. Cuenta las entradas vigentes compradas en los últimos `dias_ranking` días, de películas en cartelera o en preventa, ordenadas por cantidad. Si menos de 3 tienen ventas, completa con las películas en cartelera de estreno más reciente | RF-13, RN-20, S-15 |
| `cupones_sugeridos` | `(p_subtotal numeric) returns table (codigo text, porcentaje numeric, descuento numeric)` | Los cupones automáticos (primera compra y edad) para los que el cliente califica, de mayor a menor descuento | RF-43, RN-11, S-23 |
| `validar_cupon` | `(p_codigo text, p_subtotal numeric) returns jsonb` | `{ "valido": true, "porcentaje": 20, "descuento": 4000 }` o `{ "valido": false, "motivo": "CUPON_VENCIDO" }` | RF-43 |
| `comprobante` | `(p_compra_id uuid) returns jsonb` (invoker; el RLS limita al dueño o al admin) | Todo lo que necesitan el PDF y la pantalla del comprobante | RF-51 a RF-54, RF-05 |
| `reporte_facturacion` | `(p_desde date, p_hasta date) returns table (dia date, compras bigint, entradas bigint, subtotal_entradas numeric, subtotal_candy numeric, subtotal_combos numeric, descuentos numeric, total numeric)` | Compras confirmadas que siguen vigentes, agrupadas por día de Buenos Aires de `confirmada_en`, con tres líneas (entradas, candy y combos), los descuentos y el total | RF-79, RN-23, S-24, D-13 |
| `reporte_mas_vistas` | `(p_periodo text, p_desde date, p_hasta date) returns table (periodo date, pelicula_id bigint, nombre text, entradas bigint)` | Entradas vigentes por película, agrupadas con `date_trunc('week' \| 'month', inicio)` de la función | RF-80, RN-23 |
| `reporte_productos` | `(p_desde date, p_hasta date) returns table (nombre text, unidades bigint)` | Unidades vendidas por producto y combo, sin contar los canjes | RF-81 |

Las tres funciones de reporte exigen `es_admin()` y lanzan `SOLO_ADMIN` si no.

Vistas con `security_invoker`:

| Vista | Qué devuelve | Requerimientos |
|---|---|---|
| `mis_compras` | Compras del usuario con película, función y estado visible (`vigente`, `usada` o `cancelada`) | RF-05, D-05 |
| `mis_peliculas` | Películas vistas, con póster, fecha de la función y calificación propia | RF-72, RN-22, S-13 |

#### 6.2.10 `asignar_rol`

| | |
|---|---|
| Firma | `asignar_rol(p_usuario_id uuid, p_rol text, p_nombre text, p_apellido text) returns void` |
| Entradas | La cuenta recién creada con `signUp` desde el segundo cliente (sección 7.4), el rol (`empleado` o `admin`) y los datos de la persona |
| Pasos | Paso 1: verifica `es_admin()`. Paso 2: verifica que el rol sea `empleado` o `admin` y que el perfil exista. Paso 3: actualiza `rol`, `nombre` y `apellido` del perfil. Paso 4: escribe la auditoría `alta_cuenta_interna` con el admin que la hizo |
| Reglas | RF-06, RF-77, RNF-10 |
| Errores | `SOLO_ADMIN`, `ROL_INVALIDO`, `PERFIL_INEXISTENTE` |

### 6.3 Triggers

| Trigger | Tabla y momento | Qué hace | Requerimientos |
|---|---|---|---|
| `auth_usuario_creado` → `crear_perfil()` | `auth.users`, AFTER INSERT | Si `not new.is_anonymous`, inserta `perfiles (id, email)` con `rol = 'cliente'` | RF-01, RF-06, RNF-09 |
| `proteger_fecha_nacimiento` | `perfiles`, BEFORE UPDATE OF `fecha_nacimiento` | Si `old.fecha_nacimiento` tiene valor, el valor nuevo es distinto y `auth.uid()` no es nulo (el cambio llega por la API), lanza `FECHA_NACIMIENTO_INMUTABLE`. Deja hacer la primera carga y las correcciones desde el SQL Editor | RN-06, RN-10, RF-07 |
| `crear_butacas_sala` | `salas`, AFTER INSERT | Copia `plantilla_butacas` a `butacas` para la sala nueva | RF-17 a RF-20, S-01 |
| `preparar_funcion` | `funciones`, BEFORE INSERT OR UPDATE | Calcula `fin` y `ocupacion`; valida que la sala esté activa y soporte el formato | RF-21, RN-01, S-03 |
| `proteger_funcion_vendida` | `funciones`, BEFORE DELETE OR UPDATE OF `inicio`, `sala_id`, `pelicula_id` | Lanza `FUNCION_CON_VENTAS` si hay ocupaciones vendidas | RF-25 |
| `auditar_cambio` (genérico) | `perfiles`, AFTER UPDATE OF `fecha_nacimiento`, `rol`; `funciones`, `precios_formato`, `configuracion`, `productos`, `combos`, `recompensas`, `peliculas`, AFTER INSERT OR UPDATE OR DELETE; `cupones`, AFTER INSERT OR DELETE OR UPDATE OF `codigo`, `tipo`, `porcentaje`, `vigente_desde`, `vigente_hasta`, `limite_usos`, `edad_minima`, `activo` | Inserta en `auditoria` la acción (`alta`, `cambio` o `baja`), la tabla, el id, `to_jsonb(old)` y `to_jsonb(new)`. Actor: `auth.uid()` | RF-83, RF-84, RN-24 |
| `completar_autor_resena` | `resenas`, BEFORE INSERT OR UPDATE | Completa `autor` con el nombre y la inicial del apellido, y `editada_en` | RF-61, RF-63, D-19 |

En `cupones` el trigger es `update of` columnas, para que el `usos + 1` de cada compra no ensucie la auditoría. El uso de cupón lo registra `confirmar_compra` con `accion = 'uso_cupon'`.

Las validaciones, cancelaciones y canjes los registra cada RPC con su acción propia, por ejemplo `validacion_acceso` o `cancelacion`.

**Por qué la auditoría en la base:** El documento lo exige (RN-24) para que no dependa de que el frontend la informe. Los cambios desde el panel de Supabase también quedan registrados.

### 6.4 Tareas programadas (pg_cron)

```sql
select cron.schedule('limpiar-retenciones', '* * * * *',
  $$delete from public.ocupaciones where estado = 'retenida' and vence_en < now()$$);
select cron.schedule('recordatorios', '*/5 * * * *', $$select public.procesar_recordatorios()$$);
select cron.schedule('alertas-venta', '*/10 * * * *', $$select public.procesar_alertas()$$);
```

| Job | Cada | Qué hace | Requerimientos |
|---|---|---|---|
| `limpiar-retenciones` | 1 minuto | Borra las retenciones vencidas; Realtime emite el DELETE y los mapas abiertos liberan la butaca | RN-05, S-17 |
| `recordatorios` | 5 minutos | Llama a `procesar_recordatorios()` (ver abajo) | RF-87, RF-88 |
| `alertas-venta` | 10 minutos | Llama a `procesar_alertas()` (ver abajo) | RF-69, S-10 |

`procesar_recordatorios()`:

- **Paso 1:** inserta `recordatorio_24` para las compras confirmadas de clientes registrados cuya función empieza dentro de las próximas 24 h y en más de 2 h, siempre que la compra se haya hecho antes de `inicio − 24 h`.
- **Paso 2:** inserta `recordatorio_2` para las funciones que empiezan dentro de las próximas 2 h, si la compra se hizo antes de `inicio − 2 h`.
- **Duplicados:** los evita `on conflict (compra_id, tipo) do nothing`.
- **Contenido:** título con la película; cuerpo con sala y hora de Buenos Aires; `url = '/cuenta/compras/{id}'`, que es el acceso directo al QR.

`procesar_alertas()`:

- **Qué marca:** pone `alertas_enviadas_en = now()` en las películas que todavía no avisaron y ya tienen entradas a la venta. Eso es: `proximamente` con preventa abierta, o `en_cartelera` con alguna función futura.
- **Qué inserta:** una notificación `alerta` por cada cliente que tenía la alerta activada.
- **Una sola vez:** `alertas_enviadas_en` garantiza que cada película dispara su alerta una sola vez (S-10).

**Por qué pg_cron:** Un trigger no se dispara por el paso del tiempo (S-17). Para los puntos alcanza con `disponible_desde`, así que esos no necesitan job (S-07).

---

## 7. Supabase

### 7.1 Auth

Proveedores y orden de implementación:

1. Email y contraseña.
2. GitHub.
3. Google.

Además, la sesión anónima para comprar sin cuenta (RF-01, RF-02, RNF-09).

**Por qué este orden:** Email y contraseña no dependen de configuración externa y son las credenciales que necesitan las cuentas internas (RF-06). GitHub ya se hizo en clase. Google es el que más configuración pide.

Flujos, todos en `AuthApi` (sección 8.3):

| Flujo | Llamada | Qué pasa después |
|---|---|---|
| Registro | `Registro` pide email y clave y llama a `supabase.auth.signUp({ email, password })`. Con la sesión que devuelve, lleva a `/cuenta/completar-perfil`, donde `FormularioPerfil` guarda los datos y, si hay consentimiento, el tipo de sangre | Es el mismo paso que hacen las cuentas de GitHub y Google. El trigger ya creó el perfil con rol cliente (RF-01, RF-86) |
| Ingreso con clave | `signInWithPassword({ email, password })` | `SesionStore` carga el perfil y el rol |
| GitHub y Google | `signInWithOAuth({ provider, options: { redirectTo: `${window.location.origin}/` } })` | Al volver, `supabase-js` detecta la sesión en la URL. Si el perfil está incompleto, se ve un aviso y `puedeComprarGuard` lo exige antes de comprar (RF-86) |
| Recuperar clave | `resetPasswordForEmail(email, { redirectTo: `${origin}/restablecer-clave` })` | El enlace abre `/restablecer-clave` con un evento `PASSWORD_RECOVERY`; ahí se llama `updateUser({ password })` (RF-02) |
| Cerrar sesión | `signOut()` | Los stores vuelven a su estado vacío |
| Anónimo | `signInAnonymously()`, la primera vez que un visitante toca una butaca | La retención necesita un `usuario_id` (S-04, S-17) |
| Cuenta interna | `signUp` desde un segundo cliente sin sesión y después `rpc('asignar_rol')` | Sección 7.4 (RF-06) |

Primer administrador:

1. Registrarse con email desde la app.
2. Ejecutar una sola vez en el SQL Editor:

```sql
update public.perfiles set rol = 'admin' where email = '<tu-email>';
```

#### Configuración manual

El identificador del proyecto Supabase es `xrbjzgqosscoowmkwnxf` y la app publicada está en `tp1-sistema-cine.vercel.app`.

| URL | Valor |
|---|---|
| Callback de Supabase | `https://xrbjzgqosscoowmkwnxf.supabase.co/auth/v1/callback` |
| App local | `http://localhost:4200` |
| App publicada | `https://tp1-sistema-cine.vercel.app` |

**A. Supabase: URLs**

1. Abrir Authentication → URL Configuration.
2. En **Site URL**, poner `https://tp1-sistema-cine.vercel.app`.
3. En **Redirect URLs**, agregar:
   - `http://localhost:4200/**`
   - `https://tp1-sistema-cine.vercel.app/**`

**B. Email y contraseña**

1. Abrir Authentication → Sign In / Providers → Email y habilitarlo.
2. Desactivar **Confirm email** (D-16). El SMTP que trae Supabase solo envía a direcciones del equipo del proyecto y tiene un límite bajo por hora, así que un cliente real no podría confirmar su cuenta.
3. Fijar el largo mínimo de contraseña en 8.
4. En Authentication → Emails, traducir al español la plantilla **Reset password** (opcional).
5. La recuperación de contraseña (RF-02) se implementa completa: `RecuperarClave` → mail → `RestablecerClave`. **Limitación:** el SMTP de Supabase solo envía a las direcciones del equipo del proyecto y tiene un límite bajo por hora. En la demo se prueba con una dirección del equipo, y la limitación queda escrita en el README. Para mandarla a cualquier dirección haría falta un SMTP propio, que está fuera de alcance (sección 1.4).

**C. Sesión anónima**

En Authentication → Sign In / Providers, activar **Allow anonymous sign-ins**.

**D. GitHub**

1. En GitHub, ir a Settings → Developer settings → OAuth Apps → **New OAuth App**.
2. Completar:
   - **Application name:** `tp1-sistema-cine`.
   - **Homepage URL:** `https://tp1-sistema-cine.vercel.app`.
   - **Authorization callback URL:** `https://xrbjzgqosscoowmkwnxf.supabase.co/auth/v1/callback`. GitHub admite una sola URL de callback; localhost **no** va acá, porque la redirección a localhost la resuelve Supabase con la lista del paso A.
3. Crear la app, copiar el **Client ID** y generar un **Client secret**.
4. En Supabase, abrir Authentication → Sign In / Providers → GitHub, habilitarlo, pegar el Client ID y el secret, y guardar.

**E. Google**

1. En Google Cloud Console, crear un proyecto `tp1-sistema-cine`.
2. En Google Auth Platform → **Branding**:
   - completar el nombre de la app (`Cineverso`) y el email de soporte;
   - en dominios autorizados, agregar `xrbjzgqosscoowmkwnxf.supabase.co`;
   - **no** subir logo, porque eso obliga a una verificación de marca;
   - en la política de privacidad, cargar `https://tp1-sistema-cine.vercel.app/privacidad`. Sin esa URL, Google no deja pasar la app a producción, y la página tiene que estar publicada.
3. En **Data Access**, agregar solo los scopes `openid`, `.../auth/userinfo.email` y `.../auth/userinfo.profile`.
4. En **Audience**, elegir el tipo **External** y tocar **Publish app** para pasar el estado de *Testing* a *In production*. Así puede entrar cualquier cuenta de Google, incluida la del docente; en *Testing* solo entran los *test users*. Con esos scopes básicos y sin logo, publicar no requiere verificación.
5. En **Clients** → Create client → **Web application**:
   - **Authorized JavaScript origins:** `http://localhost:4200` y `https://tp1-sistema-cine.vercel.app`.
   - **Authorized redirect URIs:** `https://xrbjzgqosscoowmkwnxf.supabase.co/auth/v1/callback`.
6. Copiar el **Client ID** y el **Client secret**.
7. En Supabase, abrir Authentication → Sign In / Providers → Google, habilitarlo, pegar los dos valores y guardar.

**F. Prueba de cada proveedor**

Probar en `http://localhost:4200/login` y en `https://tp1-sistema-cine.vercel.app/login`. Si al volver de GitHub o Google la URL termina en la Site URL en lugar de la de origen, falta el comodín `/**` en las Redirect URLs.

### 7.2 Realtime

Las suscripciones siguen el patrón de clase: un canal sobre la tabla entera, un `switch` sobre `eventType` y `removeChannel` en `ngOnDestroy` (Clase 6, láminas 10 a 13). No se usa `filter`: el callback descarta lo que no corresponde.

| Canal | Tabla | Qué descarta el callback | Quién lo usa | Requerimientos |
|---|---|---|---|---|
| `ocupaciones-cambios` | `ocupaciones` | Los eventos de otras funciones, comparando `funcion_id` (en un DELETE, `old.funcion_id`) | `ButacasStore` | RF-28, RNF-12 |
| `notificaciones-cambios` | `notificaciones` | Nada: el RLS solo deja recibir las notificaciones propias (Clase 6, lámina 12) | `NotificacionesStore` | RNF-12, RF-89 |

- Ambas tablas van a la publicación con `alter publication supabase_realtime add table public.ocupaciones, public.notificaciones;`, como Clase 6, lámina 10.
- Realtime respeta el RLS de quien se suscribe. Con RLS activo, un DELETE trae solo la clave primaria del registro viejo. En `ocupaciones` la clave es `(funcion_id, butaca_id)`, que es justo lo que el mapa necesita para liberar la butaca.
- Sin `filter`, cada mapa abierto recibe los cambios de todas las funciones y descarta los ajenos. Para el volumen de un TP es despreciable.

```ts
// suscribe el mapa a los cambios de ocupación y descarta los de otras funciones
private escucharCambios(): void {
  this.canal = supabase
    .channel('ocupaciones-cambios')
    .on('postgres_changes', { event: '*', schema: 'public', table: 'ocupaciones' }, (cambio) => {
      if (this.esDeOtraFuncion(cambio)) return;
      this.aplicarCambio(cambio);
    })
    .subscribe();
}

// cierra el canal para no acumular sockets
private dejarDeEscuchar(): void {
  if (!this.canal) return;
  void supabase.removeChannel(this.canal);
  this.canal = null;
}
```

`ButacasStore.reiniciar()` llama a `dejarDeEscuchar()`. El `Checkout` llama a `reiniciar()` en su `ngOnDestroy`, que es el punto de limpieza de Clase 6, lámina 13.

### 7.3 Storage

| Bucket | Público | Ruta | Quién escribe | Límites |
|---|---|---|---|---|
| `posters` | sí, lectura | `{pelicula_id}/{nombreSeguro}` | solo el admin | 2 MB; `image/jpeg`, `image/png`, `image/webp` |
| `productos` | sí, lectura | `{producto_id}/{nombreSeguro}` y `combos/{combo_id}/{nombreSeguro}` | solo el admin | igual |

**Los buckets se crean desde el panel**, los dos en F0. Las políticas ya están en `tp1-sistema-cine.sql` y solo comparan el nombre del bucket, así que da igual crearlos antes o después del script.

1. Abrir Storage → **New bucket**.
2. Completar:
   - **Name:** `posters` (o `productos`).
   - **Public bucket:** activado.
   - **Restrict file upload size:** 2 MB.
   - **Allowed MIME types:** `image/jpeg, image/png, image/webp`.
3. Guardar.

Las políticas van en `tp1-sistema-cine.sql` y cubren los dos buckets. La de `select` hace falta porque `upload` con `upsert: true` también lee el objeto:

```sql
create policy "admin lee imagenes" on storage.objects for select to authenticated
  using (bucket_id in ('posters', 'productos') and (select public.es_admin()));
create policy "admin sube imagenes" on storage.objects for insert to authenticated
  with check (bucket_id in ('posters', 'productos') and (select public.es_admin()));
create policy "admin reemplaza imagenes" on storage.objects for update to authenticated
  using (bucket_id in ('posters', 'productos') and (select public.es_admin()));
create policy "admin borra imagenes" on storage.objects for delete to authenticated
  using (bucket_id in ('posters', 'productos') and (select public.es_admin()));
```

- `nombreSeguro()` es la normalización de `attachments.ts:14-19` llevada a `shared/utils/archivos.ts`.
- La URL pública se guarda en `peliculas.poster_url` o en `productos.imagen_url` (RNF-13, RF-08, RF-46).

**Por qué buckets públicos:** Clase 6, lámina 18, da el criterio: público si los archivos pueden verse por URL. Pósters y fotos de productos se muestran a cualquier visitante, no tienen datos personales y solo los sube el admin.

### 7.4 Cuentas internas sin Edge Functions

El proyecto no usa Edge Functions ni la clave `service_role` (D-23). Para RF-06, el admin crea la cuenta con `signUp` desde un **segundo cliente** de Supabase que no guarda sesión, y después le asigna el rol con la RPC `asignar_rol` (sección 6.2.10), que solo puede ejecutar un admin.

```ts
// crea un cliente aparte que no guarda sesión, para dar de alta cuentas sin cerrar la del admin
const clienteAltas = createClient(environment.supabaseUrl, environment.supabaseAnonKey, {
  auth: {
    persistSession: false,
    autoRefreshToken: false,
    detectSessionInUrl: false,
    storageKey: 'cineverso-altas',
  },
});
```

`CuentasAdmin` usa dos métodos de `CuentaInternaApi`:

1. **Paso 1:** `crearCuenta(email, clave)` llama a `clienteAltas.auth.signUp({ email, password })` y crea el usuario. Como cualquier registro, nace con `rol = 'cliente'`. Devuelve `data.user.id`.
2. **Paso 2:** `asignarRol(usuarioId, datos)`, con el cliente principal, que tiene la sesión del admin, llama a `supabase.rpc('asignar_rol', { p_usuario_id, p_rol, p_nombre, p_apellido })`.

Viabilidad:

| Punto | Resultado |
|---|---|
| Sesión del admin | No se toca: el segundo cliente tiene otra `storageKey` y no guarda nada |
| Confirmación de email | Está desactivada (D-16), así que la cuenta queda activa. Aun con confirmación, `signUp` devuelve el id y la RPC funciona igual |
| Escalada de privilegios | No se puede desde el cliente: `signUp` siempre crea un cliente y solo `asignar_rol` cambia el rol, después de verificar que quien llama es admin |
| Si falla el paso 2 | La cuenta queda como cliente. `CuentasAdmin` guarda su id y ofrece **Reintentar el rol**, que vuelve a llamar a `asignarRol` |
| Email repetido | `signUp` devuelve un error y se muestra "Ya existe una cuenta con ese email" |
| `service_role` | No se usa en ningún lado (RNF-10) |
| Límites | Supabase limita los registros por hora e IP. Alcanza para dar de alta unas pocas cuentas internas |

Técnicas: `createClient` con opciones es la misma función de Clase 6, lámina 6, y `signUp` está en Clase 5, lámina 13. La RPC no se vio en clase (sección 13.4).

- **Descartado:** una Edge Function con `auth.admin.createUser`. Necesita la clave secreta y no se vio en clase.
- **Plan B**, si el segundo cliente diera problemas: el empleado se registra solo, como cliente, y el admin lo promueve con `asignar_rol` buscándolo por email. Se pierde la lectura literal de RF-06, "las cuentas las crea un administrador".

### 7.5 Claves

| Clave | Dónde vive | Cómo se carga |
|---|---|---|
| URL del proyecto y clave publicable | `src/environments/*.ts`. Son públicas por diseño: la seguridad la da el RLS | A mano, desde Settings → API del proyecto |
| `service_role` | En ningún archivo del proyecto: no se usa | — |

### 7.6 Aplicación de los scripts SQL

La base se arma pegando scripts en el SQL Editor de Supabase, como en Clase 6, lámina 5. No se usa la CLI de Supabase ni se ejecuta nada contra la base desde la terminal.

| Regla | Detalle |
|---|---|
| Ubicación | `supabase/sql/`, una carpeta local: está en `.git/info/exclude` y no se sube al repo |
| Scripts | `01_base.sql` y `tp1-sistema-cine.sql` (aplicados), y `02_permisos_anon.sql` (sección 5.12) |
| Transacción | Cada script empieza con `begin;` y termina con `commit;`. Si una sentencia falla, no se aplica nada |
| Scripts aplicados | No se editan. Un cambio va en un script nuevo numerado desde `02`, por ejemplo `02_corrige_retenciones.sql` |
| Scripts que fallaron | No quedaron aplicados, así que se corrigen en el mismo archivo y se vuelven a correr |
| Entrega | Con cada script se indica cómo pegarlo y con qué consultas verificarlo |
| Registro | Esta sección y la bitácora de `DEFENSA.md` dicen qué se aplicó. El README no lista los scripts, porque no están en el repo |
| Buckets | `posters` y `productos` se crean desde el panel (sección 7.3). Como las políticas solo comparan el nombre, da igual crearlos antes o después del script |

Procedimiento para cada script:

1. Abrir SQL Editor → **New query**.
2. Pegar el archivo completo y tocar **Run**.
3. Si da error, no quedó nada aplicado: se corrige y se vuelve a correr.
4. Correr las consultas de verificación.
5. Anotar el script como aplicado en la bitácora de `DEFENSA.md`.

Consulta de verificación de `tp1-sistema-cine.sql`. El SQL Editor muestra solo el resultado de la última consulta que devuelve filas, así que todo va en una sola fila:

```sql
-- resume en una fila lo que creó el script
select
  (select count(*) from pg_tables where schemaname = 'public' and rowsecurity) as tablas_con_rls,
  (select count(*) from public.plantilla_butacas) as butacas_plantilla,
  (select string_agg(jobname, ', ' order by jobname) from cron.job) as jobs,
  (select string_agg(tablename, ', ' order by tablename) from pg_publication_tables where pubname = 'supabase_realtime') as realtime,
  (select string_agg(codigo || ' ' || porcentaje, ', ') from public.cupones) as cupones,
  (select count(*) from pg_policies where schemaname = 'storage') as politicas_storage;
```

Resultado esperado, y obtenido el 07/10: 27, 532, `alertas-venta, limpiar-retenciones, recordatorios`, `notificaciones, ocupaciones`, `BIENVENIDA 20.00` y 4.

### 7.7 Acceso para la corrección (D-26)

La cátedra pidió el 09/10 que el login permita entrar como admin sin pedir datos: credenciales visibles, un botón de acceso directo o algo equivalente.

| | |
|---|---|
| Cuenta | `admincine123@gmail.com`, con rol `admin` en `perfiles`. Es exclusiva para la corrección |
| Dónde | `/login`: un botón con estilo de enlace, abajo de todo, que dice "Acceso para la corrección" |
| Qué hace | `usarAccesoCorreccion()` pone el email y la clave en el borrador del formulario (`borrador.set`). No envía nada: se entra con el botón **Ingresar**, que pasa por el mismo `ingresar()` de siempre |
| Riesgo aceptado | La clave queda en el JavaScript publicado en Vercel. Con la URL pública, cualquiera entra como admin |
| Qué lo limita | El RLS y las RPC tratan a esa cuenta como a cualquier admin. No ve claves ni toca Auth de otros usuarios |
| Cómo se restaura | Con el admin privado del autor (no publicado) o desde el panel de Supabase (Authentication → Users), si le cambian la clave |
| Por qué no va al README | El README no lleva claves (D-22). Esta es la única excepción, la pidió la cátedra y vive en el código del login |

---

## 8. Frontend

### 8.1 Configuración

`app.config.ts` parte de C-02 y agrega:

```ts
export const appConfig: ApplicationConfig = {
  providers: [
    provideBrowserGlobalErrorListeners(),
    provideRouter(routes, withComponentInputBinding()),
    { provide: LOCALE_ID, useValue: 'es-AR' },
    { provide: DATE_PIPE_DEFAULT_OPTIONS, useValue: { timezone: '-0300' } },
    provideServiceWorker('ngsw-worker.js', { enabled: !isDevMode(), registrationStrategy: 'registerWhenStable:30000' }),
  ],
};
```

- **Locale:** `registerLocaleData(localeEsAr)` se llama en `main.ts`. `index.html` usa `lang="es"`.
- **Hora de Buenos Aires (RNF-21):** en la vista, el `DatePipe` usa el offset fijo `-0300`, porque Argentina no aplica horario de verano. En las reglas, `fechaBA()` usa `Intl` con `America/Argentina/Buenos_Aires`; en SQL, la misma zona IANA.
- **`tsconfig.json`:** `"strict": true` y `"strictTemplates": true` (RNF-08).
- **`LOCALE_ID` y zona horaria:** no se vieron en clase. Se mantienen por decisión, porque RNF-21 pide la interfaz en español y la hora de Buenos Aires.
- **Environments:** esquema de clase (Clase 6, lámina 20):
  - `environment.ts` es desarrollo y `environment.prod.ts` producción.
  - `angular.json` hace el reemplazo con `fileReplacements` en la configuración de producción.
  - Los dos tienen `production`, `supabaseUrl` y `supabaseAnonKey`.
  - `supabaseAnonKey` guarda la clave publicable (`sb_publishable_…`), que cumple el mismo rol que la anon key de clase (RNF-24).
- **Cliente de Supabase:** `shared/services/supabase.client.ts`, como Clase 6, lámina 6, con `createClient(environment.supabaseUrl, environment.supabaseAnonKey)`.

### 8.2 Modelos

Hay un archivo por entidad en `shared/models/`. Siguen C-09 y usan los mismos nombres de columna que las tablas (RNF-08).

| Archivo | Exporta |
|---|---|
| `perfil.model.ts` | `Rol`, `RolInterno`, `ColorOjos`, `TipoSangre`, `PerfilModel`, `PerfilPatch`, `PerfilDraft`, `DatosSensiblesModel`, `CuentaInternaDraft`, `crearPerfilVacio()`, `crearCuentaInternaVacia()`, `ROL_LABELS`, `COLORES_OJOS`, `COLOR_OJOS_LABELS`, `TIPOS_SANGRE` |
| `auth.model.ts` | `Proveedor`, `IngresoDraft`, `RegistroDraft`, `ClaveNuevaDraft`, `PROVEEDOR_LABELS` |
| `pelicula.model.ts` | `Clasificacion`, `EstadoPelicula`, `GeneroModel`, `GeneroDraft`, `PeliculaModel`, `PeliculaAdmin` (con `generos: GeneroModel[]`), `PeliculaCartelera`, `PeliculaFila`, `PeliculaDraft` (fecha en dd/mm/aaaa y `generos: number[]`), `FiltroCartelera`, `crearPeliculaVacia()`, `CLASIFICACIONES`, `CLASIFICACION_LABELS`, `ESTADOS_PELICULA`, `ESTADO_PELICULA_LABELS` |
| `sala.model.ts` | `SalaModel`, `SalaFila`, `SalaDraft`, `TipoButaca`, `Bloque`, `ButacaModel`, `ButacaCambios`, `FilaButacas<T>`, `crearSalaVacia()`, `TIPOS_BUTACA`, `TIPO_BUTACA_LABELS` |
| `funcion.model.ts` | `Formato`, `Idioma`, `FuncionModel`, `FuncionConSala`, `FuncionAdmin` (con película y sala), `FuncionDraft`, `ProgramacionDraft` (`pelicula_id` como texto, por el select), `PedidoAsignacion`, `FranjaOcupada`, `AsignacionPrevia`, `ResultadoProgramacion`, `crearProgramacionVacia()`, `FORMATOS`, `IDIOMAS`, `IDIOMA_LABELS`. Sin `FORMATO_LABELS`: la etiqueta de cada formato es su valor |
| `configuracion.model.ts` | `ConfiguracionModel`, `PrecioFormatoModel`, `PreciosDraft` |
| `butaca.model.ts` | `EstadoButaca` (`'libre' \| 'seleccionada' \| 'retenida' \| 'vendida' \| 'fuera_de_servicio'`), `OcupacionModel`, `ButacaVista` (`ButacaModel` más `estado` y `precio`), `ButacaSeleccionada` (`id`, `fila`, `numero`, `bloque`, `tipo` y `precio`), `ESTADO_BUTACA_LABELS` |
| `compra.model.ts` | `CompraModel` (fila de la vista `mis_compras`), `EstadoVisibleCompra`, `PasoCompra`, `EntradaModel`, `ItemCompra`, `ResumenCompra`, `SolicitudCompra` (`funcion_id`, `declaracion_edad`, desde F8 `productos` y `combos`, desde F9 `cupon` y desde F10 `credito`), `ResultadoConfirmacion` (unión por `estado`), `ComprobanteModel` (lo que devuelve la RPC `comprobante`), `PASOS_COMPRA`, `PASO_COMPRA_LABELS`, `ESTADO_VISIBLE_COMPRA_LABELS` |
| `candy.model.ts` | `CategoriaModel`, `CategoriaDraft`, `ProductoModel`, `ProductoFila`, `ProductoDraft` (categoría como texto, por el select), `ProductoDeCombo`, `ComboModel` (con `productos: ProductoDeCombo[]`), `ComboFila`, `ComboDraft`, `CantidadElegida` (`id` y `cantidad`, para el pedido y para los productos del combo), `GrupoProductos`, `LineaCandy`, `MAXIMO_POR_PRODUCTO` (10), `crearProductoVacio()`, `crearComboVacio()` |
| `cupon.model.ts` | `TipoCupon`, `CuponModel`, `CuponFila`, `CuponDraft` (fechas en dd/mm/aaaa), `CuponAplicado` (`codigo` y `porcentaje`), `CuponSugerido` (más `descuento`), `ResultadoCupon`, `ContextoCupon`, `TIPOS_CUPON_ADMIN` (general y edad), `TIPO_CUPON_LABELS`, `EDAD_CUPON_POR_DEFECTO` (50), `crearCuponVacio()` |
| `fidelizacion.model.ts` | Desde F10: `TipoMovimientoCredito`, `MovimientoCredito` y `MOVIMIENTO_CREDITO_LABELS`. En F13: `RecompensaModel`, `MovimientoPuntos` y `SaldoPuntos` |
| `resena.model.ts` | `ResenaModel`, `ResenaDraft` |
| `notificacion.model.ts` | `TipoNotificacion`, `NotificacionModel` |
| `validacion.model.ts` | `Sector`, `ButacaValidada`, `ProductoValidado`, `DatosValidacion`, `ResultadoValidacionModel` (unión discriminada por `resultado`; lleva `Model` para no chocar con el componente `ResultadoValidacion`), `TipoResultado`, `SECTORES`, `SECTOR_LABELS`, `RESULTADO_VALIDACION_LABELS` |
| `auditoria.model.ts` | `AccionAuditoria`, `RegistroAuditoria` (con `perfiles` embebido), `FiltroAuditoria`, `CambioAuditoria`, `crearFiltroVacio()`, `ACCIONES_AUDITORIA`, `ACCION_AUDITORIA_LABELS`, `ENTIDAD_AUDITORIA_LABELS` |
| `reporte.model.ts` | `FilaFacturacion`, `TotalFacturacion`, `FilaMasVistas`, `FilaProducto`, `PeriodoReporte`, `RangoFechas` (aaaa-mm-dd), `RangoDraft` (dd/mm/aaaa), `DatoGrafico`, `GrupoMasVistas`, `RangoRapido`, `RANGOS_RAPIDOS`, `RANGO_RAPIDO_LABELS`, `PERIODOS_REPORTE`, `PERIODO_REPORTE_LABELS` |
| `pago.model.ts` | `DatosTarjeta`, `SolicitudPago` (`compra` y `tarjeta`, nula desde F10 si el crédito o el descuento cubren el total), `PagoDraft` (la tarjeta más `declaracion_edad`), `crearPagoVacio()`. No hay `ResultadoPago`: `PagoService` devuelve `ResultadoConfirmacion` |

### 8.3 Servicios

**Servicios de Api** (`shared/api/`). Son delgados y sin estado, como C-07. Cada método hace `if (error) throw error`.

| Servicio | Métodos principales | Requerimientos |
|---|---|---|
| `AuthApi` | `registrar`, `ingresarConClave`, `ingresarConProveedor('github' \| 'google')`, `ingresarAnonimo`, `recuperarClave`, `cambiarClave`, `salir`, `alCambiarSesion(cb)`, `sesionActual` | RF-01, RF-02, RNF-09 |
| `PerfilApi` | `obtener`, `guardar`, `obtenerDatosSensibles`, `guardarDatosSensibles`, `borrarDatosSensibles` | RF-04, RF-86, S-16 |
| `PeliculaApi` | `cartelera`, `proximamente`, `top`, `obtener`, `listarTodas`, `crear`, `actualizar`, `eliminar` | RF-08 a RF-15, RF-68 |
| `GeneroApi` | `listar`, `crear`, `actualizar`, `eliminar` | RF-09 |
| `SalaApi` | `listar`, `obtener`, `crear`, `actualizar`, `eliminar`, `butacas(salaId)`, `actualizarButacas(ids, cambios)` (una llamada por grupo de butacas iguales) | RF-16, RF-20 |
| `FuncionApi` | `dePelicula(peliculaId)` (las que no empezaron), `obtener`, `desde(momento)` (las que no terminaron, para el admin), `enRango(desde, hasta)`, `programar(...)` (rpc), `modificar(...)` (rpc), `eliminar` | RF-21 a RF-25 |
| `OcupacionApi` | `deFuncion`, `retener` (rpc), `liberar` (rpc), `suscribir(funcionId, cb): RealtimeChannel` (descarta en el callback los cambios de otras funciones), `cerrar(canal)` (`removeChannel`) | RF-26 a RF-29 |
| `CompraApi` | `confirmar(solicitud, pago)` (rpc), `misCompras`, `comprobante(compraId)` (rpc, `null` si no es del usuario), `cancelar(compraId)` (rpc, desde F10; devuelve el crédito acreditado) | RF-05, RF-37, RF-73 |
| `CandyApi` | `categorias`, `productos(soloDisponibles)`, `combos(soloActivos)` (con el nombre de cada producto por `combos_productos`) y su ABM: `crear*`, `actualizar*`, `eliminar*` y `guardarProductosCombo(comboId, productos)` (borra los que se quitaron y hace `upsert` de las cantidades) | RF-46 a RF-49 |
| `CuponApi` | `sugeridos(subtotal)` (rpc), `validar(codigo, subtotal)` (rpc) y ABM | RF-40 a RF-43 |
| `FidelizacionApi` | Desde F10: `saldoCredito` (vista `saldos_credito`, 0 sin movimientos) y `movimientosCredito`. En F13: `saldoPuntos`, `movimientosPuntos`, `recompensas` y ABM de recompensas | RF-44, RF-45, RF-65, RF-66, RF-75 |
| `ResenaApi` | `dePelicula`, `guardar`, `misPeliculas` | RF-61 a RF-63, RF-72 |
| `AlertaApi` | `mias`, `activar`, `desactivar` | RF-69 |
| `NotificacionApi` | `listar` (las últimas 30), `marcarLeida(id)`, `marcarTodas()`, `suscribir(usuarioId, cb)` (descarta en el callback las de otro usuario), `cerrar(canal)` | RF-87 a RF-89 |
| `ValidacionApi` | `validar(codigo, sector)` (rpc) | RF-55 a RF-59 |
| `ConfiguracionApi` | `configuracion`, `precios`, `guardarConfiguracion`, `guardarPrecio` | RF-39, RF-41 |
| `ImagenApi` | `subir(bucket, carpeta, archivo): Promise<string>` | RNF-13 |
| `CuentaInternaApi` | `crearCuenta(email, clave)`: `signUp` desde un segundo cliente sin sesión. `asignarRol(usuarioId, datos)`: `rpc('asignar_rol')` (sección 7.4). `listar()`: el personal | RF-06 |
| `ReporteApi` | `facturacion`, `masVistas`, `productos` (rpc) | RF-79 a RF-81 |
| `AuditoriaApi` | `buscar(consulta)`: desde F11, los 200 más recientes con el perfil de quien actuó; el usuario se busca por una parte del email | RF-83, RF-85 |

**Otros servicios** (`shared/services/`):

| Servicio | Qué hace | Por qué |
|---|---|---|
| `PagoService` (`pago-service.ts`) | `pagar(solicitud): Promise<ResultadoConfirmacion>`. Llama a `CompraApi.confirmar` con el número (sin espacios), el vencimiento y el CVV; el titular no viaja, que la RPC valida y simula (S-08) | Es un servicio común, como los de clase. La pasarela queda aislada en un solo archivo: para conectar una real se cambia solo esta clase y el checkout no se toca (RNF-07) |
| `ComprobantesLocales` | `guardar`, `listar` y `obtener` sobre `localStorage` (clave `cine.comprobantes`, máximo 20, el más nuevo primero), con `try/catch` | Comprobantes offline (RNF-15, S-04) |
| `AvisosSistema` | `activar()` pide permiso con `Notification.requestPermission()`, a partir de un gesto del usuario. `mostrar(notificacion)` muestra el aviso del sistema con `registration.showNotification(...)` del service worker mientras la app está abierta (sección 9). Desde F7: `estado` es un signal con el permiso o `no_soportado` (sin `Notification` o sin service worker, como Safari sin instalar), `refrescar()` lo vuelve a leer, y el aviso lleva `tag` con el id para que dos pestañas no lo repitan | RF-87 a RF-89, RNF-16 |

**Errores:** `shared/utils/errores.ts` exporta `mensajeError(error: unknown): string`. Traduce cada código de la base (`BUTACA_NO_DISPONIBLE` → "Otra persona tomó una de las butacas. Elegí otra.") y, si no lo reconoce, usa el criterio de `task-form.ts:42`.

### 8.4 Stores

Siguen C-06: signals públicos de solo lectura, métodos `async` y `computed` para lo derivado (RNF-02).

**`SesionStore`** (raíz)

| | |
|---|---|
| Signals | `sesion: Session \| null`, `perfil: PerfilModel \| null` |
| Computed | `usuarioId`, `esAnonimo`, `registrado`, `rol`, `edad`, `perfilCompleto` |
| Otros | `inicializada: Promise<void>`, `listo()`, `guardarPerfil(borrador)` |
| Requerimientos | RF-04, RF-07, RNF-02 |

El callback de `onAuthStateChange` solo hace `sesion.set(...)`. Un `effect` sobre `usuarioId` carga el perfil y resuelve `inicializada`.

`listo()` espera `inicializada` y, además, el perfil del usuario actual. Los guards y las pantallas de ingreso esperan `listo()` y no solo `inicializada`: justo después de `signInWithPassword` o `signUp`, la sesión ya cambió pero el perfil, que trae el rol, todavía se está cargando. Sin esa espera, un guard leería `rol() === null` y mandaría al usuario a la portada.

**Por qué no se carga el perfil dentro del callback:** La documentación de `supabase-js` advierte que hacer `await` de otra llamada de Supabase ahí puede trabar el cliente.

**`CarteleraStore`** (raíz)

| | |
|---|---|
| Signals | `peliculas`, `top`, `cargando`, `texto`, `generos: string[]`, `formato`, `idioma`, `clasificacion` |
| Computed | `filtradas`, `enCartelera`, `proximamente`, `generosDisponibles` |
| Métodos | `cargar()` |
| Requerimientos | RF-13, RF-14, RF-68 |

Todos los stores son `@Injectable({ providedIn: 'root' })`, como en Clase 5, lámina 5. Los dos del checkout tienen un `reiniciar()` que el `Checkout` llama en su `ngOnDestroy`.

**`ButacasStore`** (raíz, con `reiniciar()`)

| | |
|---|---|
| Signals | `funcion`, `butacas: ButacaModel[]`, `ocupaciones: Map<number, OcupacionModel>`, `venceEn`, `ahora`, `precios` (por tipo de butaca), `diferencias` (desde F8: lo que paga cada tipo de butaca cubierta por un combo, con `diferenciaCombo`), `maxButacas`, `reteniendo` |
| Computed | `funcionId`, `vista: ButacaVista[]` (calcula el estado de cada butaca: libre, seleccionada por mí, retenida por otro, vendida o fuera de servicio, y su precio), `seleccion`, `segundosRestantes` |
| Métodos | `cargar(funcion, porcentajePreventa)`, `alternar(butaca)`, `liberar()`, `vendidas(compraId)` (desde F5: marca las propias como vendidas y apaga el contador sin esperar a Realtime), `reiniciar()` |
| Requerimientos | RF-26 a RF-31, RF-38 |

- `cargar` libera la función anterior si era otra, cierra el canal y abre uno nuevo **antes** de leer las ocupaciones (sección 7.2). Trae las butacas de la sala, la configuración y los precios, y calcula el precio de cada tipo con `precioEntrada`. Si el usuario ya tenía butacas retenidas (por ejemplo, al recargar), retoma su contador.
- `alternar`:
  - abre la sesión anónima si no hay sesión;
  - aplica el cambio de inmediato y llama a `retener` con la selección completa;
  - si falla, deshace solo esa butaca (sin pisar lo que llegó por Realtime), vuelve a leer las ocupaciones por si se perdió un aviso (por ejemplo, tras un corte de red) y lanza el error, que muestra `PasoButacas`;
  - ignora los toques mientras `reteniendo()`: el mapa queda bloqueado para que dos pedidos no lleguen en otro orden;
  - si ya hay `maxButacas` elegidas, lanza "Podés elegir hasta 10 butacas por compra." sin llamar a la base.
- **Reloj:** `vence_en` lo pone la base con su hora, y el reloj de la PC puede estar corrido (en la prueba del 07/10 estaba 2:44 adelantado y el contador arrancaba en 07:16). Cuando `retener` arranca un vencimiento nuevo, el store mide `desfaseReloj(venceEn, minutos_retencion, new Date())` y `segundosRestantes` cuenta con `ahora + desfase`. Al recargar con butacas ya retenidas no hay cómo medirlo y se usa el último desfase conocido (0 en una carga nueva).
- **Mensajes en `PasoButacas`:** el motivo de contigüidad o de máximo usa `.aviso` (ámbar, `role="status"`), porque no es un error sino algo a corregir; el rechazo de la base y el vencimiento de la retención usan `.error` (`role="alert"`). "Queda menos de un minuto" también es `.aviso`.
- El estado de una retención ajena vencida lo actualiza el job `limpiar-retenciones` (como mucho un minuto después) con su `DELETE`.
- Un `setInterval` actualiza `ahora` mientras hay retención; es el "signal con intervalo" de Clase 8, lámina 12.
- `reiniciar()`:
  - libera las retenciones;
  - cierra el canal con `removeChannel`;
  - corta el intervalo;
  - vacía los signals.

**`CompraStore`** (raíz, con `reiniciar()`)

| | |
|---|---|
| Signals | Desde F5: `pelicula`, `funciones`, `funcion`, `paso`, `resultado`, `comprobante`, `error`, `cargando`, `pagando`. Desde F8: `categorias`, `productos`, `combos` (el catálogo a la venta), `productosElegidos`, `combosElegidos` y `cargandoCandy`. Desde F9: `cupon`, `errorCupon` y `validandoCupon`. Desde F10: `saldoCredito` y `credito` (el monto elegido). Se suma `recompensas` en F13 |
| Computed | `resumen` (con `calcularResumen()`), `pideDeclaracion` (anónimo y película +13 o +18). Desde F8: `entradas` (con `entradasConCombos`), `lineasProductos`, `lineasCombos` y `combosDeMas`. Desde F9: `subtotal` y `cuponSugerido`. Desde F10: `creditoDisponible` (el saldo, sin pasarse del total) |
| Métodos | Desde F5: `cargar(peliculaId, funcionId)`, `elegirFuncion`, `irAPaso`, `limpiarError`, `vencer`, `confirmar(tarjeta, declaracionEdad)`, `cargarComprobante`, `reiniciar()`. Desde F8: `cargarCandy()` (una sola vez por compra), `agregarProducto`, `quitarProducto`, `agregarCombo` (solo si queda una entrada sin combo) y `quitarCombo`. Desde F9: `cargarCupones()` (una sola vez por compra), `aplicarCupon(codigo)` y `quitarCupon()`. Desde F10: `cargarCredito()` (una sola vez por compra), `usarCredito(monto)` y `quitarCredito()`; `confirmar` acepta `tarjeta` nula |
| Requerimientos | RF-32, RF-36, RF-43 a RF-45, RF-47 |

`reiniciar()` vuelve todos los signals al estado inicial, para que la próxima compra arranque vacía.

- `cargar` trae la película y sus funciones a la venta. Si la ruta trae un `funcionId` de esa lista, llama a `elegirFuncion` y pasa a las butacas; si no, queda en el paso de función.
- `elegirFuncion` repite `motivoRestriccionEdad` (por si la función llegó desde un enlace) y llama a `ButacasStore.cargar`.
- `vencer` lo llama el `ContadorRetencion` del checkout, que está arriba de los pasos de butacas, candy y pago: libera, vuelve a las butacas y deja el mensaje en `error`.
- `confirmar`: si el pago no se aprueba, la base ya liberó las butacas; el store libera las locales, muestra el motivo y vuelve a las butacas. Si se aprueba, llama a `ButacasStore.vendidas(compraId)`, pasa al comprobante **antes** de traerlo (así un error al traerlo no invita a pagar otra vez), lo trae con `cargarComprobante` y lo guarda en `ComprobantesLocales`.
- `error` es el mensaje del checkout que se muestra arriba del paso (`.error`, `role="alert"`): vencimiento, pago rechazado o edad.

**`NotificacionesStore`** (raíz)

| | |
|---|---|
| Signals | `notificaciones` |
| Computed | `noLeidas` |
| Métodos | `marcarLeida(id)`, `marcarTodas()` |
| Requerimientos | RF-69, RF-87 a RF-89, RNF-12 |

Un `effect` sobre el usuario abre el canal cuando hay un usuario registrado y lo cierra con `removeChannel` cuando cambia o se va, igual que `TaskStore` en clase (C-06). Cuando llega un INSERT y el usuario dio permiso, pide a `AvisosSistema` que muestre el aviso del sistema (sección 9).

Desde F6: abre el canal **antes** de traer la lista, como `ButacasStore`, para no perder un aviso que llegue en el medio. Un INSERT agrega arriba, un UPDATE reemplaza (por ejemplo, leída en otra pestaña) y un DELETE quita. Si falla la lista, la bandeja queda vacía y se completa con lo que llegue. El aviso del sistema entra en F7.

Los datos de la tarjeta **no** pasan por `CompraStore`: viven en el signal del formulario de `PasoPago` y se descartan después del envío (S-08).

### 8.5 Reglas puras y uso de `@switch`

Las reglas están en `shared/rules/`. Son funciones sin Angular y cada archivo tiene su `*.spec.ts`.

| Archivo | Funciones | Uso en la interfaz | Requerimientos |
|---|---|---|---|
| `fechas.ts` | `fechaBA(d)`, `proximosDias(hoy, 9)`, `fechasRecurrentes(dias, desde, semanas)` (días con 0 = domingo), `inicioBA(fecha, hora)` (con `-03:00` fijo: Argentina no tiene horario de verano), `horaBA(momento)`, `etiquetaDia(fecha, hoy)` ("Hoy", "Mañana" o "Vie 16/10"), `agruparPorDia(funciones)`, `sumarDias`, `diaSemana`, `PATRON_HORA`, `desfaseReloj(venceEn, minutosRetencion, ahora)` (cuánto adelanta el servidor al navegador), `fechaDesdeTexto('dd/mm/aaaa')`, `textoDesdeFecha('aaaa-mm-dd')` | Chips de días, programación, ficha, fecha de nacimiento | RF-22, RF-34, RNF-19, RNF-21 |
| `formato.ts` | `formatearDuracion(minutos)`, `formatearPrecio(monto)`, `formatearCuentaRegresiva(segundos)` ("09:05"), `formatearCodigo(codigo)` ("ABCD-EFGH-JK"), `normalizarCodigo(texto)` (mayúsculas, sin guiones ni espacios) y `LARGO_CODIGO` | Pipes `duracion` y `precioArs`, `ContadorRetencion` | RF-15, RF-31, RF-36, RNF-21, RNF-26 |
| `edad.ts` | `calcularEdad(nacimiento, aFecha)`, `edadMinima(clasificacion)`, `motivoRestriccionEdad(clasificacion, registrado, nacimiento, fechaFuncion)`, `leyendaClasificacion(clasificacion)` (leyenda del comprobante para +13 y +18) | Pipe `edad`, directiva `appRestrictEdad`, comprobante | RF-07, RF-35, RN-06 |
| `pelicula.ts` | `borradorDesdePelicula(pelicula)`, `filaDesdeBorrador(borrador)`, `filtrarCartelera(peliculas, filtro)` (texto sin tildes, alguno de los géneros y clasificación), `ordenarPorEstreno(peliculas, masNuevasPrimero)` | `PeliculaForm` y `CarteleraStore` | RF-08, RF-13, RF-14, RF-68 |
| `perfil.ts` | `perfilCompleto(perfil)`, `borradorDesdePerfil(perfil, sensibles)`, `cambiosDesdeBorrador(borrador, incluirFecha)`, `motivoFechaNacimiento(texto, hoy)` | `SesionStore`, `FormularioPerfil`, `Perfil` y `CompletarPerfil` | RF-01, RF-04, RF-86, RN-06 |
| `precio.ts` | `precioEntrada(base, tipo, recargoVip, porcentajePreventa)` (desde F4), `enPreventa(estreno, dias, activa, momento)`, `diferenciaCombo(baseFormato, base2D, tipo, recargoVip)`, `entradasConCombos(butacas, combos, diferencias)` (desde F8: los combos cubren primero las no VIP, como la base), `calcularResumen({ entradas, candy, combos, porcentajeCupon, credito }): ResumenCompra` | Precio VIP al tocar la butaca y resumen | RF-31, RF-36, RN-07, RN-08, RN-11, RN-12, RN-15 |
| `candy.ts` | `agruparPorCategoria(productos, categorias)`, `detalleCombo(combo)` ("1 entrada 2D + 1 × Pochoclo grande"), `cantidadDe`, `totalCantidad`, `cambiarCantidad(elegidos, id, cambio, maximo)`, `lineasPedido(elegidos, catalogo)`, `subtotalLineas` | `PasoCandy`, `CompraStore`, `SelectorProductos`, `ProductosAdmin` y `CombosAdmin` | RF-46 a RF-49 |
| `cupones.ts` | `estadoCupon(cupon, hoy)` (activo, vigencia y usos, para la lista del admin), `motivoRechazoCupon(cupon, contexto)` (repite `motivo_cupon`), `mejorCupon(cupones, subtotal)`, `normalizarCodigoCupon`, `textoVigencia(desde, hasta)`, `borradorDesdeCupon`, `cuponDesdeBorrador` | Sugerencia en `PasoPago`, `CuponesAdmin` | RF-43, RN-09 a RN-11 |
| `cancelacion.ts` | `limiteCancelacion(inicio, horasLimite)`, `motivoNoCancelable(compra, horasLimite, ahora)` (repite las condiciones de `cancelar_compra`) | Botón **Cancelar** de `DetalleCompra` | RF-73, RN-18 |
| `reportes.ts` | `motivoFechaRango(texto)`, `motivoOrdenRango(borrador)`, `rangoDesdeBorrador(borrador)`, `rangoRapido(tipo, ahora)` (hoy, últimos 7 días o el mes, en días de Buenos Aires), `totalFacturacion(filas)`, `etiquetaPeriodo(periodo, tipo)` ("Semana del 05/10/2026" u "Octubre de 2026"), `agruparMasVistas(filas, tipo)` | `Reportes`, `Auditoria`, `reporte-pdf.ts` y `excel.ts` | RF-79 a RF-82 |
| `auditoria.ts` | `textoValor(valor)`, `cambiosAuditoria(registro)` (en un cambio, solo los campos distintos), `referenciaEntidad(entidad, id)` ("Compra 3f2a9c1b"), `nombreActor(perfil)` ("Sistema" sin usuario), `momentosDelRango(rango)` (desde incluido y hasta excluido, en hora de Buenos Aires) | `Auditoria` | RF-83, RF-85 |
| `puntos.ts` | `puntosGanados(totalTrasCupon)`, `saldoPuntos(movimientos, ahora)` | "Puntos a ganar" y perfil | RF-36, RF-64, RF-66, RN-12, RN-19 |
| `asignacion.ts` | `ocupacionHasta(inicio, duracion, limpieza)`, `asignarSalas(pedido, salas, ocupadas, limpieza): AsignacionPrevia[]` | Vista previa de la programación | RF-23, RF-24, RN-01, RN-03 |
| `sala.ts` | `etiquetaSala(sala)` ("Sala 3 · Premium"), `agruparButacas(butacas)` (por fila, en los tres bloques; genérica, para reusarla en `MapaButacas`) | `SalasAdmin`, `SalaButacas`, funciones | RF-16, RF-17, RF-20 |
| `contiguidad.ts` | `motivoNoContiguas(butacas): string \| null` | Validador `butacasContiguas` | RF-90, RN-25 |
| `tarjeta.ts` | `PATRON_VENCIMIENTO`, `PATRON_CVV`, `digitosTarjeta(numero)`, `luhnValido(numero)`, `vencimientoValido('MM/AA', hoy)`, `cvvValido` | Formulario de pago | S-08 |

`@switch` se usa en (RNF-04):

| Dónde | Sobre qué | Requerimientos |
|---|---|---|
| `Checkout` | El paso actual | RF-32 |
| `ResultadoValidacion` | `resultado.resultado` | RF-58 |
| `MapaButacas` | El ícono de cada estado, como apoyo al color | RF-27, RNF-20 |
| `MisCompras` | El estado visible | RF-05 |

### 8.6 Componentes reutilizables y guards

Componentes en `shared/components/`, comunicados con signal inputs y outputs, como C-04 y C-05 (RNF-03):

| Componente | Inputs | Outputs o model | Requerimientos |
|---|---|---|---|
| `TarjetaPelicula` (`app-tarjeta-pelicula`) | `pelicula = input.required<PeliculaCartelera>()`, `puesto = input<number \| null>(null)` (en el Top 3 de la portada: muestra un listón "TOP n" en la esquina superior derecha del póster, con `clip-path` y el corte en V abajo), `conAlerta = input(false)` | `alertaCambiada = output<boolean>()` | RF-13, RF-14, RF-68, RF-69 |
| `MapaButacas` (`app-mapa-butacas`) | `butacas = input.required<ButacaVista[]>()`, `bloqueado = input(false)` | `alternada = output<ButacaVista>()` | RF-26 a RF-31 |
| `SelectorFecha` (`app-selector-fecha`) | `dias = input.required<string[]>()` | `value = model<string \| null>(null)` (compatible con `[formField]`) | RF-34, RNF-19 |
| `SelectorHorario` (`app-selector-horario`) | `funciones = input.required<FuncionConSala[]>()`, `elegidaId = input<number \| null>(null)` | `elegida = output<FuncionConSala>()` | RF-34 |
| `SelectorDias` (`app-selector-dias`) | — | `value = model<number[]>([])` (chips Lu a Do) | RF-22, RF-78 |
| `SelectorHora` (`app-selector-hora`) | `sugeridas = input<string[]>(['14:00', '16:30', '19:00', '21:30'])` | `value = model<string>('')` (botones y campo `HH:MM`) | RF-78 |
| `ContadorRetencion` (`app-contador-retencion`) | `segundos = input.required<number>()` | `vencido = output<void>()` | RF-29 |
| `IndicadorPasos` (`app-indicador-pasos`) | `pasos = input.required<string[]>()`, `actual = input.required<number>()` | — | RF-32 |
| `ResumenCompra` (`app-resumen-compra`) | `resumen = input.required<ResumenCompra>()` | — | RF-31, RF-36 |
| `VistaComprobante` (`app-vista-comprobante`) | `comprobante = input.required<ComprobanteModel>()` | — (el botón **Descargar PDF** carga `comprobante-pdf` por su cuenta) | RF-51 a RF-54 |
| `Estrellas` (`app-estrellas`) | `soloLectura = input(false)` | `value = model<number>(0)` | RF-61, RF-62 |
| `GraficoBarras` (`app-grafico-barras`) | `datos = input.required<DatoGrafico[]>()` (`{ etiqueta, valor }`), `formato = input<'numero' \| 'precio'>('numero')`, `titulo = input.required<string>()` (desde F11: el `caption` de la tabla para lectores de pantalla) | — | RF-80, RF-81 |
| `FormularioPerfil` (`app-formulario-perfil`) | `fechaFija = input(false)`: si la fecha de nacimiento ya está cargada, se muestra como texto. `conDatosSensibles = input(true)`, `textoBoton = input('Guardar')`, `guardando = input(false)` | `perfil = model<PerfilDraft>()`, que la pantalla enlaza con `[(perfil)]`, y `guardado = output<PerfilDraft>()`. La pantalla compara su borrador con el cargado para `cambiosPendientes()` | RF-01, RF-04, RF-86, RN-06 |
| `BandejaNotificaciones` (`app-bandeja-notificaciones`) | — (usa `NotificacionesStore`). Va en la cabecera del cliente: un botón "Avisos" con la cantidad sin leer que despliega la lista; cada aviso es un enlace a su `url` y se marca leído al tocarlo. Cierra con Escape | — | RF-89 |
| `VerClave` (`app-ver-clave`) | `visible = model(false)` (la pantalla lo usa en `[type]="verClave() ? 'text' : 'password'"`), `nombre = input('la clave')` para el texto accesible. Va dentro de `.con-ver-clave`, a la derecha del campo. Se usa en el login, el registro, la clave nueva, la cuenta interna y el código de seguridad del pago. Signal Forms lee `element.type` en cada lectura, y con `text` o `password` devuelve el mismo `value` | — | Pedido del usuario el 09/10 |
| `MenuPrincipal` (`app-menu-principal`) | — (usa `SesionStore`; los grupos salen de `menu.model.ts`: `ENLACES_PUBLICOS`, `GRUPOS_ADMIN` con Cine, Ventas, Gestión y Reportes, y `ENLACES_CUENTA_CLIENTE`). Computadora: Cartelera y Próximamente sueltos, los grupos con desplegable al pasar el mouse o con clic (`aria-expanded`, Escape, cierre al sacar el mouse o el foco) y "Mi cuenta" a la derecha con nombre, rol (`ROL_LABELS`) y Salir. Celular: botón de menú que abre un panel lateral con "Mi cuenta" primero y los grupos como acordeón (`grid-template-rows` de `0fr` a `1fr`). Animaciones con `transition`, sin `@angular/animations`, y sin movimiento con `prefers-reduced-motion`. El presupuesto de estilos por componente sube a 6 kB por este archivo | `salir = output<void>()` | Pedido del usuario el 09/10 |

El aviso de versión nueva no es un componente: va en `app.ts` con `SwUpdate`, un signal y un `@if` en `app.html`, como Clase 8, lámina 23 (sección 9).

Detalles:

- **`MapaButacas`** dibuja la pantalla y las filas A a T con sus tres bloques y los pasillos, usando `@for` anidados. Cada butaca es un `<button>` con `appHighlightButaca`. La leyenda está siempre visible (RF-30). Con el foco o el puntero sobre una VIP muestra su precio diferencial (RF-31). En el celular el mapa se desplaza horizontalmente dentro de su contenedor, sin desplazar la página.
- **`GraficoBarras`** dibuja cada barra con `@for` y un `width` en porcentaje del máximo. Desde F11, cada barra crece desde 0 hasta su ancho con un `@keyframes` de CSS (800 ms, una tras otra con `animation-delay` de 80 ms por `$index`), y una `transition` anima los cambios de valor; con `prefers-reduced-motion` aparecen ya completas. Debajo muestra la misma información en una tabla, para lectores de pantalla (RNF-20).

**Por qué `GraficoBarras` sin librería:** Son dos gráficos de barras. Una librería agregaría peso y una API nueva que defender, a cambio de nada que no se resuelva con CSS y `@for`.

Componentes de un área, que no van en `shared/`:

| Área | Componente | Inputs y outputs |
|---|---|---|
| `admin/` | `SelectorGeneros` (`app-selector-generos`) | `generos = input.required<GeneroModel[]>()`, `value = model<number[]>([])` (chips que se prenden y apagan, como `PriorityPicker` de clase, C-05) |
| `admin/` | `SelectorFormatos` (`app-selector-formatos`) | `value = model<Formato[]>([])` (chips 2D a 5D, siempre en ese orden), para los formatos de la sala |
| `admin/` | `SelectorProductos` (`app-selector-productos`) | `productos = input.required<ProductoModel[]>()`, `value = model<CantidadElegida[]>([])` (desde F8: botones − y + por producto, hasta 10), para los productos de cada combo |
| `empleado/` | `EscanerQr` (`app-escaner-qr`) | `activo = input(true)`, `leido = output<string>()`; usa `@zxing/browser`. La cámara se enciende con un botón (`decodeFromVideoDevice` con la cámara trasera) y se apaga con otro y en `ngOnDestroy`. Al video se le pasa su `id`, así no hace falta `viewChild`. Con `activo()` en falso sigue filmando pero no emite |
| `empleado/` | `ResultadoValidacion` (`app-resultado-validacion`) | `resultado = input.required<ResultadoValidacionModel>()` |

Guards y resolver en `shared/guard/`, uno por archivo con el nombre del CLI. Siguen Clase 7, láminas 5 a 7, y Clase 8, láminas 5 a 7:

| Archivo | Exporta | Tipo |
|---|---|---|
| `invitado-guard.ts` | `invitadoGuard` | `CanActivateFn` |
| `cliente-guard.ts` | `clienteGuard` | `CanMatchFn` |
| `empleado-guard.ts` | `empleadoGuard` | `CanMatchFn` |
| `admin-guard.ts` | `adminGuard` | `CanMatchFn` |
| `puede-comprar-guard.ts` | `puedeComprarGuard` | `CanActivateFn` |
| `pending-changes-guard.ts` | `pendingChangesGuard` y la interfaz `ConCambiosPendientes` | `CanDeactivateFn` |
| `pelicula-resolver.ts` | `peliculaResolver` | `ResolveFn` |

```ts
// deja entrar solo al admin y redirige al resto
export const adminGuard: CanMatchFn = async (_, segmentos) => {
  const sesion = inject(SesionStore);
  const router = inject(Router);
  await sesion.listo();
  const volver = '/' + segmentos.map((segmento) => segmento.path).join('/');
  if (!sesion.registrado()) return router.parseUrl(`/login?volver=${encodeURIComponent(volver)}`);
  return sesion.rol() === 'admin' ? true : router.parseUrl('/');
};
```

```ts
export interface ConCambiosPendientes {
  cambiosPendientes(): boolean;
  mensajeSalida(): string;
}

// pregunta antes de salir de una pantalla con cambios sin guardar o butacas retenidas
export const pendingChangesGuard: CanDeactivateFn<ConCambiosPendientes> = (pantalla) =>
  pantalla.cambiosPendientes() ? confirm(pantalla.mensajeSalida()) : true;
```

```ts
// trae la película antes de mostrar la ficha
export const peliculaResolver: ResolveFn<PeliculaCartelera | null> = (route) =>
  inject(PeliculaApi).obtener(Number(route.paramMap.get('peliculaId')));
```

- `clienteGuard` y `empleadoGuard` son iguales a `adminGuard`, con su rol. `empleadoGuard` acepta `empleado` y `admin`.
- Los formularios implementan `cambiosPendientes()` con el estado `dirty()` del formulario, y `mensajeSalida()` devuelve "Hay cambios sin guardar. ¿Salir igual?".
- El `Checkout` implementa `cambiosPendientes()` como "hay butacas retenidas". Si el usuario confirma la salida, su `ngOnDestroy` llama a `reiniciar()` y las butacas se liberan.

### 8.7 Pipes

Son pipes puros y standalone, en `shared/pipes/` (RNF-26).

| Pipe | Firma | Comportamiento | Requerimientos |
|---|---|---|---|
| `duracion` | `transform(minutos: number \| null \| undefined): string` | 135 → `2 h 15 min` · 120 → `2 h` · 45 → `45 min` · 0 → `0 min` · `null` o negativo → `''` | RF-15 |
| `precioArs` | `transform(monto: number \| null \| undefined): string` | 12500 → `$ 12.500,00`, con `Intl.NumberFormat('es-AR', { style: 'currency', currency: 'ARS' })` creado una sola vez. El espacio después de `$` es uno de no separación (` `), para que no se corte la línea · `null` → `''` | RF-31, RF-36, RNF-21 |
| `edad` | `transform(fechaNacimiento: string \| null \| undefined, aFecha: Date = new Date()): number \| null` | Años cumplidos con `calcularEdad()`. `'2008-10-07'` a la fecha `2026-10-06` → 17 | RF-04, RF-07 |

La lógica de cada pipe está en una regla pura de `shared/rules/`, y el pipe solo la llama: `formatearDuracion` y `formatearPrecio` en `formato.ts`, y `calcularEdad` en `edad.ts`. Así se prueba sin TestBed (sección 10), y se respeta el criterio de Clase 7, lámina 9: el componente piensa, la pipe formatea.

```ts
@Pipe({ name: 'duracion' })
export class DuracionPipe implements PipeTransform {
  // convierte minutos al formato "2 h 15 min" con la regla pura
  transform(minutos: number | null | undefined): string {
    return formatearDuracion(minutos);
  }
}
```

**Por qué pipes propios y no `CurrencyPipe` o `DatePipe`:** La cátedra los exige (RNF-26). Además `precioArs` no depende de registrar el locale, y `edad` reutiliza la misma regla que la directiva y la base.

### 8.8 Directivas

Las dos directivas están en `shared/directives/` (RNF-27). `appHighlightButaca` es de atributo, con la propiedad `host` y signal inputs (Clase 7, lámina 14). `appRestrictEdad` es estructural (Clase 7, lámina 15).

**`appHighlightButaca`** (RF-27, RF-30, RNF-20)

| | |
|---|---|
| Firma | `butaca = input.required<ButacaVista>({ alias: 'appHighlightButaca' })` |
| Uso | `<button [appHighlightButaca]="b" (click)="alternada.emit(b)">` |
| Clases | Agrega `butaca` y una por tipo y estado: `butaca--vip`, `butaca--accesible`, `butaca--seleccionada`, `butaca--retenida`, `butaca--vendida`, `butaca--fuera` |
| Accesibilidad | Pone `aria-label` (por ejemplo "Fila J, butaca 3, accesible, libre"), `aria-pressed` cuando está seleccionada y `aria-disabled` cuando no está disponible |

La forma y el ícono van en el CSS de `MapaButacas`:

- Accesible: más ancha y redondeada.
- VIP: estrella.
- Vendida: cruz.
- Retenida: rayado.

Así el mapa **no depende solo del color**.

**`appRestrictEdad`** (RF-35, RN-06, S-04)

Es una **directiva estructural**, como `*appIfOverdue` de Clase 7, lámina 15, y Clase 8, lámina 17. Decide si el bloque de compra existe. El motivo del bloqueo no lo muestra la directiva sino el componente, con `@if` y la misma regla.

| | |
|---|---|
| Firma | `appRestrictEdad = input.required<RestriccionEdad>()`, con `RestriccionEdad = { clasificacion: Clasificacion; fechaFuncion: Date \| null }` (en `pelicula.model.ts`; `null` mide la edad a hoy). El input se llama igual que el selector (Clase 8, lámina 27) |
| Inyecta | `TemplateRef`, `ViewContainerRef` y `SesionStore` |
| Lógica | Un `effect` calcula `motivoRestriccionEdad(...)`. Si no hay motivo y la vista no existe, la crea con `createEmbeddedView`; si hay motivo y existe, la quita con `clear()`. Un booleano `visible` evita recrear el bloque cada vez que cambia el input |
| Uso | Sobre el bloque **Comprar** de la ficha y del paso de función |

Casos de `motivoRestriccionEdad`, en `shared/rules/edad.ts`:

| Caso | Resultado |
|---|---|
| Película ATP o visitante anónimo | Sin motivo. El anónimo declara su edad en el paso de pago (S-04) |
| Cliente sin fecha de nacimiento | "Completá tu fecha de nacimiento para comprar entradas +13 o +18" |
| Cliente menor | "Película +18: necesitás tener 18 años cumplidos el día de la función" |

```ts
@Directive({ selector: '[appRestrictEdad]' })
export class RestrictEdad {
  private readonly tpl = inject(TemplateRef);
  private readonly vcr = inject(ViewContainerRef);
  private readonly sesion = inject(SesionStore);
  readonly appRestrictEdad = input.required<RestriccionEdad>();

  constructor() {
    // muestra el bloque de compra solo si la edad del cliente alcanza
    effect(() => {
      const { clasificacion, fechaFuncion } = this.appRestrictEdad();
      const motivo = motivoRestriccionEdad(
        clasificacion,
        this.sesion.registrado(),
        this.sesion.perfil()?.fecha_nacimiento ?? null,
        fechaFuncion,
      );
      this.vcr.clear();
      if (motivo === null) this.vcr.createEmbeddedView(this.tpl);
    });
  }
}
```

En la plantilla del componente:

```html
<!-- muestra la compra si la edad alcanza y el motivo si no -->
<button
  *appRestrictEdad="{ clasificacion: pelicula().clasificacion, fechaFuncion: null }"
  type="button"
  (click)="comprar()"
>
  Comprar
</button>
@if (motivoEdad(); as motivo) {
  <p class="motivo" role="alert">{{ motivo }}</p>
}
```

`motivoEdad` es un `computed` del componente que llama a la misma regla. La directiva y el mensaje no se pueden contradecir, porque los dos usan `motivoRestriccionEdad`.

### 8.9 Validador `butacasContiguas`

Está en `shared/validators/butacas-contiguas.ts` (RNF-28, RF-90, RN-25). Es una regla de Signal Forms, de la misma familia que `required` o `minLength`, construida con `validate()`. La API se verificó contra `@angular/forms/signals` 22.1.

```ts
import { SchemaPath, validate } from '@angular/forms/signals';
import { ButacaSeleccionada } from '../models/butaca.model';
import { motivoNoContiguas } from '../rules/contiguidad';

// valida que las butacas elegidas sean contiguas y explica el motivo si no lo son
export function butacasContiguas(campo: SchemaPath<ButacaSeleccionada[]>): void {
  validate(campo, ({ value }) => {
    const motivo = motivoNoContiguas(value());
    return motivo ? { kind: 'butacasContiguas', message: motivo } : null;
  });
}
```

`motivoNoContiguas(butacas)` devuelve `null` si hay 0 o 1 butacas. Si no, devuelve el primer motivo que encuentra:

| Motivo | Mensaje |
|---|---|
| Hay más de una fila | "Las butacas tienen que estar en la misma fila" |
| Hay más de un bloque | "Un pasillo separa las butacas elegidas" |
| Los números no son consecutivos | "Hay lugares sin elegir entre J5 y J8" (no dice "libres": el lugar del medio puede estar vendido) |

En `PasoButacas`:

```ts
protected readonly seleccion = signal<{ butacas: ButacaSeleccionada[] }>({ butacas: [] });

constructor() {
  // copia la selección del store al modelo del formulario cada vez que cambia
  effect(() => this.seleccion.set({ butacas: this.butacasStore.seleccion() }));
}

protected readonly seleccionForm = form(this.seleccion, (f) => {
  validate(f.butacas, ({ value }) => (value().length === 0 ? { kind: 'requerido', message: 'Elegí al menos una butaca' } : null));
  maxLength(f.butacas, () => this.butacasStore.maxButacas(), {
    message: () => `Podés elegir hasta ${this.butacasStore.maxButacas()} butacas`,
  });
  butacasContiguas(f.butacas);
});
```

- El modelo es un `signal` que un `effect` sincroniza con el store, como el `effect` + `draft.set()` de la edición en Clase 4, lámina 25.
- El botón **Continuar** queda deshabilitado mientras `!seleccionForm.butacas().valid()`.
- El motivo se muestra en un `role="alert"` apenas cambia la selección, sin esperar `touched` (RF-90).
- `confirmar_compra` repite la validación en la base (RN-25).

### 8.10 Formularios con Signal Forms

Todos siguen C-12 y Clase 4, láminas 20 a 25 (RNF-05):

- el borrador es un `signal`, y `form(this.draft, validadores, { submission: { action } })`;
- `[formRoot]` en el `<form>`;
- los errores se muestran con `touched() && !valid()`.

La Clase 4, lámina 24, muestra `onInvalid: 'touch'` para marcar todos los campos al enviar un formulario inválido, así el usuario ve de una vez todo lo que falta. En Angular 22.2 eso ya lo hace `submit` antes de validar, y `onInvalid` pasó a ser una función opcional: escrito como en la lámina no compila con `strict`. El comportamiento es el mismo sin escribir nada. Signal Forms también pone solo los atributos `min` y `max` del input a partir de los validadores, así que no se escriben a mano. Los formularios de edición precargan el borrador con un `effect` y `draft.set()` (Clase 4, lámina 25).

| Formulario | Validaciones | Requerimientos |
|---|---|---|
| `Registro` | `email` (`required`, `email`); `clave` (`required`, `minLength(8)`); repetir clave (`validate` contra la otra). Los datos del perfil se cargan en el paso siguiente, `CompletarPerfil` | RF-01 |
| `Login`, `RecuperarClave`, `RestablecerClave` | `email` (`required`, `email`); clave nueva (`required`, `minLength(8)`) y repetir clave (`validate` contra la otra) | RF-02 |
| `FormularioPerfil` (en `CompletarPerfil` y `Perfil`) | `nombre` y `apellido` (`required`, `maxLength(60)`); `fecha_nacimiento` (`required`, `pattern` dd/mm/aaaa y `validate` con `motivoFechaNacimiento`: fecha real, no futura y posterior a 1900); `color_ojos` (`required`); `dias_vacaciones` (`required`, `min(0)`, `max(365)`); `tipo_sangre` opcional; `consentimiento` obligatorio solo si hay tipo de sangre (`validate`) | RF-01, RF-04, RF-86, S-16 |
| `CompletarPerfil` y `Perfil` | Los de `FormularioPerfil`. En `Perfil`, la fecha de nacimiento ya cargada es de solo lectura y no se envía en el `update` | RF-04, RF-86, RN-06, RN-10 |
| `PeliculaForm` | `nombre` (`required`, `maxLength(120)`); `duracion_min` (`min(1)`, `max(600)`); `fecha_estreno` (dd/mm/aaaa); `clasificacion`; `generos` (al menos uno, con `validate`); `preventa_porcentaje` (0 a 100); `preventa_dias` (1 a 60) | RF-08 a RF-11, RF-70 |
| `ProgramarFunciones` | `pelicula_id` (`required`); `formato`; `idioma`; `dias` (al menos uno); `hora` (`pattern` `HH:MM`); `desde` (chip); `semanas` (1 a 8) | RF-22, RF-78, D-21 |
| `FuncionForm` | Día por chip, hora, formato, idioma | RF-21 |
| `CuponesAdmin` | `codigo` (`pattern` `^[A-Z0-9_-]{3,20}$`); `porcentaje` (1 a 100); vigencia válida; `limite_usos` (≥ 1 o vacío); `edad_minima` obligatoria si el tipo es edad | RF-40 a RF-42 |
| `PreciosAdmin`, `ProductosAdmin`, `CombosAdmin`, `RecompensasAdmin`, `SalasAdmin`, `CuentasAdmin` | Montos > 0, nombres obligatorios, al menos un formato por sala, email y clave de la cuenta | RF-06, RF-16, RF-39, RF-46, RF-48, RF-65 |
| `PasoButacas` | `butacasContiguas`, máximo de butacas y al menos una | RF-38, RF-90 |
| `PasoPago` | `numero` (`required`, `validate` Luhn); `vencimiento` (`pattern` MM/AA, `validate` no vencida); `cvv` (`pattern` 3 o 4 dígitos); `titular` (`required`); `declaracion_edad` (obligatoria para el anónimo en +13 o +18) | RF-37, S-04, S-08 |

Fechas sin calendarios con scroll (RF-78, RNF-19):

- **Días:** chips (`SelectorFecha`, `SelectorDias`).
- **Horas:** botones y un campo `HH:MM` (`SelectorHora`).
- **Rango de la programación:** un chip de inicio más la cantidad de semanas (D-21).
- **Fecha de nacimiento y fecha de estreno:** campo de texto `dd/mm/aaaa` con `inputmode="numeric"`.

### 8.11 Utilidades y dependencias nuevas

En `shared/utils/`:

| Archivo | Exporta | Qué hace | Requerimientos |
|---|---|---|---|
| `errores.ts` | `mensajeError` | Sección 8.3. Traduce los códigos de la base y, en los errores de Supabase Auth, su `code` (por ejemplo `invalid_credentials`) | — |
| `rutas.ts` | `rutaSegura(ruta)` | Valida el `volver` de la URL: solo deja rutas internas | RF-86 |
| `archivos.ts` | `nombreSeguro(nombre)`, `motivoImagenInvalida(archivo)` | Normaliza el nombre de un archivo (C-15) y rechaza lo que no sea JPG, PNG o WebP de hasta 2 MB antes de subirlo | RNF-13 |
| `qr.ts` | `generarQr(codigo): Promise<string>` | Devuelve un dataURL PNG generado con `qrcode` | RF-53, RF-54 |
| `comprobante-pdf.ts` | `descargarComprobantePdf(comprobante, qr)` | Arma el PDF con `jspdf` y lo descarga con `save()`. Modelo acordado el 07/10: arriba y centrados "Cineverso", "Comprobante de compra", el QR y el código; debajo, alineados a la izquierda, película, clasificación, fecha y hora, sala, formato e idioma, las entradas con tipo y precio, el candy y el total; al final, la leyenda para +13 y +18. Recibe el QR que `VistaComprobante` ya generó, y `VistaComprobante` empieza a cargar este archivo apenas se muestra, así el clic descarga sin esperar | RF-51, RF-52 |
| `reporte-pdf.ts` | `exportarFacturacionPdf(filas, rango)` | Exporta el reporte de facturación a PDF | RF-82 |
| `excel.ts` | `exportarFacturacionExcel(filas, rango)` | Exporta el reporte de facturación a Excel con `write-excel-file` | RF-82 |

Las librerías pesadas no se importan directo desde los componentes. Los componentes cargan a demanda los archivos propios que las envuelven:

```ts
// genera el pdf del comprobante cargando jspdf y qrcode solo cuando se pide
const { descargarComprobantePdf } = await import('../utils/comprobante-pdf');
```

| Archivo propio | Lo carga | Chunk que genera el build (`namedChunks: true`) |
|---|---|---|
| `utils/qr.ts` | `VistaComprobante` | `qr-*.js` |
| `utils/comprobante-pdf.ts` | `VistaComprobante` | `comprobante-pdf-*.js` |
| `utils/reporte-pdf.ts` | `Reportes` | `reporte-pdf-*.js` |
| `utils/excel.ts` | `Reportes` | `excel-*.js` |
| `@zxing/browser` | `EscanerQr` | queda dentro del chunk de `/empleado` (`validacion`, 499 kB sin comprimir y 97 kB transferidos en F6) |

**Por qué:** Las librerías no pesan en la carga inicial (presupuesto de 500 kB de `angular.json`). Además, el nombre del chunk es predecible y `ngsw-config.json` puede precachear justo los dos que necesita `/comprobantes` sin conexión (sección 9). Si una librería la comparten dos archivos (por ejemplo, `jspdf` en el comprobante y en el reporte), el build la saca a un `chunk-*.js`, que también se precachea.

| Dependencia | Para qué | Por qué esta | Descartada |
|---|---|---|---|
| `@supabase/supabase-js` | Todo Supabase | La de clase | — |
| `@angular/service-worker` (vía `ng add @angular/pwa`) | PWA, aviso de versión y avisos del sistema | Oficial de Angular; lo pide RNF-14 | Workbox (otra API que aprender, sin necesidad) |
| `qrcode` y `@types/qrcode` | Generar el QR | Chica, sin dependencias y funciona sin conexión | Generar el QR en el servidor (el comprobante dejaría de funcionar offline) |
| `@zxing/browser` | Leer el QR con la cámara | Funciona en Chrome, Firefox y Safari | `BarcodeDetector` nativo (no existe en Safari ni en Firefox); `html5-qrcode` (sin mantenimiento activo) |
| `jspdf` | PDF del comprobante y del reporte | Genera en el navegador, también sin conexión | `pdfmake` (más pesada); PDF en una Edge Function (no funcionaría offline) |
| `write-excel-file` | Excel del reporte | Una función, solo escritura, `.xlsx` real | `xlsx` (la versión de npm está desactualizada y tiene vulnerabilidades); `exceljs` (pesada para un solo reporte); CSV (no es un Excel) |
| Gráficos | — | CSS y `@for` | Chart.js y ng2-charts |

Si el build avisa que `qrcode` o `@zxing/library` son CommonJS, se agregan a `allowedCommonJsDependencies` en `angular.json`. En F6, `@zxing/browser` y `@zxing/library` no avisaron: se usan sus versiones ESM.

### 8.12 Estilos e identidad

Es CSS puro, con un archivo por componente (C-16). `styles.css` define los **tokens**, y los componentes solo usan `var(--…)`. Para cambiar la identidad alcanza con editar ese bloque. La identidad vale "por ahora" y se puede ajustar sin tocar componentes. La vista previa con la paleta, la navbar y las tarjetas de película está publicada como página privada.

**Tipografía** (RNF-17):

| Uso | Fuente | Token |
|---|---|---|
| Títulos (`h1` a `h4`, marca) | Bree Serif, que tiene un solo peso (400) | `--fuente-titulos` |
| Texto general, botones y formularios | Poppins 400 y 500 | `--fuente-texto`, `--peso-normal`, `--peso-medio` |
| Botones y precios de productos | Poppins 700 (negrita) | `--peso-negrita` |

- Las fuentes se cargan desde Google Fonts con un `<link>` en `index.html` y `display=swap`, y cada una tiene su pila de respaldo.
- En F7, `ngsw-config.json` suma `https://fonts.googleapis.com/**` y `https://fonts.gstatic.com/**` al grupo `recursos`, para que también se vean sin conexión.
- `styles.css` hace que `button`, `input`, `select` y `textarea` hereden la fuente, y pone los botones en negrita.
- Los precios de productos usan la clase `.precio` del componente que los muestra, con `--peso-negrita` y `tabular-nums`.

**Paleta**: fondo casi negro de tono cálido, un solo violeta con dos variantes más oscuras y ningún color neón. Contrastes contra el fondo `#12100D` (AA: 4,5:1 para texto chico; 3:1 para texto grande, líneas e íconos; RNF-20):

| Token | Valor | Uso | Contraste |
|---|---|---|---|
| `--color-fondo` | `#12100D` | Página y cabecera | — |
| `--color-superficie` | `#201D19` | Tarjetas y paneles, siempre con sombra | — |
| `--color-superficie-alta` | `#2A2722` | Campos, botón secundario y etiquetas de estado | — |
| `--color-texto` | `#F3EFF9` | Texto principal y foco del teclado | 16,8:1 |
| `--color-texto-suave` | `#AAA9A8` | Texto secundario, menú inactivo | 8,1:1 |
| `--color-primario` | `#8226EF` | Botón principal, marca, subrayado de la navbar, butaca elegida | 3,1:1; texto encima 5,3:1 |
| `--color-primario-oscuro` | `#660FCC` | Botón principal al pasar el mouse | texto encima 7,4:1 |
| `--color-primario-profundo` | `#4B0B98` | Botón presionado, chips y etiquetas | texto encima 10,3:1 |
| `--color-exito` / `-error` / `-advertencia` | `#4FB286` / `#DE7A82` / `#D9A94E` | Estados | 7,3 / 6,5 / 8,8:1 |

Reglas del violeta:

- **Dónde va:** en rellenos con texto encima, en líneas sobre el fondo y en texto grande (24 px o más), como la marca. **Nunca en texto chico**: da 3,1:1 y el texto chico pide 4,5:1. Los enlaces van en `--color-texto`, subrayados.
- **Siempre sobre el fondo:** sobre `--color-superficie` el violeta baja a 2,9:1. Por eso la cabecera usa `--color-fondo`, y el subrayado y la marca quedan sobre el fondo.
- **Las variantes son el mismo tono con menos luz:** `#4B0B98` es `#8226EF` (267°, saturación 86 %) con 32 % de luz en lugar de 54 %, y `#660FCC` queda en el medio, con 43 %. No hay violeta claro ni aguamarina.
- **El foco del teclado** usa `--color-foco`, que vale `--color-texto`: un anillo violeta no llegaría a 3:1 sobre las tarjetas.

**Sin bordes** (no existe `--color-borde`):

- Las tarjetas y los paneles se separan con `--sombra` y con una superficie apenas más clara que el fondo. En un fondo casi negro, la sombra sola casi no se nota.
- Al pasar el mouse, la tarjeta sube 4 px y usa `--sombra-alta`.
- Los campos de formulario (F1) marcan su línea con una sombra interior: `box-shadow: inset 0 -2px 0 var(--color-texto-suave)`, que con el foco pasa a `inset 0 -3px 0 var(--color-primario)`. Esa línea identifica al campo y contrasta 8,1:1 (RNF-20).

**Radios:** `--radio-tarjeta` (12 px) en tarjetas y paneles, `--radio-control` (8 px) en botones y campos, y `--radio-chip` en los chips, que tienen forma de píldora.

**Tarjeta de película** (`TarjetaPelicula`, F2):

- La tarjeta entera es un enlace a `/peliculas/:peliculaId`, con proporción 1:2 y dos filas: 3/4 y 1/4.
- **Arriba**, el póster ocupa toda la fila con `object-fit: cover`. Como la tarjeta es 1:2, esa fila queda en 2:3, la proporción de un póster.
- **Abajo**, la superficie con el título (Bree Serif, hasta 2 líneas) y una línea de detalles en `--color-texto-suave`: duración con el pipe `duracion`, clasificación y género. Lo que no entra se corta con puntos suspensivos.
- En F14, el botón de alerta (`conAlerta`) va en la franja de abajo, como ícono a la derecha del título. El póster queda siempre limpio.

```css
/* reparte la tarjeta: 3/4 para el póster y 1/4 para el título y los detalles */
.tarjeta-pelicula {
  display: grid;
  grid-template-rows: minmax(0, 3fr) minmax(0, 1fr);
  aspect-ratio: 1 / 2;
  border-radius: var(--radio-tarjeta);
  overflow: hidden;
  background: var(--color-superficie);
  box-shadow: var(--sombra);
}
```

**Navbar** (`app.html` y `app.css`):

- A partir de 768 px, la cabecera es una grilla `1fr auto 1fr`: la marca va a la izquierda y la navegación en el centro exacto de la pantalla. En el celular, las dos van apiladas y centradas.
- La cabecera usa `--color-fondo` y se separa del contenido con `--sombra`, sin borde.
- Cada enlace tiene un `::after` de 3 px en `--color-primario`, con `transform: scaleX(0)` y `transform-origin: center`.
- En la sección activa (`routerLinkActive="activo"`), al pasar el mouse y con el foco del teclado, pasa a `scaleX(1)`. Así el subrayado crece desde el centro hasta las dos puntas de la palabra.
- Con `prefers-reduced-motion: reduce` la animación se desactiva (RNF-20).

```css
/* estira el subrayado hasta las dos puntas en la sección activa y al pasar el mouse */
.navegacion a:hover::after,
.navegacion a:focus-visible::after,
.navegacion a.activo::after {
  transform: scaleX(1);
}
```

**Butacas:** sus colores se definen en F4 con esta paleta: la elegida en `--color-primario` y las demás en neutros y estados. Cada estado se distingue además por forma o ícono, así que el mapa no depende solo del color.

Responsive (RNF-18): se diseña primero para el celular, con un único quiebre en `min-width: 768px` y el contenido limitado por `--ancho-maximo`. El presupuesto de 4 kB por componente de `angular.json` obliga a mantener chico el CSS de cada uno.

---

## 9. PWA

| Pieza | Qué se hace | Requerimientos |
|---|---|---|
| Instalación | `ng add @angular/pwa` (en F7) agrega `@angular/service-worker`, `ngsw-config.json`, `public/manifest.webmanifest`, `public/icons/`, `provideServiceWorker(...)` en `app.config.ts`, y el `<link rel="manifest">` y el `theme-color` en `index.html` | RNF-14 |
| Manifest | `name: "Cineverso"`, `short_name: "Cineverso"`, `lang: "es-AR"`, `start_url: "/"`, `display: "standalone"`, `theme_color` y `background_color` iguales a los tokens, íconos de 72 a 512 px y uno `maskable` | RNF-14, RNF-17 |
| Caché de recursos estáticos | Solo lo inicial y lo que necesita `/comprobantes`. Los chunks de las áreas quedan en `installMode: lazy` (ver abajo) | RNF-15, RNF-06 |
| Caché de la cartelera | `dataGroups` (ver abajo) | RNF-15 |
| Comprobantes sin conexión | Ver abajo | RNF-15, S-04 |
| Aviso de versión nueva | Como Clase 8, lámina 23: `app.ts` inyecta `SwUpdate`, tiene `actualizacionDisponible = signal(false)` y en el constructor se suscribe a `versionUpdates`, que la pone en `true` con `VERSION_READY`. `app.html` muestra con `@if` el botón "Nueva versión: recargar", que llama a `document.location.reload()` | RNF-15 |
| Avisos | Bandeja en tiempo real y aviso del sistema con la app abierta; push real solo en F16 (ver abajo) | RNF-16, RF-69, RF-87 a RF-89 |

#### Qué se precachea

En la configuración de producción de `angular.json` va `"namedChunks": true`. Con eso, el build nombra cada chunk diferido según su archivo. Se comprobó con Angular 22.2 en un proyecto mínimo con dos áreas diferidas y un `import()` dinámico:

| Archivo del build | Qué es |
|---|---|
| `main-*.js` | Lo inicial. La app no tiene `polyfills` porque no usa zone.js, así que el bundle inicial es solo `main-*.js` |
| `peliculas-admin-*.js`, `validacion-*.js`, `perfil-*.js`, etc. | Cada pantalla de las áreas, cargada con `loadComponent`. El nombre es el del archivo del componente |
| `checkout-*.js` | El checkout |
| `qr-*.js`, `comprobante-pdf-*.js` | Los envoltorios de la sección 8.11 |
| `chunk-*.js` | Código compartido entre varios chunks diferidos (por ejemplo, `jspdf` si lo usan el comprobante y el reporte) |

`ngsw-config.json`:

```json
{
  "$schema": "./node_modules/@angular/service-worker/config/schema.json",
  "index": "/index.html",
  "assetGroups": [
    {
      "name": "inicial",
      "installMode": "prefetch",
      "updateMode": "prefetch",
      "resources": {
        "files": ["/favicon.ico", "/index.html", "/manifest.webmanifest", "/*.css", "/main-*.js", "/polyfills-*.js", "/chunk-*.js", "/qr-*.js", "/comprobante-pdf-*.js"]
      }
    },
    {
      "name": "diferido",
      "installMode": "lazy",
      "updateMode": "prefetch",
      "resources": { "files": ["/*.js"] }
    },
    {
      "name": "recursos",
      "installMode": "lazy",
      "updateMode": "prefetch",
      "resources": {
        "files": ["/icons/**", "/**/*.(svg|png|jpg|webp|woff2)"],
        "urls": ["https://fonts.googleapis.com/**", "https://fonts.gstatic.com/**"]
      }
    }
  ],
  "dataGroups": [
    {
      "name": "cartelera",
      "urls": ["https://*.supabase.co/rest/v1/peliculas_cartelera*"],
      "cacheConfig": { "strategy": "freshness", "maxSize": 50, "maxAge": "1d", "timeout": "5s" }
    },
    {
      "name": "imagenes",
      "urls": ["https://*.supabase.co/storage/v1/object/public/**"],
      "cacheConfig": { "strategy": "performance", "maxSize": 200, "maxAge": "7d" }
    }
  ]
}
```

Cómo se reparte:

- Cada archivo va al **primer** grupo que lo nombra. Todo `.js` que no está en `inicial` cae en `diferido`.
- `diferido` se descarga y se guarda recién cuando el usuario entra al área (`installMode: lazy`).
- Si sale una versión nueva, se actualiza solo lo que ya estaba guardado (`updateMode: prefetch`).
- `chunk-*.js` va en `inicial`. Lo compartido entre áreas es poco, y así nunca falta una pieza del arranque sin conexión.

**Por qué:** Con este reparto, un visitante no descarga el código de `/admin` ni a través del service worker. Se comprueba en `dist/tp1-sistema-cine/browser/ngsw.json`: el grupo `inicial` no tiene ningún chunk de pantallas de `/admin`, `/empleado` ni `/cuenta`, como `peliculas-admin-*.js` o `validacion-*.js`. Es una mejora de **rendimiento**: los datos los protegen el RLS y las RPC, no el service worker (sección 4).

**Desvío respecto de la clase:** en Clase 8, lámina 21, el grupo `app` precachea todo el JS con `/*.js`. Acá los chunks de las pantallas quedan en `installMode: lazy`, a propósito. Si se precacheara todo, el service worker le descargaría a cualquier visitante el código del admin, y la carga diferida con `canMatch` dejaría de ahorrar esa descarga.

#### Comprobantes sin conexión

`/comprobantes` es una ruta del bundle inicial. Para mostrar un comprobante sin red necesita tres cosas:

| Necesita | De dónde lo saca sin red |
|---|---|
| Los datos de la compra | `ComprobantesLocales` (`localStorage`). Se guardan después de un pago aprobado y cada vez que se abre `DetalleCompra` |
| El QR | `qr-*.js`, precacheado en `inicial`, que lo genera a partir del código |
| El PDF | `comprobante-pdf-*.js` y, si lo hay, el `chunk-*.js` con `jspdf`. Los dos están en `inicial` |

Visitante anónimo que compra en su primera visita: la compra se hace con red. El service worker se instala unos segundos después (`registerWhenStable:30000`) y precachea el grupo `inicial`. Desde ese momento `/comprobantes` funciona en modo avión.

**Por qué precachear `jspdf` y no guardar el PDF ya generado:** Es una regla en `ngsw-config.json` y no hace falta código para guardar archivos binarios. El costo es una descarga en segundo plano, una sola vez por versión.

#### Caché de datos

`dataGroups`:

| Grupo | URL | Estrategia | Vida |
|---|---|---|---|
| `cartelera` | `https://*.supabase.co/rest/v1/peliculas_cartelera*` | `freshness` (red primero, la copia guardada si no hay red), `timeout: 5s` | 1 día |
| `imagenes` | `https://*.supabase.co/storage/v1/object/public/**` | `performance` (la copia guardada primero) | 7 días |

El service worker solo guarda pedidos GET, así que las RPC nunca se cachean.

**Por qué esas estrategias:** La cartelera tiene que estar actualizada pero verse sin red. Los pósters casi no cambian.

#### Avisos: bandeja en tiempo real y aviso del sistema con la app abierta

Hasta que el docente diga si el aviso tiene que llegar con la app cerrada, no hay push real (D-24). Los recordatorios de 24 h y 2 h (RF-87, RF-88), el aviso de QR validado (RF-89) y las alertas de Próximamente (RF-69) llegan por dos canales:

| Canal | Cómo funciona | Cuándo llega |
|---|---|---|
| Bandeja de la app | La base inserta en `notificaciones` (con el job `recordatorios`, `validar_qr` o `procesar_alertas`) y `NotificacionesStore` lo recibe por Realtime, como el canal de `tasks` de Clase 6, lámina 10 | Siempre: en vivo si la app está abierta, o al abrirla |
| Aviso del sistema | Si el usuario dio permiso, `AvisosSistema.mostrar()` llama a `registration.showNotification(titulo, { body, icon, data })` del service worker | Solo con la app abierta, en cualquier pantalla |

Flujo:

1. El cliente toca **Activar avisos** en `/cuenta/notificaciones`. El navegador solo pide permiso a partir de un gesto del usuario, así que `AvisosSistema.activar()` llama a `Notification.requestPermission()` desde ese botón.
2. La base inserta una notificación, y Realtime la entrega a `NotificacionesStore`, que la agrega a la bandeja.
3. Si hay permiso, `NotificacionesStore` pide a `AvisosSistema` que muestre el aviso del sistema. En `data` va `{ onActionClick: { default: { operation: 'navigateLastFocusedOrOpen', url } } }`.
4. Al tocar el aviso, el service worker de Angular enfoca la app y abre la URL, por ejemplo el QR de la compra (RF-87).

**Por qué `showNotification` del service worker y no `new Notification()`:** En Chrome para Android el constructor `new Notification()` no está permitido; solo funciona el aviso que muestra el service worker. El service worker ya está registrado por la PWA.

La bandeja es el canal garantizado (S-10). El aviso del sistema es un complemento, y no se vio en clase (sección 13.4). En iOS solo funciona con la PWA instalada (iOS 16.4 o superior).

#### Push con la app cerrada (F16, D-25)

La cátedra exigió el 09/10 que los avisos lleguen también con la app cerrada. Se suma un tercer canal, sin cambiar los dos de arriba: cada INSERT en `notificaciones` también se envía como push a los dispositivos suscriptos del usuario.

| Pieza | Dónde | Qué hace |
|---|---|---|
| Suscripción | `AvisosSistema.suscribir()` y `PushApi.guardar()` | Después del permiso (botón **Activar avisos**), y también al entrar un cliente si el permiso ya estaba dado: `SwPush.requestSubscription({ serverPublicKey })` (mismo paquete que `SwUpdate`, Clase 8) y la RPC `guardar_suscripcion_push` |
| Baja | `AvisosSistema.desuscribir()`, desde `salir()` en `app.ts` | Llama a `borrar_suscripcion_push` antes del `signOut` y da de baja la suscripción del navegador, para que el próximo usuario del equipo no reciba avisos ajenos. Si falla, la salida sigue igual |
| Trigger | `03_push.sql`, `after insert on notificaciones` | Si el usuario tiene suscripciones, `net.http_post` (pg_net) a la función con la notificación y las suscripciones. Es asíncrono: no frena `validar_qr` ni los jobs, y si falla, la bandeja igual tiene el aviso |
| Secreto compartido | Vault (`push_secreto`) y secreto `PUSH_SECRETO` de la función | El trigger lo manda en el encabezado `x-push-secreto` y la función rechaza todo lo demás con 401 |
| Edge Function | `enviar-push` (`supabase/functions/`, local) | Cifra y firma con `web-push` y envía a cada dispositivo. No lee la base: no usa `service_role` |
| Mensaje | La función | `{ notification: { title, body, icon, tag, data.onActionClick } }`, el formato que muestra el service worker de Angular. Al tocarlo abre la `url` (el QR de la compra) |
| Sin duplicados | `tag: notificacion-{id}` | El aviso por Realtime y el push usan el mismo `tag`: con la app abierta, uno reemplaza al otro |
| RF-89 completo | `03_push.sql`, `before insert on notificaciones` | En `validacion_acceso` y `validacion_candy` suma al cuerpo el sector, la fecha y la hora, por ejemplo "· Acceso · 09/10 21:34 h", sin reescribir `validar_qr` |

Claves:

| Clave | Dónde va |
|---|---|
| VAPID pública | `environment.ts` y `environment.prod.ts` (es pública por diseño) y secreto `VAPID_PUBLIC_KEY` |
| VAPID privada | Solo como secreto `VAPID_PRIVATE_KEY` de la función. Nunca en el repo ni en la base |
| `VAPID_SUBJECT` | Secreto de la función, con forma `mailto:...` (Apple rechaza otras formas) |
| Secreto compartido | Vault y secreto `PUSH_SECRETO` de la función |

Dispositivos:

| Dispositivo | Cómo llega | Cuidado |
|---|---|---|
| Android (Chrome) | Con la app y el navegador cerrados | Ahorro de batería y "No molestar" |
| PC (Chrome o Edge) | Con la pestaña cerrada, mientras el navegador siga corriendo; si está cerrado, al abrirlo | Notificaciones de Chrome habilitadas en Windows y "Asistencia de concentración" apagada |
| iPhone (iOS 16.4 o superior) | Solo con la PWA instalada (Safari → Compartir → Agregar a inicio) | La PWA no comparte sesión con Safari: hay que iniciar sesión y tocar **Activar avisos** dentro de la app instalada |

Sin herramientas de demo (decisión del 09/10): los avisos salen solo por los caminos reales, que es lo que pide el documento de requerimientos.

- **Recordatorios:** el job `recordatorios` (cada 5 minutos). Una compra para una función que empieza dentro de 24 h y 5 min recibe el de 24 h en el primer ciclo después de que falten 24 h; con 2 h y 5 min, el de 2 h. `procesar_recordatorios()` no avisa a compras hechas con menos de 24 h o 2 h de anticipación.
- **Alertas:** el job `alertas-venta` (cada 10 minutos), cuando la película pasa a tener entradas a la venta.
- **QR validado:** al validar en `/empleado`, en el acto.
- **Historia:** el 09/10 se armó `/admin/avisos` con las RPC `probar_recordatorio` y `disparar_alertas` (en `03_push.sql`) y se sacó ese mismo día porque no está en el documento de requerimientos; `04_quita_demo_avisos.sql` borra las RPC y vuelve `alertas-venta` a 10 minutos.

Cómo probarlo:

- `ng serve` no registra el service worker.
- Para probar localmente: `ng build` y servir `dist/tp1-sistema-cine/browser` con `npx http-server -p 8080 -c-1`.
- La prueba completa se hace en Vercel, que sirve HTTPS.
- Después de cada `ng build`, revisar en `ngsw.json` qué archivos quedaron en el grupo `inicial`.

---

## 10. Pruebas

Se usa Vitest a través de `@angular/build:unit-test` y se ejecuta con `npm test -- --watch=false` (RNF-23).

**Decisión: solo se prueban funciones puras**, las de `shared/rules/` y `shared/utils/`, sin TestBed. En clase el CLI corre los tests (Clase 2, lámina 6), pero no se vio cómo escribirlos; probar funciones puras es lo más simple y cubre todas las reglas que pide RNF-23.

Consecuencias:

- Componentes, directivas, pipes, guards, stores y el validador no tienen spec. `angular.json` tiene `skipTests: true` para que el CLI no las genere.
- Pipes, directivas y validador delegan su lógica en una regla pura, y esa regla sí se prueba:

  | Pieza | Regla que usa |
  |---|---|
  | `duracion` | `formatearDuracion` |
  | `precioArs` | `formatearPrecio` |
  | `edad` | `calcularEdad` |
  | `appRestrictEdad` | `motivoRestriccionEdad` |
  | `butacasContiguas` | `motivoNoContiguas` |

- Las fechas de las pruebas llevan un offset explícito, por ejemplo `new Date('2026-10-14T23:59:00-03:00')`, para que el resultado no dependa de la zona horaria de la máquina.

| Spec | Casos | Reglas |
|---|---|---|
| `rules/precio.spec.ts` | Base 2D de 8000, recargo VIP de 3000, preventa del 20 %: estándar → 6400, VIP → 9400 (el recargo se suma completo). Sin preventa: VIP → 11000. Combo en 3D (base 10000) con butaca VIP → diferencia 2000 + 3000 = 5000. `entradasConCombos` con una VIP y una estándar en 3D: un combo cubre la estándar (2000) y la VIP paga 13000; dos combos → 2000 y 5000. Resumen de la compra de F8 (2000 + 13000 de entradas, 3500 de candy y 12000 de combo) → 30500 con los tres subtotales. `enPreventa` con estreno el 15/10 y 7 días: 07/10 23:59 → no, 08/10 00:00 → sí, 14/10 23:59 → sí, 15/10 00:00 → no (hora de Buenos Aires). Resumen con subtotal 20000, cupón del 20 % y crédito de 5000 → total 16000, a pagar 11000 | RN-07, RN-08, RN-11, RN-12, RN-15, S-09, S-12 |
| `rules/candy.spec.ts` (desde F8) | Agrupa por categoría en orden alfabético y deja afuera las vacías. `detalleCombo` empieza por la entrada 2D. `cambiarCantidad` agrega, suma, respeta el máximo, quita en 0 y no baja de 0. `lineasPedido` toma nombre y precio del catálogo e ignora lo que ya no está; `subtotalLineas` suma con dos decimales | RF-46 a RF-49 |
| `rules/cupones.spec.ts` | Primera compra: el anónimo se rechaza; un cliente con compras previas se rechaza; un cliente nuevo se acepta. Edad: 50 años cumplidos se acepta y 49 se rechaza (el cumpleaños se cuenta el mismo día). Vencido, agotado o inactivo se rechaza con su motivo. `mejorCupon` elige el porcentaje mayor y solo uno | RN-09, RN-10, RN-11, S-06, S-23 |
| `rules/edad.spec.ts` | Nacido el 07/10/2013 → 12 años el 06/10/2026 y 13 el 07/10/2026. Nacido el 29/02/2008 → 17 el 28/02/2026 y 18 el 01/03/2026, igual que `age()` de Postgres. `motivoRestriccionEdad`: ATP sin motivo; anónimo sin motivo; cliente de 16 en +18 con motivo; edad a la fecha de la función y no de hoy | RF-07, RN-06, S-04 |
| `rules/perfil.spec.ts` | Perfil completo con los cinco datos (cero días de vacaciones vale). Borrador con la fecha en dd/mm/aaaa y el consentimiento marcado si hay datos sensibles. La fecha viaja solo en la primera carga. Fecha inexistente, futura o anterior a 1900 se rechaza | RF-01, RF-86, RN-06 |
| `utils/rutas.spec.ts` | `rutaSegura` deja pasar `/cuenta/completar-perfil` y manda a `/` una URL externa o `//otro.sitio` | RF-86 |
| `rules/puntos.spec.ts` | `puntosGanados(12500.75)` → 12500. Un movimiento con `disponible_desde` futuro cuenta como pendiente y uno pasado como disponible. Una anulación neta los pendientes. Un canje resta de los disponibles | RN-12, RN-13, RN-19, S-07 |
| `rules/asignacion.spec.ts` | Dos salas libres → la de menor número. Una sala inactiva o sin el formato se excluye. Una función nueva a las 18:00 entra justo cuando termina la limpieza de la anterior (rango semiabierto) y choca si esa limpieza termina 18:30. La limpieza de la nueva choca con una de las 20:15 y, con limpieza 0, no. Sin sala libre → `null`. Fechas repetidas o desordenadas se ordenan y se quitan, como en la base. Las fechas del mismo lote se tienen en cuenta entre sí (con un solo horario y películas de hasta 600 min no pasa; queda como defensa) | RN-01, RN-03, RF-23, RF-24, S-19 |
| `rules/sala.spec.ts` | `etiquetaSala` con y sin nombre. `agruparButacas` ordena por fila y número y reparte en los tres bloques | RF-17, RF-20 |
| `rules/contiguidad.spec.ts` | A5, A6, A7 (desordenadas) → válido. Una sola butaca o ninguna → válido. A4 y A5 → pasillo. A5 y A7 → "Hay lugares sin elegir entre A5 y A7". A5 y B5 → distinta fila. J2 y J3 → pasillo (en J el bloque izquierdo es de 1 a 2) | RN-25, RF-90 |
| `rules/fechas.spec.ts` | `proximosDias` arranca en el día de Buenos Aires aunque en UTC ya sea el siguiente; `etiquetaDia` da "Hoy", "Mañana" o "Vie 16/10". `fechasRecurrentes([1, 2, 5], '2026-10-12', 2)` → lunes, martes y viernes de dos semanas. `inicioBA('2026-10-12', '23:30')` → 02:30 UTC del 13 y `horaBA` vuelve a 23:30. `agruparPorDia` por día de Buenos Aires. `PATRON_HORA` rechaza 24:00, 18:60 y 9:30. `fechaDesdeTexto('31/02/2026')` → inválida | RF-22, RF-34 |
| `rules/tarjeta.spec.ts` | `4242424242424242` → Luhn válido; `4242424242424241` → inválido. `12/25` → vencida al 06/10/2026 | S-08 |
| `rules/formato.spec.ts` | `formatearDuracion`: 135 → `2 h 15 min`, 120 → `2 h`, 45 → `45 min`, 0 → `0 min`, `null` o negativo → `''`. `formatearPrecio`: 12500 → `$ 12.500,00`, `null` → `''` | RNF-26, RNF-21 |
| `rules/pelicula.spec.ts` | Búsqueda sin tildes ni mayúsculas; géneros con "alguno de"; clasificación; formato e idioma de las funciones futuras; filtros combinados. Orden por estreno en los dos sentidos. Borrador con la fecha en dd/mm/aaaa y vuelta a aaaa-mm-dd | RF-13, RF-14 |
| `utils/archivos.spec.ts` | `nombreSeguro` quita tildes y símbolos. `motivoImagenInvalida` acepta WebP de 2 MB y rechaza GIF o más de 2 MB | RNF-13 |
| `utils/errores.spec.ts` (ya existe) | Código conocido de la base → su mensaje en español. Una restricción con nombre (`salas_numero_key`, `funciones_sin_solapamiento`) gana sobre el mensaje genérico de su código. Mensaje y detalle de un error de Supabase → unidos. `Error` común → su mensaje. Algo desconocido → texto genérico | — |

Lo que no se prueba con Vitest:

- **Funciones SQL:** se prueban a mano en cada fase con el criterio de terminado y las consultas de verificación de cada script.
- **Pantallas, guards y directivas:** se prueban a mano en el navegador, con el criterio de terminado de cada fase.

---

## 11. Plan de implementación

Reglas del plan:

- **Despliegue:** cada fase termina desplegada en Vercel, con `npm test -- --watch=false` y `ng build` sin errores.
- **Base:** `tp1-sistema-cine.sql` crea en F0 todo lo de F1 a F14 (sección 5.12). Si una fase necesita cambiar la base, el cambio va en un script nuevo numerado desde `02` (sección 7.6).
- **README:** se actualiza en cada fase, en "Arquitectura y decisiones técnicas" (RNF-25).
- **Regresión:** toda fase que empieza a usar otra parte de una RPC repite la prueba de la fase que la usó primero:
  - `confirmar_compra` (F8, F9, F10 y F13) → la **compra básica de F5**:
    - un anónimo compra dos butacas 2D con `4242 4242 4242 4242`;
    - descarga el PDF con el QR;
    - con `4000 0000 0000 0002`, el pago se rechaza y las butacas se liberan.
  - `validar_qr` (F8) → la validación de F6.
  - `cancelar_compra` (F13) → la cancelación de F10.
- **Commits:** descriptivos, en español, con el formato `tipo(área): descripción`. Por ejemplo, `feat(compra): agrega confirmar_compra con pago simulado`.
- **Técnicas:** si hace falta una que no se vio en clase, se frena y se consulta (sección 13.4).

El README que se sube va sumando, para que el docente pueda probar la app desplegada:

| Contenido | Fase |
|---|---|
| Arquitectura y decisiones técnicas | Cada fase |
| La limitación del correo de recuperación de clave (D-16) | F0 |
| Los roles de las cuentas de prueba, sin emails ni claves | F1 |
| Las tarjetas de prueba (D-22) | F5 |

Las cuentas de prueba se crean solo para la evaluación, sin datos reales. El repositorio era público (desde el 09/10 es privado, con los profes como colaboradores); el README nombra solo los roles: los emails y las claves se mandan al docente por mail, y las claves se cambian después de la nota.

Calendario:

| Fecha | Fases |
|---|---|
| 06/10 | F0, F1 |
| 07/10 | F2, F3 |
| 08/10 | F4, F5, F6 |
| 09/10 | F7, F8, F9 |
| 10/10 | F10, F11 |
| 11/10 | F12, F13, F14 |
| 12/10 | F15. **No se agregan funciones**: se prueba la app desplegada, se cierra el README y se prepara el oral |
| 13/10 | Entrega |
| 09/10 (replanificado) | F16: push con la app cerrada (ver abajo) |

**Replanificación del 09/10 (novedades de la cátedra).** Manda sobre el calendario de arriba:

**Contexto para retomar (anotado el 09/10, 16:30, después de DEFENSA):**

- **Hecho y pusheado:** F0 a F11, el acceso para la corrección (`e4af56a`, sección 7.7), F16 push (`9ba9bb7`), `VerClave` (`cb7aea7`) y la barra con grupos más la quita de `/admin/avisos` (`cdb816e`). `04_quita_demo_avisos.sql` está aplicado (verificado por lectura).
- **DEFENSA.md al día (local):** filas de F16 y de los pedidos del 09/10, secciones 2.15, 2.16, 2.23 (push) y 2.24 (barra), flujos 6.8 y 6.8 bis, preguntas 45 a 58 y la sección 12, "Lo visto en clase, decisión por decisión". Build con inicial de 562 kB y 179 pruebas en verde.
- **F14 recortada:** cerrada (`e3cff7c`, ver su "Estado").
- **Tabla de verificación:** hecha el 09/10, todo ✅ (detalle en la sección 10 de DEFENSA). F1 queda cerrada: OAuth con GitHub y Google probado en Vercel.
- **Push arreglado a las 18:54:** no salía nunca (401 por el secreto); ver el "Estado" de F16. F14 recortada probada: la alerta llegó a la bandeja a las 17:20 y el push funciona desde el arreglo.
- **Orden acordado después de la tabla:** preventa en `PeliculaForm` (resto de F14, RF-70), F12 y F13 si llega antes del corte.


- **Plazo:** la entrega es el 09/10 hasta las 23:59. Después, hasta las 18:00 del 10/10, solo cambios mínimos o sutiles (textos y estilos, sin SQL ni Edge Functions).
- **Repo:** pasó a privado y los profes son colaboradores.
- **Orden:**
  1. Acceso para la corrección (sección 7.7, D-26).
  2. F16, push con la app cerrada (D-25).
  3. Barra de navegación con grupos desplegables y menú lateral en el celular (pedido del 09/10, sección 8).
  4. F14 recortada: el botón de alerta, para RF-69.
  5. Tabla de verificación en Vercel de lo que pidió el profe: OAuth con Google y GitHub; push de 24 h, 2 h y QR validado; `butacasContiguas`; pipes `duracion`, `precioArs` y `edad`; `appHighlightButaca` y `appRestrictEdad`; carga diferida de `/admin` y `/empleado` con guards. Lo que falte va antes que F12 y F13.
  6. F12 (reseñas) y F13 (puntos), si queda tiempo.
- **Alcance:** solo lo que está en el documento de requerimientos y lo que pide explícitamente el profe; nada de herramientas de demo.
- **Cada bloque** cierra con `ng build` y las pruebas en verde, y el usuario commitea y pushea, así Vercel siempre tiene una versión que anda.
- **Corte a las 21:00 del 09/10:** lo que no esté queda como pendiente en el README.

El 09/10 quedan el flujo central, la PWA y todo lo que exigió el docente (F0 a F7). F8 a F14 van en el orden de prioridad acordado:

1. Candy y combos.
2. Cupones.
3. Cancelaciones y crédito.
4. Reportes y auditoría.
5. Reseñas.
6. Puntos.
7. Preventa.

### F0 · Proyecto, Supabase y despliegue

1. El proyecto Supabase `tp1-sistema-cine` ya está creado y la base está vacía. Copiar la URL y la clave publicable desde Settings → API.
2. En `tsconfig.json`, agregar `"strict": true` y `"strictTemplates": true`, y corregir lo que falle.
3. Environments con el esquema de clase: `environment.ts` y `environment.prod.ts`, con `fileReplacements` en producción y `supabaseUrl` y `supabaseAnonKey` completos.
4. Correr `npm i @supabase/supabase-js` y crear `shared/services/supabase.client.ts`.
5. Pegar `supabase/sql/01_base.sql` y después `supabase/sql/tp1-sistema-cine.sql` en el SQL Editor, crear los buckets `posters` y `productos` desde el panel, y verificar con las consultas de la sección 7.6.
6. Reemplazar el placeholder de `app.html` por un layout con navegación y `<router-outlet />`. `Inicio` muestra provisoriamente los formatos de `precios_formato`, para comprobar la conexión con Supabase.
7. Poner los tokens provisorios en `styles.css`. En `index.html`, usar `lang="es"` y el título "Cineverso".
8. En `app.config.ts`, `LOCALE_ID` en `es-AR` y `DATE_PIPE_DEFAULT_OPTIONS` con `-0300`; en `main.ts`, `registerLocaleData(localeEsAr)`.
9. Subir el repo a GitHub en la rama `main`, como en Clase 6, lámina 21.
10. En Vercel, importar el repo con el preset Angular y el directorio de salida `dist/tp1-sistema-cine/browser` (Clase 6, lámina 22). El preset ya resuelve el 404 al refrescar una ruta (Clase 6, errores típicos).
11. Escribir el esqueleto del README, con la limitación del correo de recuperación (D-16) y lo que tiene la base.

| | |
|---|---|
| Requerimientos | RNF-01, RNF-08, RNF-17 (tokens), RNF-21, RNF-24, RNF-25 |
| Terminado | `https://tp1-sistema-cine.vercel.app` muestra el layout y los cuatro formatos que vienen de Supabase. Una ruta inexistente vuelve a `/` sin dar 404 |

### F1 · Autenticación, perfiles, roles y áreas

Orden: email y contraseña → GitHub → Google.

1. La base de esta fase ya está creada (`tp1-sistema-cine.sql`, sección 5.12).
2. `AuthApi`, `PerfilApi` y `SesionStore`.
3. Guards en `shared/guard/`: `invitadoGuard`, `clienteGuard`, `empleadoGuard`, `adminGuard`, `puedeComprarGuard` y `pendingChangesGuard` (sección 8.6).
4. **Email** (pasos B y C de la sección 7.1):
   - `Login` con email y clave;
   - `Registro` con email y clave, que lleva a `CompletarPerfil` (`FormularioPerfil`);
   - `RecuperarClave` y `RestablecerClave`, con el flujo completo.
5. Áreas `/cuenta`, `/empleado` y `/admin`: cada pantalla con `loadComponent` y el guard de su rol en `canMatch` (sección 4), y el menú de cada rol en `app.html` con `@if`. Al principio son pantallas mínimas. `Perfil` muestra los datos con el pipe `edad` y la fecha de nacimiento como solo lectura.
6. Primer admin por SQL. `CuentaInternaApi` (segundo cliente sin sesión y la RPC `asignar_rol`, sección 7.4) y `CuentasAdmin`.
7. **GitHub:** pasos A y D de la sección 7.1, y el botón en `Login`.
8. **Google:** paso E de la sección 7.1, con la app **publicada**, y el botón.
9. `CompletarPerfil` para las cuentas que entran por OAuth.
10. Datos sensibles con consentimiento.
11. `Privacidad` en `/privacidad`: qué datos guarda Cineverso según la base, para qué se usan y un mail de contacto. Existe porque Google la pide para pasar el login a producción. Se enlaza desde el pie de todas las páginas, el registro y el campo del tipo de sangre.
12. Cuentas de prueba (cliente, empleado y admin). El README nombra solo los roles; los emails y las claves se mandan por mail al docente.

| | |
|---|---|
| Requerimientos | RF-01, RF-02, RF-04 (datos), RF-06, RF-07, RF-77 (estructura), RF-86, RN-06 y RN-10 (fecha inmutable), RNF-02, RNF-05, RNF-06, RNF-09, RNF-10, RNF-22, RNF-26 (`edad`) |

Criterio de terminado, en Vercel:

- **Email:** registrarse, cerrar sesión, volver a entrar y recuperar la clave con una dirección del equipo del proyecto.
- **OAuth:** entrar con GitHub y con Google y completar el perfil. Para Google, usar una cuenta que no esté en el proyecto de Google Cloud.
- **Fecha de nacimiento:** intentar cambiarla por la API (por ejemplo, con un `update` desde la consola del navegador) da `FECHA_NACIMIENTO_INMUTABLE`.
- **Roles:** el admin crea un empleado desde `CuentasAdmin` sin cerrar su propia sesión. El empleado entra a `/empleado` y, si intenta entrar a `/admin`, termina en `/`.
- **Escalada:** un cliente que llama a `asignar_rol` recibe `SOLO_ADMIN`.
- **Carga diferida:** en DevTools → Network, un cliente no descarga el chunk de `/admin`.
- **Datos sensibles:** el admin no ve el tipo de sangre de nadie.
- **Pruebas:** pasan las de `rules/edad.spec.ts`.

### F2 · Catálogo y cartelera

**Estado (07/10):** código completo, con pruebas y build sin avisos. Falta que el usuario la pruebe en Vercel con el criterio de terminado y la commitee. El contexto para seguir está al principio de F3.

1. La base y el bucket `posters` ya están creados (F0).
2. `PeliculaApi`, `GeneroApi`, `ImagenApi` y `CarteleraStore`.
3. Admin: `PeliculasAdmin`, `PeliculaForm` con póster y `GenerosAdmin`. Los formularios implementan `ConCambiosPendientes` para el `pendingChangesGuard`.
4. Público:
   - `Inicio`, con el top 3 y el listado. Hasta que exista `top_peliculas` en F5, el top 3 lo arma `CarteleraStore` con los estrenos más recientes. El listado tiene buscador y filtros por género y clasificación; formato e idioma se suman en F3, cuando hay funciones.
   - `PeliculaDetalle`, que recibe la película del `peliculaResolver` como input, y `Proximamente`.
5. `TarjetaPelicula` y el pipe `duracion`.

| | |
|---|---|
| Requerimientos | RF-08 a RF-11, RF-13 (orden por estreno), RF-14, RF-15 (sin funciones ni reseñas), RF-68 (listado), RNF-03, RNF-13, RNF-26 (`duracion`) |

Criterio de terminado:

- El admin crea una película con póster y dos géneros, la pasa a "en cartelera" y la ve en `/`.
- Una película en borrador no aparece para el visitante.
- El filtro por género funciona.
- Pasan las pruebas de `rules/formato.spec.ts`.

### F3 · Salas, precios y funciones

**Estado (07/10):** código completo, con pruebas y build sin avisos (inicial de 515 kB). Falta que el usuario la pruebe con el criterio de terminado y la commitee. El contexto para seguir está al principio de F4.

1. La base de esta fase ya está creada (`tp1-sistema-cine.sql`, sección 5.12).
2. `SalaApi`, `FuncionApi` y `ConfiguracionApi`.
3. Admin:
   - `SalasAdmin` y `SalaButacas` (tipo de butaca y fuera de servicio);
   - `PreciosAdmin`: precio por formato y recargo VIP, con el pipe `precioArs`;
   - `FuncionesAdmin` y `FuncionForm`;
   - `ProgramarFunciones`, con `SelectorDias`, `SelectorHora`, los chips de inicio, la cantidad de semanas y la vista previa de `asignarSalas`.
4. Las funciones aparecen en la ficha.

| | |
|---|---|
| Requerimientos | RF-12, RF-15 (funciones), RF-16 a RF-25, RF-39, RF-78, RF-84 (funciones y precios), RN-01 a RN-03, RN-24, RNF-11, RNF-26 (`precioArs`) |

Criterio de terminado:

- Crear tres salas, una sin 4D.
- Programar "lunes, martes y viernes a las 18:00" durante dos semanas. La vista previa muestra la sala de cada fecha y marca las que no tienen sala.
- Insertar a mano en el SQL Editor una función que se solape con otra: la base la rechaza.
- Cambiar un precio: el cambio aparece en la auditoría.
- Pasan las pruebas de `rules/asignacion.spec.ts`.

### F4 · Mapa de butacas en tiempo real

**Contexto para retomar (anotado el 07/10, al cerrar F3):**

- **Estado:**
  - F0, F1 y F2 commiteadas. F3 con código completo: `SalaApi`, `FuncionApi`, `ConfiguracionApi` completa, el pipe `precioArs`, `SalasAdmin`, `SalaButacas`, `PreciosAdmin`, `FuncionesAdmin`, `FuncionForm`, `ProgramarFunciones`, `SelectorDias`, `SelectorHora`, `SelectorFecha`, `SelectorFormatos`, los filtros de formato e idioma en la cartelera y las funciones en la ficha. Reglas nuevas: `asignacion.ts`, `sala.ts` y lo agregado en `fechas.ts`, con sus pruebas (79 en total).
  - Siguen pendientes de F1 y F2: Google, las cuentas de prueba, el mail de contacto de `/privacidad` y los límites de los buckets.
- **Base:** `funciones` no tiene políticas de alta ni de cambio: todo pasa por `programar_funciones` y `modificar_funcion` (definer, solo `authenticated`). `asignar_sala` no se puede llamar desde la API. F4 no debería necesitar SQL nuevo; si hiciera falta, va en `03_motivo.sql`.
- **Presupuesto:** el inicial pesa 515 kB (aviso en 600 kB).
- **Qué hay para reusar en F4:**
  - `agruparButacas` (`rules/sala.ts`) es genérica: sirve para armar las filas y bloques de `MapaButacas` con `ButacaVista`.
  - `SalaButacas` ya dibuja la sala con pasillos (CSS con columnas fijas por bloque y desplazamiento horizontal en el celular) y distingue los tipos por forma además de color: base para el CSS de `MapaButacas`.
  - `SelectorFecha`, `etiquetaDia`, `horaBA` y `agruparPorDia` sirven para el paso de función del checkout. La ficha ya agrupa las funciones por día con `FuncionApi.dePelicula` y los precios por formato.
  - `mensajeError` también reconoce restricciones por nombre (`/constraint "(\w+)"/`).
- **Cuidado con los scripts de Node:** además de los escapes unicode, un `node -e` dentro de comillas dobles de bash pierde las barras invertidas (`\d` queda como `d`). Para regex, editar con la herramienta de edición.

**Estado (07/10):** código completo, con 94 pruebas y build sin avisos (inicial de 517 kB). Falta que el usuario lo pruebe con el criterio de terminado y lo commitee. Sin SQL nuevo: se verificó por lectura que `ocupaciones` está en la publicación de Realtime y que el job `limpiar-retenciones` corre cada minuto. La sesión anónima se activa en el panel (sección 7.1, paso C).

1. La base de esta fase ya está creada (`tp1-sistema-cine.sql`, sección 5.12).
2. `OcupacionApi` y `ButacasStore`.
3. `MapaButacas` con `appHighlightButaca`, la leyenda y el precio VIP. `ContadorRetencion`.
4. Validador `butacasContiguas` y máximo de 10 butacas en `PasoButacas`, montado provisoriamente dentro de `Checkout`.
5. Sesión anónima al tocar la primera butaca.

| | |
|---|---|
| Requerimientos | RF-26 a RF-31, RF-38, RF-90, RN-04, RN-05, RN-25 (interfaz), RNF-12, RNF-20 (mapa), RNF-27 (`appHighlightButaca`), RNF-28 |

Criterio de terminado, con dos navegadores (uno anónimo y otro con un cliente):

- Las butacas que se eligen en uno aparecen al instante como retenidas en el otro.
- Si los dos intentan tomar la misma butaca, uno recibe "Otra persona tomó…".
- Pasados los 10 minutos, la butaca vuelve a libre en los dos.
- Al elegir A4 y A5 aparece el motivo "pasillo" y **Continuar** queda deshabilitado.
- El mapa se puede recorrer con el teclado.
- Pasan las pruebas de `rules/contiguidad.spec.ts`.

### F5 · Compra, pago simulado y comprobante

**Contexto para retomar (anotado el 07/10, al cerrar F4):**

- **Qué dejó F4:**
  - `publico/compra/checkout` es **provisorio**: lee `peliculaId` de la ruta y `funcionId` de la query, valida que la función sea de la película y no haya empezado, y llama a `ButacasStore.cargar(funcion, porcentajePreventa)` con `pelicula.en_preventa ? preventa_porcentaje : 0`. Ya implementa `ConCambiosPendientes` (hay butacas retenidas) y `ngOnDestroy` con `butacasStore.reiniciar()`. Sin `funcionId` muestra un enlace a la ficha; F5 lo reemplaza por `PasoFuncion` y el `@switch` de pasos.
  - `PasoButacas` emite `continuar`; hoy el checkout solo muestra un aviso. Tiene el formulario con `butacasContiguas`, `maxLength` y "al menos una", el resumen por butaca con total y el `ContadorRetencion`, que al vencer llama a `liberar()`.
  - La ficha muestra "Elegir butacas" en cada función si la película está en cartelera o en preventa.
  - `precio.ts` tiene solo `precioEntrada`; faltan `enPreventa`, `diferenciaCombo` y `calcularResumen` con sus casos de la sección 10.
  - `mensajeError` ya traduce `NO_PUEDE_COMPRAR`, `MAXIMO_BUTACAS`, `FUNCION_NO_DISPONIBLE`, `PERFIL_INCOMPLETO`, `EDAD_INSUFICIENTE` y `anonymous_provider_disabled`.
- **Siguen pendientes de antes:** Google, las cuentas de prueba, el mail de contacto de `/privacidad` y los límites de los buckets.

**Estado (07/10):** terminada y commiteada (`3421241`), con 115 pruebas y build sin avisos (inicial de 529 kB). Sin SQL nuevo: se verificó por lectura que `authenticated` puede leer `mis_compras` y ejecutar `confirmar_compra` y `comprobante`, y que `anon` ejecuta `top_peliculas`. Las dos películas cargadas son ATP: para probar la restricción por edad hay que crear una +18.

1. La base de esta fase ya está creada (`tp1-sistema-cine.sql`, sección 5.12).
2. Checkout completo, con `IndicadorPasos`. Implementa `ConCambiosPendientes` (hay butacas retenidas) y en `ngOnDestroy` llama a `reiniciar()` de `CompraStore` y `ButacasStore`:
   - `PasoFuncion`, con `SelectorFecha` y `SelectorHorario`.
   - `PasoButacas`.
   - `PasoCandy` y la parte de descuentos de `PasoPago`, visibles pero vacíos hasta F8 y F9.
   - `PasoPago`, con Signal Forms y `PagoService`.
   - `PasoComprobante`, con `VistaComprobante`, el QR, el código, la leyenda para +13 y +18, y el PDF.
3. `*appRestrictEdad` en la ficha y en el paso de función, con el motivo mostrado por el componente con `@if`, y la declaración de edad del anónimo.
4. Comprobantes guardados en el dispositivo y `/comprobantes`.
5. `MisCompras` y `DetalleCompra`, con la descarga del PDF.
6. Las tarjetas de prueba en el README.

| | |
|---|---|
| Requerimientos | RF-05, RF-13 (top con ventas), RF-32 a RF-37, RF-51 a RF-54, RN-06, RN-07, RN-12 (todavía sin puntos), RN-25 (base), RNF-15 (comprobante local), RNF-19, RNF-27 (`appRestrictEdad`) |

Criterio de terminado. Los dos primeros puntos son la **compra básica** que se repite como regresión:

- Un anónimo compra dos butacas 2D con la tarjeta `4242 4242 4242 4242` y descarga el PDF con el QR.
- La tarjeta `4000 0000 0000 0002` se rechaza y las butacas se liberan.
- Un cliente de 16 años no puede comprar una película +18, y la app dice por qué.
- En el SQL Editor no aparece ningún número de tarjeta en ninguna tabla.
- Pasan las pruebas de `rules/precio.spec.ts` y `rules/tarjeta.spec.ts`.

### F6 · Validación de QR

**Contexto para retomar (anotado el 07/10, al cerrar F5):**

- **Qué dejó F5:**
  - El checkout es un `@switch` sobre `CompraStore.paso()`, con `PasoFuncion`, `PasoButacas`, `PasoCandy` (vacío hasta F8), `PasoPago` y `PasoComprobante`. El `ContadorRetencion` vive en el checkout, arriba de los pasos, y al vencer llama a `CompraStore.vencer()`.
  - `PasoPago` tiene el lugar de "Descuentos" vacío para F9 y F10. `CompraApi.confirmar` manda solo `p_funcion_id`, `p_pago` y `p_declaracion_edad`; F8 a F13 suman sus parámetros ahí y en `SolicitudCompra`.
  - Comprobantes: `VistaComprobante` (QR con `utils/qr.ts` y PDF con `utils/comprobante-pdf.ts`, los dos con `import()`), `ComprobantesLocales`, `/comprobantes` (carga inicial), `MisCompras` y `DetalleCompra` (`/cuenta/compras` y `/cuenta/compras/:compraId`). En el menú: "Mis compras" y "Comprobantes" para el cliente, "Comprobantes" para el visitante.
  - `formatearCodigo` arma `ABCD-EFGH-JK`; `validar_qr` acepta el código con o sin guiones.
  - `angular.json` tiene `allowedCommonJsDependencies` con `qrcode` y las dependencias opcionales de `jspdf` (`html2canvas`, `canvg`, `core-js`, `raf`, `rgbcolor`). F6 suma `@zxing/library` si el build avisa.
  - `namedChunks` todavía no está: entra en F7 con la PWA.
- **Siguen pendientes de antes:** Google, las cuentas de prueba, el mail de contacto de `/privacidad` y los límites de los buckets.

1. La base de esta fase ya está creada (`tp1-sistema-cine.sql`, sección 5.12).
2. `ValidacionApi` y la pantalla `Validacion`:
   - `EscanerQr` con `@zxing/browser` e ingreso manual del código;
   - el selector de modo (acceso o candy);
   - `ResultadoValidacion` con `@switch`.
3. `NotificacionesStore` y `BandejaNotificaciones` en la navegación.

| | |
|---|---|
| Requerimientos | RF-55 a RF-60, RF-89 (bandeja), RN-16, RN-17, RNF-04 (`@switch`), RNF-12 |

Criterio de terminado, desde un celular en Vercel:

- El empleado escanea el QR de F5 en modo acceso y ve **Válido** con las butacas.
- Si lo escanea otra vez, ve **Ya usado** con la fecha y hora.
- Un código inventado da **Inexistente**.
- Si la función es mañana, da **Fuera de horario**.
- El cliente ve la notificación en la bandeja sin recargar.
- La validación aparece en `auditoria`.

**El flujo central queda completo con esta fase.**

**Estado (08/10):** commiteada (7586aba) y probada por el usuario, salvo la cámara en Vercel. Sin SQL nuevo: se verificó por lectura que `authenticated` ejecuta `validar_qr` y puede hacer UPDATE de `notificaciones.leida`, y que `notificaciones` está en la publicación de Realtime. Las compras de prueba actuales son del 08/10 y del 09/10: para ver **Válido** hay que programar una función que empiece dentro de la próxima hora y comprarla con un cliente registrado.

### F7 · PWA, avisos, recordatorios y control de lo exigido

**Contexto para retomar (anotado el 07/10, al cerrar F6):**

- **Qué dejó F6:**
  - `/empleado` (`Validacion`): selector de modo con `.opcion`, `EscanerQr`, ingreso manual con Signal Forms y `ResultadoValidacion` arriba de los dos. `escanerActivo` es falso mientras valida, con un resultado o con un error a la vista; **Validar otro** o **Probar de nuevo** lo vuelven a activar. El admin también ve "Validar" en su menú.
  - `NotificacionesStore` (raíz) y `BandejaNotificaciones` en la cabecera del cliente. El punto 3 del flujo de la sección 9 (pedir a `AvisosSistema` el aviso del sistema al llegar un INSERT) entra acá: el lugar es `aplicarCambio`, en el `case 'INSERT'`.
  - `/cuenta/notificaciones` todavía no existe: entra acá con el botón **Activar avisos**.
  - `mensajeError` traduce `SOLO_PERSONAL` y `SECTOR_INVALIDO`.
  - La cámara necesita HTTPS: en el celular se prueba en Vercel, no con `ng serve` por IP.
- **Siguen pendientes de antes:** Google, las cuentas de prueba (ahora también la de empleado, que se crea en `/admin/cuentas`), el mail de contacto de `/privacidad` y los límites de los buckets.

1. `ng add @angular/pwa` (Clase 8, lámina 20), con `namedChunks: true` en la configuración de producción de `angular.json`.
2. `ngsw-config.json` de la sección 9 y el manifest con la identidad de Cineverso.
3. Aviso de versión nueva en `app.ts` con `SwUpdate`, un signal y `@if` en `app.html` (Clase 8, lámina 23).
4. La base de esta fase ya está creada (`tp1-sistema-cine.sql`, sección 5.12).
5. `AvisosSistema` y el botón **Activar avisos** en `/cuenta/notificaciones`.

| | |
|---|---|
| Requerimientos | RF-87 y RF-88 (bandeja y aviso con la app abierta), RF-89 (aviso con la app abierta), RNF-14, RNF-15, RNF-16 (complemento con la app abierta) |

Criterio de terminado:

- **Instalación:** la app se instala desde Chrome, en Android y en escritorio.
- **Sin conexión:** con el modo avión, la cartelera se ve y en `/comprobantes` el comprobante de F5 muestra el QR y descarga el PDF.
- **Versión nueva:** al publicar un cambio aparece el aviso de versión nueva.
- **Recordatorios:** con una función creada para dentro de 24 h y 5 min, en pocos minutos aparece el recordatorio en la bandeja y, con la app abierta, como aviso del sistema. Con una para dentro de 2 h y 5 min, pasa lo mismo con el de 2 h.
- **Validación:** al validar el QR, el cliente con la app abierta recibe el aviso del sistema.
- **Precache:** en `ngsw.json`, el grupo `inicial` no tiene chunks de las áreas.

**Estado (08/10):** código completo, con 118 pruebas y build sin avisos (inicial de 546 kB: el service worker suma unos 8 kB). Falta que el usuario lo pruebe en Vercel con el criterio de terminado y lo commitee. Sin SQL nuevo: se verificó por lectura que existen los jobs `recordatorios` (cada 5 min), `alertas-venta` y `limpiar-retenciones`, y que `procesar_recordatorios` arma la `url` `/cuenta/compras/{id}`. Lo hecho:

- `ng add @angular/pwa` (22.2.1) agregó `provideServiceWorker` con `registerWhenStable:30000` y `serviceWorker` en producción. Se sumaron `namedChunks: true`, el `ngsw-config.json` de la sección 9, el manifest con `lang`, colores e íconos `any` y `maskable` (el 192 y el 512), y `theme-color` en `index.html`.
- Íconos propios (círculo violeta con un triángulo de play sobre el fondo), generados con un script de node sin dependencias. El dibujo ocupa el 68 % del ancho, así que entra en la zona segura de `maskable`.
- `ngsw.json` comprobado: `inicial` tiene `main`, los `chunk-*`, `qr` y `comprobante-pdf`, sin pantallas de áreas. `html2canvas`, `index.es` (canvg) y `purify.es` son opcionales de `jspdf` que el comprobante no usa, y quedan en `diferido`.
- `CarteleraStore.cargar`: si falla `top_peliculas` (es una RPC, no se guarda), el top queda vacío y la cartelera se muestra igual. Sin esto, el modo avión dejaba la portada en error.
- `app.ts`: `SwUpdate`, `actualizacionDisponible` y `recargar()`; `app.html` muestra "Hay una versión nueva de Cineverso." con el botón **Nueva versión: recargar**.
- `AvisosSistema` (sección 8.3), llamado desde `aplicarCambio` en el `case 'INSERT'`; si falla, la notificación igual queda en la bandeja.
- `/cuenta/notificaciones` (`Notificaciones`): estado del permiso con `@switch` (activado, bloqueado, no soportado o el botón **Activar avisos**) y los últimos avisos con "Marcar todos como leídos". La bandeja tiene el enlace "Ver todos y activar avisos".

Checklist de lo que exigió el docente:

| Punto exigido | Fase | Cómo se comprueba |
|---|---|---|
| OAuth con Google y GitHub | F1 | Entrar con cada uno en Vercel |
| Recordatorios push de 24 h y 2 h | F7 (bandeja y aviso con la app abierta); F16 si el docente exige que lleguen con la app cerrada | Ver el criterio de terminado de F7 |
| Aviso de QR validado | F6 (bandeja), F7 (aviso del sistema) | Validar y ver ambos avisos |
| Validador `butacasContiguas` | F4 | Elegir A4 y A5 |
| Pipes `duracion`, `precioArs` y `edad` | F2, F3, F1 | Ficha, precios y perfil |
| Directivas `appHighlightButaca` y `appRestrictEdad` | F4, F5 | Mapa y botón Comprar |
| Carga diferida de `/admin` y `/empleado` con guards | F1, F7 | DevTools → Network y `ngsw.json` |

### F8 · Candy y combos

**Contexto para retomar (anotado el 08/10, al cerrar F7):**

- **Qué dejó F7:** PWA instalable, `ngsw-config.json` con `inicial`, `diferido` y `recursos`, el aviso de versión nueva en `app.ts`, `AvisosSistema` y `/cuenta/notificaciones`. Toda pantalla nueva con `loadComponent` cae sola en `diferido`; si un archivo nuevo tiene que andar sin conexión, se suma a `inicial` en `ngsw-config.json`. Después de cada build, mirar `ngsw.json`.
- **README:** la sección "Arquitectura del proyecto" termina con el "Árbol de carpetas", limitado a lo que existe. Al cerrar cada fase se suman las carpetas nuevas (en F8, `admin/categorias-admin/`, `productos-admin/` y `combos-admin/`) y se cambia "Estado al cierre de F7".
- **Para F8:** `PasoCandy` está vacío desde F5. `ResultadoValidacion` ya muestra la lista de productos que devuelve `validar_qr`, y `sin_candy` ya tiene su texto. `CompraApi.confirmar` y `SolicitudCompra` suman los parámetros del candy.
- **Siguen pendientes de antes:** Google, las cuentas de prueba (incluida la de empleado, que se crea en `/admin/cuentas`), el mail de contacto de `/privacidad` y los límites de los buckets. La cámara del escáner se prueba en Vercel.

1. La base y el bucket `productos` ya están creados (F0).
2. `CandyApi`, `CategoriasAdmin`, `ProductosAdmin` y `CombosAdmin`.
3. `PasoCandy`, con los productos agrupados por categoría y los combos destacados.
4. La validación en modo candy muestra los productos y combos a entregar.

| | |
|---|---|
| Requerimientos | RF-46 a RF-50, RN-14, RN-15, S-11, S-12 |

Criterio de terminado:

- Se compran 2 entradas 3D (una VIP), un combo y un pochoclo. El resumen muestra la diferencia del combo, y la compra guarda los tres subtotales.
- En la puerta se valida el acceso y en el candy se entregan los productos. Con las dos partes usadas, el QR queda inválido.
- Regresión: la compra básica de F5 y la validación de F6.

**Estado (08/10):** código completo, con 130 pruebas y build sin avisos (inicial de 547 kB). Falta que el usuario lo pruebe con el criterio de terminado y lo commitee. Sin SQL nuevo: se verificó por lectura que `confirmar_compra` ya recibe `p_productos` y `p_combos`, que `validar_qr` devuelve los productos y combos, que las tablas de candy se leen sin sesión y solo el admin las modifica, y que existe el bucket `productos`. Las tablas de candy estaban vacías: hay que cargar categorías, productos y combos desde el admin antes de probar. Lo hecho:

- `CandyApi`, `candy.model.ts` y la regla `candy.ts` con su spec; `entradasConCombos` en `precio.ts`.
- `CategoriasAdmin`, `ProductosAdmin` (foto en `productos/{id}/`) y `CombosAdmin` (foto en `productos/combos/{id}/`, productos con `SelectorProductos`), con lista y formulario en la misma pantalla y `pendingChangesGuard`. Las tres pantallas tienen arriba una botonera `.secciones` (estilo global) para pasar de una a otra; en el menú del admin, "Candy" lleva a `/admin/productos`. Borrar algo vendido falla por la FK y el mensaje sugiere marcarlo como no disponible o inactivo.
- `ButacasStore.diferencias` y, en `CompraStore`, el catálogo, lo elegido, `entradas`, las líneas, `combosDeMas` y el resumen con candy y combos. `confirmar` manda `productos` y `combos`.
- `PasoCandy`: combos destacados arriba (fondo violeta), productos por categoría, botones − y +, y "Tu pedido" con cada entrada ("con combo" y su diferencia), las líneas y el resumen. El + de combos se apaga con un combo por entrada; si al volver a las butacas quedan más combos que entradas, aparece un aviso y **Continuar** se deshabilita. Sin nada elegido, el botón dice "Seguir sin candy".
- `PasoPago` muestra el candy elegido. El comprobante y el PDF suman "con {combo}" en la entrada cubierta. `mensajeError` traduce `PRODUCTO_NO_DISPONIBLE` y `COMBOS_SUPERAN_ENTRADAS`.

### F9 · Cupones

**Contexto para retomar (anotado el 08/10, al cerrar F8):**

- **Qué dejó F8:** `CompraStore.resumen` ya suma entradas (con las diferencias de combo), candy y combos, y pasa `porcentajeCupon: 0` y `credito: 0` a `calcularResumen`; F9 cambia el primero. `SolicitudCompra` tiene `productos` y `combos`; `CompraApi.confirmar` suma `p_cupon` en F9. `PasoPago` tiene el lugar de "Descuentos" vacío. `reiniciar()` vacía también el candy.
- **README:** en el árbol de carpetas se suma `admin/cupones-admin/` y se cambia "Estado al cierre de F8".
- **Siguen pendientes de antes:** Google, las cuentas de prueba (incluida la de empleado, que se crea en `/admin/cuentas`), el mail de contacto de `/privacidad` y los límites de los buckets. La cámara del escáner se prueba en Vercel.

1. La base de esta fase ya está creada (`tp1-sistema-cine.sql`, sección 5.12).
2. `CuponApi`, `CuponesAdmin` y el porcentaje de primera compra en `PreciosAdmin`.
3. La parte de descuentos de `PasoPago`, con la sugerencia del mejor cupón.
4. El perfil muestra el cupón de bienvenida mientras no se use.

| | |
|---|---|
| Requerimientos | RF-03, RF-40 a RF-43, RN-09 a RN-11, S-06, S-23 |

Criterio de terminado:

- Un cliente nuevo ve sugerido el cupón del 20 % y lo usa. En la segunda compra ya no aparece.
- Un cupón por edad de 50 años se rechaza, con su motivo, para alguien de 49.
- Pasan las pruebas de `rules/cupones.spec.ts`.
- Regresión: la compra básica de F5.

**Estado (08/10):** commiteada (8282f60), con 149 pruebas (19 nuevas en `cupones.spec.ts`) y build sin avisos (inicial de 548 kB). Sin SQL nuevo: se verificó por lectura que existe `BIENVENIDA` (20 %), que `cupones_sugeridos` y `validar_cupon` son `security definer` y solo las ejecuta `authenticated`, que `motivo_cupon` no se puede llamar desde la API y que la tabla `cupones` solo la toca el admin. Para el criterio de edad hay que crear antes, desde `/admin/cupones`, un cupón por edad de 50 años. Lo hecho:

- `CuponApi` (`sugeridos`, `validar`, `listar`, `primeraCompra`, `crear`, `actualizar`, `eliminar`), `cupon.model.ts` y la regla `cupones.ts` con su spec. `mensajeError` traduce los motivos de `motivo_cupon`, `CUPON_INEXISTENTE` y `cupones_codigo_key`.
- `CuponesAdmin` en `/admin/cupones` (menú "Cupones"), con lista y formulario: código en mayúsculas, porcentaje de 1 a 100, tipo general o por edad (con edad mínima, 50 por defecto), vigencia opcional en dd/mm/aaaa, límite de usos opcional (no menor que los usos que ya tiene) y activo. La lista muestra el estado de hoy con `estadoCupon`. `BIENVENIDA` aparece aparte, con un enlace a Precios; borrar un cupón usado falla por la FK y el mensaje sugiere desactivarlo.
- `PreciosAdmin` suma "Cupón de primera compra (%)", que actualiza la fila de `BIENVENIDA`.
- `CompraStore`: `cupon`, `errorCupon`, `validandoCupon`, `subtotal` y `cuponSugerido` (con `mejorCupon`, solo si no hay otro aplicado); `cargarCupones()` (una vez por compra, solo registrados), `aplicarCupon(codigo)` (valida con `validar_cupon` y reemplaza al anterior) y `quitarCupon()`. `resumen` usa el porcentaje aplicado, `confirmar` manda `cupon` y, si la base rechaza el cupón al pagar, lo saca y muestra el motivo. `reiniciar()` lo vacía.
- `PasoPago`, en "Descuentos": para el anónimo, un aviso; para el cliente, el cupón sugerido con **Aplicar**, un campo para tipear otro código y, si hay uno aplicado, **Quitar**.
- `Perfil` muestra "Tus cupones" (los de `cupones_sugeridos`) solo si hay alguno: `BIENVENIDA` desaparece después de la primera compra.
- El comprobante en pantalla y el PDF muestran "Descuento (cupón X)".

### F10 · Cancelaciones y crédito

**Contexto para retomar (anotado el 08/10, al cerrar F9):**

- **Qué dejó F9:** `CompraStore.resumen` pasa el porcentaje del cupón aplicado y `credito: 0` a `calcularResumen`; F10 cambia el segundo. `SolicitudCompra` tiene `cupon`; `CompraApi.confirmar` suma `p_credito` en F10. En `PasoPago`, "Descuentos" ya tiene el cupón: el crédito va debajo, y `ResumenCompra` ya muestra "Crédito a favor" y "A pagar con tarjeta" cuando `credito_usado > 0`. Si el crédito cubre todo, la base no pide tarjeta (`p_pago` puede ir nulo). `Perfil` ya tiene la sección "Tus cupones"; el crédito se suma como otra sección.
- **README:** se cambia "Estado al cierre de F9" y la fila de F10 en la tabla de estado (no hay carpetas nuevas).
- **Siguen pendientes de antes:** Google, las cuentas de prueba (incluida la de empleado, que se crea en `/admin/cuentas`), el mail de contacto de `/privacidad` y los límites de los buckets. La cámara del escáner se prueba en Vercel.

1. La base de esta fase ya está creada (`tp1-sistema-cine.sql`, sección 5.12).
2. El botón **Cancelar** en `DetalleCompra`, el crédito en `Perfil` y su uso en `PasoPago`.

| | |
|---|---|
| Requerimientos | RF-04 (crédito), RF-44, RF-73 a RF-76, RN-18, S-20 |

Criterio de terminado:

- **Cancelación a más de 2 h:** al cancelar una compra de $ 16.000 quedan $ 16.000 de crédito, las butacas se liberan en el mapa de otro usuario y el cupón usado no vuelve.
- **Uso del crédito:** otra compra usa $ 5.000 de crédito y paga el resto con tarjeta.
- **Fuera de plazo:** a menos de 2 h, el botón explica por qué no se puede cancelar.
- **Regresión:** la compra básica de F5.

**Estado (08/10):** commiteada (`fb1be95`), con 156 pruebas (7 nuevas en `cancelacion.spec.ts`) y build sin avisos (inicial de 548 kB). Sin SQL nuevo: se verificó por lectura que `cancelar_compra` es `security definer` y solo la ejecuta `authenticated`, que `confirmar_compra` lanza `CREDITO_INSUFICIENTE` si se pide más que el saldo y acepta `p_pago` nulo cuando no queda resto, y que `creditos_movimientos` y `saldos_credito` solo dejan leer lo propio. Lo hecho:

- `FidelizacionApi` (`saldoCredito` y `movimientosCredito`), `fidelizacion.model.ts` con `MovimientoCredito`, `CompraApi.cancelar` y la regla `cancelacion.ts` con su spec. `mensajeError` traduce `CREDITO_INSUFICIENTE`, `COMPRA_INEXISTENTE`, `COMPRA_YA_CANCELADA`, `FUERA_DE_PLAZO` y `COMPRA_USADA`.
- `DetalleCompra`: si la compra está vigente, una sección "Cancelar la compra" con el plazo (de `configuracion.horas_limite_cancelacion`) o, si no se puede, el botón deshabilitado con el motivo de `motivoNoCancelable`. Pide confirmación (menciona que el cupón no se devuelve), muestra el crédito acreditado y vuelve a traer el comprobante, que queda guardado como cancelado.
- `CompraStore`: `saldoCredito`, `credito`, `creditoDisponible`, `cargarCredito()`, `usarCredito(monto)` y `quitarCredito()`. `resumen` resta el crédito elegido y `confirmar` manda `credito_usado` (nunca más que el total). Si la base responde `CREDITO_INSUFICIENTE`, saca el crédito y vuelve a traer el saldo.
- `PasoPago`: debajo del cupón, "Crédito a favor" (solo si hay saldo) con **Usar** todo, un campo para usar una parte y **Quitar**. Si no queda nada para pagar, en lugar de la tarjeta muestra **Confirmar compra**, que manda `p_pago` nulo.
- `Perfil`: sección "Tu crédito" con el saldo y los movimientos (cada uno enlaza a su compra).

### F11 · Reportes y auditoría

**Contexto para retomar (anotado el 08/10, al cerrar F10):**

- **Qué dejó F10:** la cancelación escribe en `auditoria` la acción `cancelacion` (con `estado` y `credito`), que es la que pide filtrar el criterio de terminado. Las compras canceladas quedan con `estado = 'cancelada'`: el reporte no las cuenta.
- **Dependencia nueva:** `write-excel-file` todavía no está instalada (sección 8.11); `jspdf` ya está y se carga con `import()` como en `comprobante-pdf.ts`.
- **README:** se cambia "Estado al cierre de F10", la fila de F11 y el árbol (carpetas nuevas de `admin/`).
- **Siguen pendientes de antes:** Google, las cuentas de prueba (incluida la de empleado, que se crea en `/admin/cuentas`), el mail de contacto de `/privacidad` y los límites de los buckets. La cámara del escáner se prueba en Vercel.

1. La base de esta fase ya está creada (`tp1-sistema-cine.sql`, sección 5.12).
2. `ReporteApi` y la pantalla `Reportes`: rango de fechas, tabla con las tres líneas (entradas, candy y combos), `GraficoBarras` y exportación a PDF y Excel.
3. `AuditoriaApi` y la pantalla `Auditoria`, con filtros por usuario, acción y rango.
4. El panel del admin completo.

| | |
|---|---|
| Requerimientos | RF-77, RF-79 a RF-85, RN-23, RN-24, S-24, D-13 |

Criterio de terminado:

- El reporte del día cuadra con las compras de las fases anteriores, en sus tres líneas; las compras canceladas no cuentan.
- El Excel se abre en Excel o en LibreOffice.
- El log se puede filtrar por la acción "cancelacion" y por un usuario.

**Estado (09/10):** commiteada (`ebbcd8f`), con 179 pruebas (23 nuevas en `reportes.spec.ts` y `auditoria.spec.ts`) y build sin avisos (inicial de 549 kB). Sin SQL nuevo: se verificó por lectura que las tres RPC de reporte son `security definer`, exigen `es_admin()`, solo cuentan `estado = 'confirmada'` y no las ejecuta `anon`, y que `auditoria` solo tiene la política de lectura del admin. Para el día 08/10 la base tiene 10 compras vigentes (entradas $ 192.000, candy $ 114.000, combos $ 120.000, descuentos $ 38.250, total $ 387.750) y una cancelada que no cuenta. Lo hecho:

- `write-excel-file` 4.1.1 instalada; se importa de `write-excel-file/browser`.
- `reporte.model.ts` y `auditoria.model.ts` (sección 8.2).
- Reglas `reportes.ts` y `auditoria.ts` con sus specs (sección 8.5).
- `ReporteApi` (las tres RPC, con los `numeric` pasados a número) y `AuditoriaApi.buscar` (hasta 200 registros, del más nuevo al más viejo, con `perfiles(nombre, apellido, email)`; el usuario se busca por una parte del email en `perfiles` y se filtra con `in('usuario_id', ids)`).
- `GraficoBarras` en `shared/components/`: barras con `@for` y `[style.width.%]`, ocultas para el lector (`aria-hidden`), y una tabla con la clase global `.solo-lectores` (nueva en `styles.css`).
- `Reportes` en `/admin/reportes`: atajos Hoy, Últimos 7 días y Este mes, rango en dd/mm/aaaa con Signal Forms (hoy por defecto) y tres secciones: la tabla de facturación por día con la fila de totales y **Exportar PDF** y **Exportar Excel** (`reporte-pdf.ts` y `excel.ts` con `import()`), las más vistas por semana o por mes (un gráfico por período) y el candy más vendido.
- `Auditoria` en `/admin/auditoria`: filtros opcionales por email, acción y rango; cada registro muestra la acción, la fila afectada, la fecha y quién actuó ("Sistema" si no hay usuario) con **Ver solo sus acciones**, y en "Ver datos" los campos (en un cambio, solo los que cambiaron, con el valor anterior tachado).
- Menú del admin completo salvo Recompensas, que entra en F13: se sumaron Reportes y Auditoría. `mensajeError` traduce `PERIODO_INVALIDO`.
- El build saca `jspdf` a un `chunk-*.js` compartido por `comprobante-pdf` y `reporte-pdf`, que el grupo `inicial` del service worker ya precachea (sección 9).

### F12 · Reseñas y Mis películas

**Contexto para retomar (anotado el 08/10, al cerrar F11):**

- **Qué ya existe en la base:** la tabla `resenas` (`id`, `pelicula_id`, `usuario_id`, `autor`, `estrellas`, `comentario`, `creada_en`, `editada_en`) con lectura para todos, alta solo con `vio_pelicula(pelicula_id)` y cambio solo de la propia; `autor` lo completa el trigger `completar_autor_resena`. La vista `mis_peliculas` devuelve `pelicula_id`, `nombre`, `poster_url`, `vista_en`, `resena_id` y `estrellas`. `peliculas_cartelera` ya trae `promedio` y `cantidad_resenas`, y `PeliculaCartelera` ya los tiene.
- **Qué falta en Angular:** `resena.model.ts`, `ResenaApi`, el componente `Estrellas` (con `model()`, como `SelectorGeneros`), la ruta `/cuenta/peliculas` con `MisPeliculas` y "Mis películas" en el menú del cliente.
- **README:** se cambia "Estado al cierre de F11", la fila de F12 y el árbol (`cliente/mis-peliculas/`).
- **Siguen pendientes de antes:** Google, las cuentas de prueba (incluida la de empleado, que se crea en `/admin/cuentas`), el mail de contacto de `/privacidad` y los límites de los buckets. La cámara del escáner se prueba en Vercel.

1. La base de esta fase ya está creada (`tp1-sistema-cine.sql`, sección 5.12).
2. `ResenaApi`, `Estrellas`, las reseñas en la ficha y `MisPeliculas` con acceso directo a reseñar.

| | |
|---|---|
| Requerimientos | RF-15 (reseñas), RF-61 a RF-63, RF-72, RN-21, RN-22, S-13, S-14 |

Criterio de terminado:

- Después de validar el acceso, la película aparece en Mis películas y se puede reseñar.
- Sin haberla visto, el INSERT falla por RLS.
- La ficha y el listado muestran el promedio.

**Estado (09/10):** código hecho; falta que el usuario lo pruebe y lo commitee. Sin SQL nuevo (se verificó por lectura la tabla, el RLS, el `unique`, el trigger del autor y la vista). Lo hecho:

- `resena.model.ts` (`ResenaModel`, `ResenaDraft`, `MiPeliculaModel` y `crearResenaVacia`) y `ResenaApi` (`dePelicula`, `mias`, `guardar` y `misPeliculas`). Se agregó `mias(usuarioId)` respecto de la sección 8, para precargar el comentario al editar: la vista solo trae `resena_id` y `estrellas`.
- `rules/resena.ts`: `textoPromedio`, `textoCantidadResenas` y `estrellasLlenas`, con 5 pruebas.
- `Estrellas` (`app-estrellas`): `soloLectura = input(false)` y `value = model<number>(0)`. Editable son 5 botones con `aria-pressed`; solo lectura es un `role="img"` con "n de 5 estrellas".
- `MisPeliculas` en `/cuenta/peliculas` (`clienteGuard` y `pendingChangesGuard`), con "Mis películas" en el menú de la cuenta. La reseña se escribe en línea, una a la vez. Cada película muestra las estrellas y el comentario propios (pedido del usuario), y arriba dice "Historial de películas que viste. Calificá tu experiencia y compartí tu opinión.".
- La ficha suma "Calificación" en los detalles (si hay reseñas) y la sección Reseñas. Al cliente le indica que reseñe desde Mis películas.
- `TarjetaPelicula` muestra "★ 4,5" al principio de la línea de detalles.
- `mensajeError` traduce `resenas_pelicula_id_usuario_id_key` y el código `42501`.
- Build sin avisos (inicial de 563.73 kB) y 188 pruebas en verde.

### F13 · Puntos y recompensas

1. La base de esta fase ya está creada (`tp1-sistema-cine.sql`, sección 5.12).
2. `FidelizacionApi`, `RecompensasAdmin`, los canjes en `PasoPago` y los puntos en `Perfil` (disponibles, pendientes, movimientos y canjes).

| | |
|---|---|
| Requerimientos | RF-04 (puntos y canjes), RF-36 (puntos a ganar), RF-45, RF-64 a RF-67, RN-12, RN-13, RN-18 (puntos), RN-19, S-07 |

Criterio de terminado:

- Una compra de $ 16.000 deja 16.000 puntos pendientes, que pasan a disponibles cuando empieza la función.
- Canjear un pochoclo resta los puntos y no suma puntos.
- Al cancelar una compra, se anulan los puntos pendientes y se devuelven los canjeados.
- Pasan las pruebas de `rules/puntos.spec.ts`.
- Regresión: la compra básica de F5 y la cancelación de F10.

**Estado (09/10), dividida en dos entregas.** Sin SQL nuevo: `confirmar_compra` ya suma los puntos desde F5 y acepta `p_recompensas`, `cancelar_compra` ya los anula, y `recompensas` tiene la política `ALL` del admin y lectura para todos (se verificó por lectura).

- **F13 (puntos), código hecho:**
  - `rules/puntos.ts` (`puntosGanados` y `saldoPuntos`) con 5 pruebas.
  - Modelos de movimientos de puntos y de recompensas en `fidelizacion.model.ts`.
  - `FidelizacionApi` suma `movimientosPuntos`, `recompensas` y el ABM.
  - `Perfil` muestra "Tus puntos": los disponibles, los pendientes, los movimientos con enlace a la compra y la fecha en que se liberan, y el catálogo activo.
  - `RecompensasAdmin` en `/admin/recompensas` (en el menú Ventas): producto del candy o entrada 2D estándar, costo en puntos enteros y activa. El producto se valida solo si el tipo es producto, leyendo `borrador().tipo`.
  - Build sin avisos (inicial de 563.92 kB) y 194 pruebas en verde.
- **F13 (canje), código hecho** (el usuario decidió completar F13 antes de la entrega):
  - `SolicitudCompra.recompensas` y `CompraApi.confirmar` mandan `p_recompensas`, con un elemento por canje.
  - `CompraStore` carga una vez los puntos disponibles y las recompensas activas (`cargarPuntos`). Tiene `canjes` (ids), `puntosCanje`, `puntosRestantes`, `entradasCanjeadas`, `lineasCanjes`, `puntosAGanar`, `puedeCanjear`, `agregarCanje` y `quitarCanje`.
  - `entradasConCombos` recibe `canjes` (4.º parámetro, 0 por defecto): las entradas canjeadas cubren butacas después de los combos y pagan la diferencia, como el paso 7 de `confirmar_compra`. Suma 2 pruebas.
  - `agregarCombo` descuenta las butacas ya cubiertas por un canje.
  - Si la base rechaza el canje (`PUNTOS_INSUFICIENTES`, `RECOMPENSA_NO_DISPONIBLE` o `RECOMPENSAS_SUPERAN_ENTRADAS`), se sacan los canjes y se vuelven a traer los puntos. `mensajeError` traduce los tres.
  - `PasoPago` suma "Canjeá tus puntos" con contadores − y +, la línea "Canjes" en el resumen y "Con esta compra sumás N puntos" (RF-36).
  - El comprobante (pantalla y PDF) marca las entradas y el candy canjeados, y muestra los puntos canjeados y los ganados.
  - Build sin avisos (inicial de 565.27 kB) y 196 pruebas en verde.
  - **Regresión a repetir:** la compra básica de F5 y la cancelación de F10.

### F14 · Preventa y alertas

1. La base de esta fase ya está creada (`tp1-sistema-cine.sql`). La preventa ya entra en `precio_entrada` desde F5.
2. `AlertaApi`, el botón de alerta en `TarjetaPelicula` y en la ficha, y la sección de preventa en `PeliculaForm`.

| | |
|---|---|
| Requerimientos | RF-68 a RF-71, RN-08, S-09, S-10 |

Criterio de terminado:

- Un cliente activa la alerta de una película próxima.
- El admin activa la preventa con un 15 % y una apertura que ya pasó. En 10 minutos llega la notificación a la bandeja y, con la app abierta, el aviso del sistema.
- La compra en preventa tiene el descuento solo sobre el precio base.
- Con el estreno cambiado al día de hoy, el precio vuelve al normal.

**Estado (09/10), F14 recortada:** cerrada (`e3cff7c`). Probada en Vercel: el aviso de *Los Increíbles* llegó a la bandeja a las 17:20, y el push con la app cerrada anda desde el arreglo del secreto de las 18:54 (ver F16). Build sin avisos (inicial de 562 kB) y 179 pruebas en verde. Sin SQL nuevo: se verificó por lectura que `alertas` deja leer y borrar las propias, y crear solo con `rol_actual() = 'cliente'` y la película en `proximamente`, y que `peliculas_cartelera` trae `alertas_enviadas_en`. Lo hecho:

- `AlertaApi` (`mias`, `activar(usuarioId, peliculaId)` y `desactivar`).
- `AlertasStore` en `root`: los ids con alerta, cargados una vez cuando entra un cliente (mismo `effect` que `NotificacionesStore`).
- `BotonAlerta` (`app-boton-alerta`, `pelicula = input.required<PeliculaCartelera>()`), debajo de cada tarjeta de Próximamente y en la ficha. Se muestra solo si la película está en `proximamente`, sin preventa abierta y con `alertas_enviadas_en` nulo. Sin sesión, "Iniciá sesión" con `volver`; con otro rol, nada. Con la alerta activa y sin permiso de avisos, enlaza a `/cuenta/notificaciones`.
- **Cambio respecto de la sección 8.6:** el botón no va dentro de `TarjetaPelicula` (`conAlerta` y `alertaCambiada` no se agregaron), porque la tarjeta entera es un enlace y no puede contener un botón (sección 2.2).
- ~~**Pendiente:** la sección de preventa en `PeliculaForm` (RF-70).~~ Hecha el 09/10: ver "Estado de F14 (preventa)" más abajo.

Criterio de terminado de la versión recortada: un cliente activa la alerta de una película de Próximamente; el admin le crea una función futura y la pasa a cartelera; en hasta 10 minutos llega el aviso a la bandeja y, con los avisos activados, también con la app cerrada. Al tocarlo se abre la ficha.

**Estado de F14 (preventa), 09/10:** cerrada (`acd19fc`) y probada: alerta por el camino de la preventa, precio con 15 % sobre la base y vuelta al precio normal con el estreno en el día. Sin SQL nuevo: se verificó por lectura que el admin escribe `peliculas` con la política `ALL` y que la tabla ya tiene los checks de 0 a 100 y de 1 a 60. Lo hecho:

- `PeliculaDraft` y `PeliculaFila` suman `preventa_activa`, `preventa_porcentaje` y `preventa_dias`; `crearPeliculaVacia` arranca en `false`, `0` y `7`, como los valores por defecto de la base.
- `borradorDesdePelicula` y `filaDesdeBorrador` las pasan, y la nueva `aperturaPreventa(estreno, dias)` en `rules/pelicula.ts` devuelve el día de apertura (o `null`).
- `PeliculaForm` tiene la sección Preventa: la casilla, el descuento y los días, con `required`, `min`, `max` y un `validate` de días enteros. Un `computed` muestra "Abre el dd/mm/aaaa a las 00:00 y cierra el día del estreno".
- Los campos siempre están visibles y validados, aunque la casilla esté apagada: los valores se guardan igual y así no queda un campo oculto inválido que trabe el envío.
- Build sin avisos (inicial de 562.82 kB) y 183 pruebas en verde (4 nuevas en `pelicula.spec.ts`).

### F15 · Cierre (12/10, sin funciones nuevas)

1. Probar la app desplegada de punta a punta con los usuarios de prueba de cada rol.
2. Corregir lo que aparezca. Solo correcciones, sin funciones nuevas.
3. Revisar accesibilidad (contraste, teclado y lector de pantalla en el mapa) y responsive (celular, tablet y escritorio).
4. Aplicar la paleta y la tipografía definitivas en los tokens, si ya están definidas.
5. Cerrar el README con:
   - arquitectura y decisiones;
   - configuración y lo que tiene la base;
   - la limitación del correo de recuperación;
   - las tarjetas de prueba y los roles de las cuentas de prueba (las claves van por mail al docente);
   - capturas.
6. Revisar la tabla de la sección 12 contra la app desplegada.

| | |
|---|---|
| Requerimientos | RNF-17 a RNF-20, RNF-25 |

Criterio de terminado: cada fila de la sección 12 está comprobada en Vercel.

### F16 · Push con la app cerrada (obligatoria desde el 09/10)

La cátedra la exigió el 09/10 (D-25). Diseño completo en la sección 9, "Push con la app cerrada".

Orden:

1. **Prueba aislada:** la función `enviar-push` desplegada desde el panel, con sus secretos, envía a un solo dispositivo. La suscripción se saca desde la consola del navegador en Vercel. Si `npm:web-push` falla en el runtime de Supabase, se cambia por una librería pensada para Deno (`jsr:@negrel/webpush`) sin cambiar el diseño.
2. `03_push.sql`: `pg_net`, `suscripciones_push` con RLS, el secreto en Vault y los dos triggers. `04_quita_demo_avisos.sql` saca la herramienta de demo que se había sumado.
3. Front: la clave pública en los environments, `PushApi`, la suscripción en **Activar avisos** (y al entrar, si el permiso ya estaba dado) y la baja al cerrar sesión.
4. F14 recortada: el botón de alerta en Próximamente y en la ficha (RF-69).

| | |
|---|---|
| Requerimientos | RF-69, RF-87 a RF-89 y RNF-16, en su versión con la app cerrada |
| Terminado | Con la app cerrada, en Android, PC y iPhone con la PWA instalada, llegan los recordatorios de 24 h y 2 h (con película, sala, hora y acceso al QR), el QR validado (con sector, fecha y hora) y la alerta de venta. Al tocar cada aviso se abre su pantalla |

**Estado (09/10):** terminada. **Corrección de las 18:54:** la prueba de la madrugada en Chrome con Windows no había probado el push. `enviar-push` respondía 401 a todo porque `PUSH_SECRETO` no coincidía con `push_secreto` de Vault, y los avisos que se veían eran del canal 2 (Realtime con la app abierta o en segundo plano). Se detectó porque la alerta de venta, con la app cerrada de verdad, no llegó. Se arregló pegando en `PUSH_SECRETO` el valor de Vault, sin redesplegar. Después, un QR validado con la app cerrada llegó en Android: la función respondió 200 y FCM, 201 para los dos dispositivos. **Cómo verificar un envío:** `select status_code, content from net._http_response order by created desc`. Commits: `9ba9bb7` (push) y `cdb816e` (la barra nueva, que también quita la herramienta de demo). SQL aplicado: `03_push.sql` y `04_quita_demo_avisos.sql`. Edge Function `enviar-push` desplegada desde el panel con `verify_jwt` apagado y los secretos `VAPID_PUBLIC_KEY`, `VAPID_PRIVATE_KEY`, `VAPID_SUBJECT` y `PUSH_SECRETO`; el secreto `push_secreto` está en Vault. La prueba aislada con `npm:web-push` respondió 201 (runtime de Supabase compatible con Deno 2.1.4), así que no hizo falta otra librería. Falta probar en Android y en iPhone con la PWA instalada.

Plan de demo, solo con los caminos reales:

| Dispositivo | Cuenta | Para qué |
|---|---|---|
| Celular del profe (Android, o iPhone con la PWA instalada) | Cliente propio, registrado en el momento | Recibe los avisos |
| PC del profe (opcional) | La misma cuenta de cliente | Muestra que llega a todos sus dispositivos |
| Notebook del autor | Admin, con "Acceso para la corrección" | Crea las funciones de la demo |
| Celular del autor | Empleado | Escanea el QR del profe |

1. Al empezar: el admin crea una función que empieza dentro de 24 h y 5 min y otra dentro de 2 h y 5 min.
2. El profe se registra, toca **Activar avisos** y acepta (en iPhone, dentro de la PWA instalada), y compra una entrada para cada función.
3. Cierra la app del todo. En 5 a 10 minutos llegan los dos recordatorios; al tocarlos se abre la compra con el QR.
4. El empleado escanea un QR en modo acceso: llega el aviso con sector, fecha y hora.
5. Alerta de venta (con F14): el profe activa la alerta de una película de Próximamente, el admin le crea una función y la pasa a cartelera, y en hasta 10 minutos llega la alerta.
6. El profe abre la app y ve los avisos en la bandeja (canal garantizado).

---

## 12. Trazabilidad

### 12.1 Requerimientos funcionales

| ID | Requerimiento | Sección | Fase |
|---|---|---|---|
| RF-01 | Registro con email, Google o GitHub, con los datos personales | 5.2, 7.1, 8.10 | F1 |
| RF-02 | Inicio y cierre de sesión, recuperación de contraseña | 7.1, 8.10 | F1 |
| RF-03 | Cupón de primera compra al registrarse | 5.8, 6.2.9 | F9 |
| RF-04 | Perfil con datos, puntos, canjes y crédito | 4, 8.4, 8.7 | F1, F10, F13 |
| RF-05 | Historial de compras con estado y PDF | 5.6, 6.2.9, 8.11 | F5 |
| RF-06 | Cuentas internas creadas por un admin | 5.2, 7.4 | F1 |
| RF-07 | Edad calculada a partir de la fecha de nacimiento | 6.1, 8.5, 8.7 | F1 |
| RF-08 | ABM de películas con póster | 5.3, 7.3 | F2 |
| RF-09 | Géneros administrables | 5.3 | F2 |
| RF-10 | Clasificación ATP, +13 y +18 | 5.3 | F2 |
| RF-11 | Estado de la película | 5.3 | F2 |
| RF-12 | Formato e idioma por función | 5.5 | F3 |
| RF-13 | Top 3 en la página principal | 6.2.9 | F2, F5 |
| RF-14 | Buscador y filtros | 5.3, 8.4 | F2 |
| RF-15 | Ficha con funciones y reseñas | 4, 8.7 | F2, F3, F12 |
| RF-16 | ABM de salas | 5.4 | F3 |
| RF-17 | Plantilla común con bloques y pasillos | 5.4 | F3 |
| RF-18 | Filas J y K accesibles | 5.4 | F3 |
| RF-19 | Filas R, S y T VIP | 5.4 | F3 |
| RF-20 | Distribución por sala | 5.4 | F3 |
| RF-21 | ABM de funciones con fin calculado | 5.5, 6.2.2, 6.2.3 | F3 |
| RF-22 | Programación recurrente | 6.2.2, 8.5, 8.10 | F3 |
| RF-23 | Asignación automática de sala | 6.2.1 | F3 |
| RF-24 | Vista previa de la asignación | 6.2.2, 8.5 | F3 |
| RF-25 | Función con ventas no se borra ni cambia de horario | 5.5, 6.3 | F3 |
| RF-26 | Mapa interactivo | 8.6 | F4 |
| RF-27 | Estados de cada butaca | 8.4, 8.8 | F4 |
| RF-28 | Realtime en el mapa | 7.2 | F4 |
| RF-29 | Retención con contador | 6.2.4, 8.6 | F4 |
| RF-30 | Accesibles y VIP por color e ícono, con leyenda | 8.6, 8.8 | F4 |
| RF-31 | Precio VIP y resumen por butaca | 8.5, 8.6 | F4, F5 |
| RF-32 | Flujo guiado con indicador de paso | 8.4, 8.5, 8.6 | F5, F8, F9 |
| RF-33 | Compra registrada o anónima | 1.2, 7.1 | F4, F5 |
| RF-34 | Chips de días y horarios | 8.6 | F5 |
| RF-35 | Restricción por edad | 6.1, 8.8 | F5 |
| RF-36 | Resumen con desglose | 8.4, 8.5 | F5, F8, F9, F10, F13 |
| RF-37 | Pago simulado en el servidor | 6.2.6, 8.3 | F5 |
| RF-38 | Máximo de 10 butacas | 5.1, 6.2.4, 8.9 | F4 |
| RF-39 | Precio base por formato y recargo VIP | 5.1 | F3 |
| RF-40 | ABM de cupones | 5.8, 8.10 | F9 |
| RF-41 | Cupón de primera compra editable | 5.8 | F9 |
| RF-42 | Cupones para mayores de 50 | 5.8 | F9 |
| RF-43 | Aplicar cupón y sugerir el mejor | 6.2.9, 8.5 | F9 |
| RF-44 | Uso de crédito total o parcial | 6.2.6 | F10 |
| RF-45 | Canje de puntos en la compra | 5.8, 6.2.6 | F13 |
| RF-46 | ABM de categorías y productos | 5.7, 7.3 | F8 |
| RF-47 | Productos agrupados por categoría | 8.4 | F8 |
| RF-48 | ABM de combos | 5.7 | F8 |
| RF-49 | Combos destacados | 11 (F8) | F8 |
| RF-50 | Retiro de productos con el mismo QR | 6.2.7 | F8 |
| RF-51 | PDF del comprobante | 8.11 | F5 |
| RF-52 | Leyenda +13 y +18 | 8.11 | F5 |
| RF-53 | Un único QR por compra | 5.6 | F5 |
| RF-54 | Código aleatorio y código corto | 5.6, 6.1 | F5 |
| RF-55 | Pantalla de empleado con escáner | 4, 8.6 | F6 |
| RF-56 | Ingreso manual del código | 6.2.7 | F6 |
| RF-57 | Dos modos de validación | 6.2.7 | F6 |
| RF-58 | Resultado grande y claro | 6.2.7, 8.5 | F6 |
| RF-59 | Una parte usada no vuelve a validar | 6.2.7 | F6 |
| RF-60 | Validación en la auditoría | 6.2.7 | F6 |
| RF-61 | Calificación de 1 a 5 y comentario | 5.9 | F12 |
| RF-62 | Promedio y cantidad de reseñas | 5.3 | F12 |
| RF-63 | Una reseña por usuario, editable | 5.9 | F12 |
| RF-64 | 1 punto por peso, pendiente (RN-12, RN-19) | 5.8 | F13 |
| RF-65 | Catálogo de recompensas | 5.8 | F13 |
| RF-66 | Saldo y movimientos de puntos | 5.8, 8.5 | F13 |
| RF-67 | Puntos no transferibles | 5.8, 5.11 | F13 |
| RF-68 | Sección Próximamente | 4 | F2, F14 |
| RF-69 | Alerta con notificación y push | 6.4, 9 | F14; F16 |
| RF-70 | Preventa configurable | 5.3 | F14 |
| RF-71 | El precio vuelve al normal en el estreno | 6.1 | F14 |
| RF-72 | Mis películas | 6.2.9 | F12 |
| RF-73 | Cancelación hasta 2 h antes | 6.2.8 | F10 |
| RF-74 | El importe vuelve como crédito | 6.2.8 | F10 |
| RF-75 | Crédito en el perfil | 5.8 | F10 |
| RF-76 | Crédito combinable con otros medios | 6.2.6 | F10 |
| RF-77 | Panel de administración | 4 | F1, F11 |
| RF-78 | Selectores rápidos sin scroll | 8.6, 8.10 | F3 |
| RF-79 | Reporte de facturación diaria | 6.2.9 | F11 |
| RF-80 | Gráfico de películas más vistas | 6.2.9, 8.6 | F11 |
| RF-81 | Gráfico del producto más vendido | 6.2.9, 8.6 | F11 |
| RF-82 | Exportación a PDF y Excel | 8.11 | F11 |
| RF-83 | Log de actividad | 5.10 | F1, F11 |
| RF-84 | Acciones mínimas que se auditan | 6.3 | F1, F3, F6, F8, F9, F10, F13 |
| RF-85 | Filtros del log | 5.10 | F11 |
| RF-86 | El usuario de OAuth completa su perfil | 5.2, 7.1 | F1 |
| RF-87 | Recordatorio 24 h antes | 6.4, 9 | F7; F16 |
| RF-88 | Recordatorio 2 h antes | 6.4, 9 | F7; F16 |
| RF-89 | Aviso de QR validado | 6.2.7, 9 | F6, F7; F16 |
| RF-90 | Validador `butacasContiguas` | 8.9 | F4 |

### 12.2 Reglas de negocio

| ID | Regla | Dónde se garantiza (base) | Dónde se anticipa (interfaz) | Fase |
|---|---|---|---|---|
| RN-01 | Ocupación con 30 min de limpieza | `preparar_funcion`, `asignar_sala` | `asignarSalas` | F3 |
| RN-02 | Sin solapamiento | `funciones_sin_solapamiento` | — | F3 |
| RN-03 | Sala de menor número | `asignar_sala` | Vista previa | F3 |
| RN-04 | Butaca única por función | Clave primaria de `ocupaciones` | Estados del mapa | F4 |
| RN-05 | Retención de 10 min | `retener_butacas` y `limpiar-retenciones` | `ContadorRetencion` | F4 |
| RN-06 | Edad según clasificación; el anónimo declara | `validar_venta`, `confirmar_compra`, `proteger_fecha_nacimiento` | `appRestrictEdad` | F1, F4, F5 |
| RN-07 | Fórmula del precio de entrada | `precio_entrada` | `precioEntrada` | F5 |
| RN-08 | Ventana de preventa | `en_preventa` | `enPreventa` | F14 |
| RN-09 | Cupón de primera compra | `motivo_cupon` | `motivoRechazoCupon` | F9 |
| RN-10 | Cupón por edad | `motivo_cupon`, `proteger_fecha_nacimiento` | `motivoRechazoCupon` | F1, F9 |
| RN-11 | Un cupón; sugerir el mejor | `confirmar_compra`, `cupones_sugeridos` | `mejorCupon` | F9 |
| RN-12 | Orden de cálculo y puntos | `confirmar_compra` | `calcularResumen`, `puntosGanados` | F5, F9, F10, F13 |
| RN-13 | Canje al confirmar, sin sumar puntos | `confirmar_compra` | `calcularResumen` | F13 |
| RN-14 | Candy solo con entrada | `confirmar_compra` | El paso de candy está después del de butacas | F8 |
| RN-15 | Combo con una entrada 2D estándar | `confirmar_compra` | `diferenciaCombo` | F8 |
| RN-16 | QR con dos partes | `validar_qr` | `ResultadoValidacion` | F6 |
| RN-17 | Ventanas de validación | `validar_qr` | — | F6 |
| RN-18 | Condiciones de la cancelación | `cancelar_compra` | Botón con motivo | F10, F13 |
| RN-19 | Puntos pendientes hasta la función | `disponible_desde` | `saldoPuntos` | F13 |
| RN-20 | Top 3 | `top_peliculas` | — | F2, F5 |
| RN-21 | Reseña solo de quien vio la película | RLS con `vio_pelicula` | Botón visible solo en Mis películas | F12 |
| RN-22 | Cuándo aparece en Mis películas | `mis_peliculas`, `vio_pelicula` | — | F12 |
| RN-23 | Facturación y más vistas | `reporte_facturacion`, `reporte_mas_vistas` | — | F11 |
| RN-24 | Auditoría desde la base | `auditar_cambio` y las RPC | — | F1 en adelante |
| RN-25 | Butacas contiguas | `butacas_contiguas` en `confirmar_compra` | `butacasContiguas` | F4, F5 |

### 12.3 Requerimientos no funcionales

| ID | Requerimiento | Sección | Fase |
|---|---|---|---|
| RNF-01 | Angular 22 standalone con `bootstrapApplication` | 2.1, 8.1 | F0 |
| RNF-02 | Signals en stores por dominio | 8.4 | F1 a F5 |
| RNF-03 | Componentes reutilizables con signal inputs y outputs | 8.6 | F2 a F5 |
| RNF-04 | `@if`, `@for` y `@switch` en todas las plantillas | 2.1, 8.5 | todas |
| RNF-05 | Signal Forms en registro, ABM y programación | 8.10 | F1, F2, F3, F5 |
| RNF-06 | Carga diferida por área y guards por rol | 4, 8.6 | F1 |
| RNF-07 | Capas separadas | 3.2, 8.3 | F0 en adelante |
| RNF-08 | `strict`, `strictTemplates` y modelos tipados | 2.2, 8.1, 8.2 | F0 |
| RNF-09 | Auth con email, Google, GitHub y anónima; perfil con rol | 7.1 | F1, F4 |
| RNF-10 | RLS en todas las tablas; `service_role` fuera del frontend | 5.11, 7.4, 7.5 | todas |
| RNF-11 | Reglas críticas en la base | 6 | F3 a F6, F10, F13 |
| RNF-12 | Realtime para el mapa y las notificaciones | 7.2 | F4, F6 |
| RNF-13 | Storage para imágenes; solo el admin escribe | 7.3 | F2, F8 |
| RNF-14 | PWA instalable | 9 | F7 |
| RNF-15 | Caché, comprobantes offline y aviso de versión | 9 | F5, F7 |
| RNF-16 | Push como complemento de la bandeja | 9 | F7; F16 |
| RNF-17 | Diseño visual propio | 8.12, 13 | F0, F15 |
| RNF-18 | Responsive | 8.12 | F15 (revisión) |
| RNF-19 | Navegación fácil; fechas sin scroll | 8.6, 8.10 | F3, F5 |
| RNF-20 | Accesibilidad AA, teclado y mapa que no depende solo del color | 8.6, 8.8, 8.12 | F4, F15 |
| RNF-21 | Español, ARS y hora de Buenos Aires | 8.1, 8.7 | F0 |
| RNF-22 | Ley 25.326; tipo de sangre aparte | 5.2 | F1 |
| RNF-23 | Vitest para precio, cupones, edad, puntos y asignación | 10 | todas |
| RNF-24 | Vercel y configuración por entorno | 8.1, 11 (F0) | F0 |
| RNF-25 | GitHub y README con arquitectura y decisiones | 11 | F0 a F15 |
| RNF-26 | Pipes `duracion`, `precioArs` y `edad` | 8.7 | F1, F2, F3 |
| RNF-27 | Directivas `appHighlightButaca` y `appRestrictEdad` | 8.8 | F4, F5 |
| RNF-28 | Validador `butacasContiguas` | 8.9 | F4 |

### 12.4 Supuestos

| ID | Supuesto | Dónde se aplica |
|---|---|---|
| S-01 | J y K con 14 lugares; 532 butacas | 5.4 (plantilla) |
| S-02 | Token aleatorio y validación por sector | 5.6, 6.2.7 |
| S-03 | Formatos por sala | 5.4, 6.2.1 |
| S-04 | El anónimo solo paga y declara la edad | 1.2, 6.2.6, 8.8 |
| S-05 | Leyenda con la clasificación | 8.11 |
| S-06 | Cupón con `edad_minima` | 5.8 |
| S-07 | Puntos sobre el total tras el cupón, pendientes y anulables | 5.8, 6.2.8 |
| S-08 | Pasarela simulada e intercambiable en el servidor | 6.1, 6.2.6, 8.3 |
| S-09 | Preventa por fecha de compra, porcentaje sobre la base | 6.1 |
| S-10 | Alertas por bandeja y push, una vez | 6.4, 9 |
| S-11 | Candy solo con entrada | 6.2.6 |
| S-12 | Combo con 2D estándar más la diferencia | 5.7, 6.2.6 |
| S-13 | Cuándo una película cuenta como vista | 6.2.9 |
| S-14 | Reseña solo de quien la vio, una por película | 5.9 |
| S-15 | Top 3 de 30 días por fecha de compra | 6.2.9 |
| S-16 | Tipo de sangre opcional y en una tabla aparte | 5.2, 5.11 |
| S-17 | Retención de 10 min, reemplazo y pg_cron | 6.2.4, 6.4 |
| S-18 | Horarios de validación | 6.2.7 |
| S-19 | Asignación por menor número, viables primero | 6.2.1, 6.2.2 |
| S-20 | Cancela solo el registrado, la compra completa; el cupón no vuelve | 6.2.8 |
| S-21 | Máximo de 10 butacas configurable | 5.1 |
| S-22 | Precio base más recargo VIP fijo | 6.1 |
| S-23 | Un cupón, combinable con crédito | 6.2.6 |
| S-24 | Facturación vigente y más vistas por fecha de función | 6.2.9 |

---

## 13. Dudas abiertas y decisiones del SPEC

### 13.1 Decisiones ante ambigüedades

Todas las decisiones están aceptadas. Si el docente observa alguna, se cambia sin tocar la arquitectura.

| ID | Decisión | Afecta | Estado |
|---|---|---|---|
| D-01 | La recompensa "entrada" cubre una 2D estándar; la diferencia de formato y VIP se paga | RF-65, RN-13 | aceptada |
| D-02 | La preventa no se aplica sobre el combo, que mantiene su precio fijo | RN-07, RN-15 | aceptada |
| D-03 | El límite de usos del cupón es global | RF-40 | aceptada |
| D-04 | Al estado de la película lo cambia el admin a mano; solo el precio cambia solo | RF-11, RF-71 | aceptada |
| D-05 | Una compra está "usada" si se validó el acceso o la función terminó | RF-05, S-13 | aceptada |
| D-06 | Sala con `numero` obligatorio y `nombre` opcional | RF-16, RN-03 | aceptada |
| D-07 | Una butaca fuera de servicio afecta solo las ventas nuevas | RF-20 | aceptada |
| D-08 | Recordatorios, alertas y push solo para clientes registrados | RF-69, RF-87 a RF-89, S-04 | aceptada |
| D-09 | RF-64 se apoya en RN-12 (cálculo) y RN-19 (pendientes) | RF-64 | aceptada |
| D-10 | Una función se vende hasta su hora de inicio | RF-32, `validar_venta` | aceptada |
| D-11 | La retención tiene un único vencimiento por usuario y función: agregar butacas no reinicia el contador | RN-05, RF-29 | aceptada |
| D-12 | Combos y recompensas de entrada cubren primero las butacas no VIP | RN-15, D-01 | aceptada |
| D-13 | La facturación tiene tres líneas: entradas (incluye las diferencias de formato o VIP de las entradas de combo), candy y combos (precio fijo completo). El descuento se informa aparte y después el total neto | RF-79, RN-23 | aceptada |
| D-14 | Validar en modo candy una compra sin candy da el resultado "Esta compra no incluye candy" | RF-58 | aceptada |
| D-15 | Todos los cupones (también los generales) requieren cliente registrado, como dice la tabla de roles | RF-43, sección 3 | aceptada |
| D-16 | Se desactiva la confirmación de email. La recuperación de clave se implementa completa, pero el SMTP de Supabase solo envía a las direcciones del equipo del proyecto: en la demo se prueba con una de ellas y la limitación queda en el README (sección 7.1, paso B) | RF-01, RF-02 | aceptada |
| D-17 | Color de ojos como lista cerrada con la opción "otro" | RF-01 | aceptada |
| D-18 | El cupón de primera compra se puede usar si el cliente no tiene ninguna compra, confirmada o cancelada | RN-09, S-20 | aceptada |
| D-19 | La reseña muestra el nombre y la inicial del apellido del autor | RF-61, RNF-22 | aceptada |
| D-20 | Una compra corresponde a una sola función | RF-32, RN-16 | aceptada |
| D-21 | El rango de la programación se elige con un chip de inicio y la cantidad de semanas (1 a 8), sin calendario | RF-22, RF-78 | aceptada |
| D-22 | Tarjetas de prueba: `4242 4242 4242 4242` aprueba, `4000 0000 0000 0002` rechaza, `4000 0000 0000 0119` da error. Se listan en el README. De las cuentas de prueba (una por rol), el README nombra solo los roles: los emails y las claves se mandan al docente por mail | S-08 | aceptada |
| D-23 | Sin Edge Functions ni `service_role`. El admin crea la cuenta interna con `signUp` desde un segundo cliente que no guarda sesión y le asigna el rol con la RPC `asignar_rol`, que solo ejecuta un admin (sección 7.4) | RF-06 | aceptada (viable; plan B en 7.4) |
| D-24 | Sin push real hasta que responda el docente. Los avisos llegan a la bandeja en tiempo real y, con la app abierta, como aviso del sistema. El push con la app cerrada queda como fase opcional F16 | RF-69, RF-87 a RF-89, RNF-16 | reemplazada por D-25 el 09/10 |
| D-25 | Push con la app cerrada con una Edge Function (`enviar-push`, con `web-push`) que llama un trigger `after insert on notificaciones` mediante pg_net. Las suscripciones van en `suscripciones_push`, la VAPID privada solo como secreto de la función y el secreto compartido en Vault. La función no lee la base ni usa `service_role`. Alternativas descartadas: función en Vercel, OneSignal o Firebase, y Postgres solo (no puede cifrar ni firmar el push) | RF-69, RF-87 a RF-89, RNF-16 | aceptada el 09/10 |
| D-26 | Acceso para la corrección: botón en `/login` que completa la cuenta admin de la cátedra (sección 7.7) | Pedido de la cátedra | aceptada el 09/10 |

### 13.2 Pendientes

1. **Identidad visual** (RNF-17): definida "por ahora" (sección 8.12). Bree Serif y Poppins; fondo `#12100D`, violeta `#8226EF` con dos variantes oscuras y sin aguamarina; sombras en lugar de bordes; tarjetas de 12 px con el póster en los 3/4 de arriba. Falta definir los colores de las butacas en F4, con esta misma paleta.
2. **URL y clave del proyecto Supabase** y **subdominio de Vercel**: completados en F0 (`xrbjzgqosscoowmkwnxf` y `tp1-sistema-cine.vercel.app`).
3. **Cuentas de prueba**: se crean en F1. El README nombra solo los roles; los datos de acceso se mandan al docente por mail.
   - Una por rol: cliente, empleado y admin.
   - Para la demo, además: un cliente mayor de 50 y uno de 16 años.
   - Son cuentas solo para la evaluación y su clave se cambia después de la nota.

### 13.3 Riesgos técnicos que hay que verificar

| Riesgo | Cómo se detecta | Alternativa |
|---|---|---|
| El SMTP por defecto de Supabase no envía la recuperación de clave a direcciones fuera del equipo | Es una limitación conocida (D-16) | La demo se hace con una dirección del equipo y el README lo aclara. Un SMTP propio (por ejemplo Resend) solo si el docente lo exige |
| Un chunk de inicio con nombre `chunk-*.js` que no se precachee | `ngsw.json` después de `ng build` | `chunk-*.js` ya está en el grupo `inicial` (sección 9) |
| Las claves de prueba quedarían públicas si fueran al README | Cualquiera podría modificar datos de la app desplegada | No van al README: se mandan por mail al docente. Son cuentas exclusivas para la evaluación y la clave se cambia después de la nota |
| Aviso del sistema en iOS | Solo funciona con la PWA instalada y la app abierta | La bandeja de la app es el canal garantizado (S-10) |
| El push no llega en la demo | Prueba previa en Android, PC y iPhone con la PWA instalada | La bandeja es el canal garantizado; el celular del autor queda configurado de respaldo (F16) |
| `npm:web-push` no anda en el runtime de Supabase | Prueba aislada de F16, paso 1 | `jsr:@negrel/webpush`, pensada para Deno, con el mismo diseño |
| Con el acceso para la corrección, cualquiera entra como admin | La clave está en el JavaScript publicado | El RLS limita todo al rol; se restaura con el admin privado o desde el panel (sección 7.7) |
| Argentina vuelve a usar horario de verano | Cambio de normativa | Cambiar `DATE_PIPE_DEFAULT_OPTIONS` y `inicioBA` a la zona IANA; SQL ya usa `America/Argentina/Buenos_Aires` |
| Abuso del inicio de sesión anónimo | Límite por IP de Supabase | CAPTCHA (fuera de alcance) |
| Presupuesto de 4 kB por componente para el CSS de `MapaButacas` | `ng build` | Subir el presupuesto en `angular.json` solo para ese caso o simplificar los estilos |
| Bundle inicial cerca del aviso: en F0 ya pesa 453 kB, casi todo `supabase-js` | `ng build` en cada fase. En F1 llegó a 566 kB con Signal Forms en la carga inicial; con las pantallas de acceso en `loadComponent` quedó en 487 kB. En F2, con la cartelera, 508 kB aun con la ficha y Próximamente diferidas | **Aplicado en F2:** `maximumWarning` en 600 kB, anotado en las decisiones del README. El error sigue en 1 MB. Mantener las áreas y las librerías pesadas en carga diferida |
| Proyectos nuevos de Supabase sin los permisos por defecto en `public` | Error 401 o 42501 en F0 | Agregar en un script nuevo los `grant` explícitos a `anon` y `authenticated` |

### 13.4 Técnicas: lo visto en clase y las decisiones cerradas

Las clases 1 a 8 son la referencia definitiva del "cómo". Donde difieren del código de `taskflow`, mandan las clases. Criterio acordado: **en Angular, todo como en clase; la base queda como está diseñada.**

**Regla para todo el proyecto:** si hace falta una técnica que no se vio en clase, se frena y se consulta, ofreciendo la alternativa con lo visto en clase.

**No vistas en clase, pero se mantienen** porque las exige el documento de requerimientos o porque así se decidió:

| Técnica | Motivo |
|---|---|
| Funciones SQL y RPC (`security definer`, `search_path`, `for update`, `on conflict`) | RNF-11 y sección 5 del documento |
| Triggers (auditoría, perfil, fecha de nacimiento inmutable) | RN-24, RNF-11; los de perfil y fecha, por decisión |
| pg_cron | Lo nombra S-17 |
| Restricción de exclusión con `btree_gist` | La pide RN-02 |
| Sesión anónima | RNF-09 |
| Vistas con `security_invoker` y `grant` por columna | Decisión: la base queda como está |
| Vitest, solo con funciones puras | RNF-23 (sección 10) |
| `LOCALE_ID`, `registerLocaleData` y `DATE_PIPE_DEFAULT_OPTIONS` | RNF-21 |
| `localStorage` para los comprobantes | RNF-15 y S-04 |
| Aviso del sistema con `showNotification` | Complemento de la bandeja con la app abierta. El usuario lo confirmó el 08/10, al cerrar F7 |
| Edge Function `enviar-push`, `web-push`, pg_net, Vault y `SwPush` | La cátedra exigió push con la app cerrada (D-25). `SwPush` es del mismo paquete que `SwUpdate` (Clase 8) |
| Segundo cliente de Supabase y la RPC `asignar_rol` | RF-06 sin Edge Functions (D-23) |
| `namedChunks` y chunks de pantallas en `installMode: lazy` | Desvío a propósito respecto de Clase 8, lámina 21 (sección 9) |
| `qrcode`, `@zxing/browser`, `jspdf` y `write-excel-file` | RF-51, RF-53, RF-55 y RF-82 |

**Reemplazadas por la técnica de clase:**

| Antes | Ahora | Clase · lámina |
|---|---|---|
| `enum` en SQL | `text` + `check (... in (...))` | 5·10 |
| `loadChildren` y `*.routes.ts` por área | `loadComponent` con `canMatch` en cada ruta; el menú de cada rol en `app.html` | 3·6, 7·7 |
| Guard fábrica `rolGuard(roles)` | `adminGuard`, `empleadoGuard` y `clienteGuard` | 7·7 |
| Stores provistos en el componente | Stores en `root` con `reiniciar()` llamado en `ngOnDestroy` | 5·5, 6·13 |
| `DestroyRef` y `effect` con `onCleanup` | `ngOnDestroy` con `removeChannel` | 6·13 |
| `linkedSignal` | `signal` + `effect` | 4·25 |
| Clase abstracta `PagoService` + `useClass` | Un `PagoService` común | 5·5 |
| Realtime con `filter` | Toda la tabla, filtrada en el callback | 6·10 |
| `appRestrictEdad` de atributo con `exportAs` | Estructural, sin `else`; el componente muestra el motivo con `@if` | 7·15, 8·17 |
| Componente `AvisoVersion` con `toSignal` | `SwUpdate` en `app.ts` con un signal y `@if` en `app.html` | 8·23 |
| `ng generate environments` (`environment.development.ts`, `supabaseKey`) | `environment.ts` y `environment.prod.ts` con `fileReplacements` y `supabaseAnonKey` | 6·20 |
| `compartido/` con subcarpetas en español | `shared/` con `api/`, `stores/`, `services/`, `models/`, `guard/`, `pipes/`, `directives/`, `validators/`, `components/`, `rules/` y `utils/` | 8·3 |
| `publico/ingresar/` | `publico/login/` (`Login`, ruta `/login`) | 6·16, 8·8 |

**Agregadas, porque se vieron en clase:**

| Técnica | Dónde | Clase · lámina |
|---|---|---|
| `pendingChangesGuard` (`canDeactivate`) | Formularios del admin, registro, perfil y checkout con butacas retenidas | 7·6, 8·6 |
| `peliculaResolver` (`resolve`) | Ficha de película, con la película como input | 7·7, 8·7 |
| Marcar todos los campos al enviar (`onInvalid: 'touch'` en la lámina) | Todos los formularios. En Angular 22.2 lo hace `submit` solo (sección 8.10) | 4·24 |

**Nombres de archivo:** los que genera el CLI 22.2 (sección 2.4). Los servicios usan `@Injectable({ providedIn: 'root' })`, que es lo que muestra el código de clase, en lugar del `@Service()` que el CLI 22.2 pone por defecto.
