# Defensa del TP — guía personal

Este archivo es solo para vos. Igual que `SPEC.md`, está en `.git/info/exclude`: git lo ignora sin que figure en el `.gitignore` del repo. Lo que el docente tiene que ver del repo (arquitectura y decisiones técnicas) va en el README (RNF-25).

Describe lo implementado hasta F11, F16 y los pedidos del 09/10 (acceso para la corrección, ver la clave y la barra con grupos) y, para F12 a F15, lo planificado en `SPEC.md`. Se actualiza al cerrar cada fase con lo que realmente quedó implementado:

- Se marcan con ✅ los archivos que ya existen.
- Se corrige lo que haya cambiado durante la implementación.
- Se agregan las preguntas nuevas que surjan.

Registro de fases:

| Fase | Estado | Qué quedó realmente |
|---|---|---|
| F0 | cerrada el 07/10: commit, push y despliegue en https://tp1-sistema-cine.vercel.app (el refresco de una ruta lo resuelve el preset Angular) | Hecho:<br>• `strict` y `strictTemplates` sin errores.<br>• Environments con el esquema de clase (`environment.ts` y `environment.prod.ts` con `fileReplacements`), con la URL y la clave publicable en `supabaseAnonKey`.<br>• `supabase-js` 2.117.2. La CLI de Supabase se desinstaló, porque la base va por SQL Editor.<br>• `supabase/sql/01_base.sql`, escrito con técnicas de clase, aplicado y verificado por REST (cuatro formatos y la configuración).<br>• `supabase/sql/tp1-sistema-cine.sql`: todo el resto de la base en un solo script, probado en PGlite con 151 controles y aplicado el 07/10 (27 tablas con RLS, 532 butacas, 3 jobs de pg_cron funcionando, Realtime, Storage y `BIENVENIDA`).<br>• Buckets `posters` y `productos`, públicos.<br>• `shared/` con `api/configuracion-api.ts`, `models/`, `services/supabase.client.ts` y `utils/errores.ts`.<br>• Identidad: Bree Serif y Poppins; fondo `#12100D`, violeta `#8226EF` con dos variantes oscuras y sin aguamarina; sombras en lugar de bordes; navbar centrada con subrayado violeta animado desde el centro.<br>• Locale `es-AR` y hora `-0300`.<br>• `Inicio` muestra los formatos que vienen de la base.<br>• README esqueleto.<br>• `skipTests` en los generadores del CLI.<br><br>Build: `main` de 453 kB, cerca del aviso de 500 kB por `supabase-js`. Pruebas: 1 archivo (`errores.spec.ts`), 4 en verde |
| F1 | cerrada el 09/10: GitHub y Google configurados y probados en Vercel (en la base, dos identidades de Google y una de GitHub; una completó el perfil). Las cuentas de prueba ya existían (2 admin, 2 empleados, 8 clientes; las claves van por mail al docente, no al README) | Hecho:<br>• `AuthApi`, `PerfilApi`, `CuentaInternaApi` y `SesionStore` con `listo()`.<br>• Seis guards: `invitadoGuard`, `clienteGuard`, `empleadoGuard`, `adminGuard`, `puedeComprarGuard` y `pendingChangesGuard`.<br>• Pantallas: `Login` (clave, GitHub y Google), `Registro`, `RecuperarClave`, `RestablecerClave`, `Perfil` (con el pipe `edad`), `CompletarPerfil`, `Validacion` (mínima) y `CuentasAdmin`.<br>• `FormularioPerfil` compartido, con fecha de nacimiento fija una vez cargada y datos sensibles con consentimiento.<br>• Menú por rol con `@switch` y aviso de perfil incompleto en `app.html`.<br>• Reglas puras `edad`, `fechas` y `perfil`, más `rutaSegura`.<br>• `/privacidad`, enlazada desde el pie, el registro y el tipo de sangre: existe porque Google la pide para pasar el login a producción.<br><br>Build: inicial de 487 kB (las pantallas de acceso van con `loadComponent`). Pruebas: 5 archivos, 31 en verde |
| F2 | cerrada el 07/10 (`4512a09`) | Hecho:<br>• `PeliculaApi`, `GeneroApi`, `ImagenApi` (póster en el bucket `posters` con `nombreSeguro`) y `CarteleraStore`.<br>• Admin: `PeliculasAdmin`, `PeliculaForm` con `SelectorGeneros` (control propio con `model()`) y `GenerosAdmin`, con `pendingChangesGuard`.<br>• Público: `Inicio` con top 3 y cartelera con buscador y filtros por género y clasificación; `PeliculaDetalle` con `peliculaResolver`; `Proximamente`.<br>• `TarjetaPelicula` y el pipe `duracion`.<br>• Reglas `formato.ts` y `pelicula.ts`, más `utils/archivos.ts`.<br><br>Pruebas: 8 archivos, 52 en verde |
| F3 | cerrada el 07/10 (`5d6d196`) | Hecho:<br>• `SalaApi`, `FuncionApi` y `ConfiguracionApi` completa; pipe `precioArs`.<br>• Admin: `SalasAdmin` con `SelectorFormatos`, `SalaButacas` (tipo de butaca y fuera de servicio), `PreciosAdmin`, `FuncionesAdmin`, `FuncionForm` y `ProgramarFunciones` con `SelectorDias`, `SelectorHora`, `SelectorFecha`, semanas y vista previa de `asignarSalas`.<br>• Filtros de formato e idioma en la cartelera y funciones por día en la ficha.<br>• Reglas `asignacion.ts`, `sala.ts` y lo nuevo de `fechas.ts`.<br>• Sin SQL nuevo: `funciones` no tiene políticas de alta ni de cambio, todo pasa por `programar_funciones` y `modificar_funcion`.<br><br>Build: inicial de 515 kB. Pruebas: 10 archivos, 79 en verde |
| F4 | cerrada el 07/10 (`cb44f46`) | Hecho:<br>• `OcupacionApi` y `ButacasStore` con Realtime sobre `ocupaciones`.<br>• `MapaButacas` con `appHighlightButaca`, leyenda y precio VIP; `ContadorRetencion`.<br>• `PasoButacas` con el validador `butacasContiguas` y el máximo de 10, dentro de un `Checkout` provisorio.<br>• Sesión anónima (`signInAnonymously`) al tocar la primera butaca.<br>• Reglas `contiguidad.ts` y `precioEntrada`.<br>• Sin SQL nuevo: se verificó que `ocupaciones` está en Realtime y que `limpiar-retenciones` corre cada minuto.<br><br>Build: inicial de 517 kB. Pruebas: 12 archivos, 94 en verde |
| F5 | cerrada el 07/10 (`3421241`) | Hecho:<br>• Checkout completo con `@switch` sobre el paso e `IndicadorPasos`: `PasoFuncion` (`SelectorFecha` y `SelectorHorario`), `PasoButacas`, `PasoCandy` (vacío hasta F8), `PasoPago` con Signal Forms y `PagoService`, y `PasoComprobante`.<br>• `VistaComprobante` con QR (`utils/qr.ts`) y PDF (`utils/comprobante-pdf.ts`), los dos con `import()`, y la leyenda para +13 y +18.<br>• `*appRestrictEdad` en la ficha y en el paso de función, con el motivo en `@if`; declaración de edad del anónimo.<br>• `ComprobantesLocales`, `/comprobantes`, `MisCompras` y `DetalleCompra`.<br>• Top 3 con `top_peliculas`. Tarjetas de prueba en el README.<br>• Reglas `edad.ts` (leyenda), `tarjeta.ts`, `enPreventa`, `diferenciaCombo` y `calcularResumen`.<br><br>Build: inicial de 529 kB. Pruebas: 13 archivos, 115 en verde |
| F6 | cerrada el 08/10 (`7586aba`), probada salvo la cámara en Vercel | Hecho:<br>• `ValidacionApi` y `/empleado`: modo acceso o candy, `EscanerQr` con `@zxing/browser`, ingreso manual con Signal Forms y `ResultadoValidacion` con `@switch`. El admin también ve "Validar".<br>• `NotificacionesStore` (abre el canal antes de traer la lista) y `BandejaNotificaciones` en la cabecera del cliente.<br>• Sin SQL nuevo: `authenticated` ejecuta `validar_qr` y puede marcar `notificaciones.leida`.<br><br>Pruebas: 13 archivos, 118 en verde |
| F7 | cerrada el 08/10 (`048fa77`) | Hecho:<br>• `ng add @angular/pwa`, `namedChunks`, `ngsw-config.json` con `inicial`, `diferido` y `recursos` (más `cartelera` e `imagenes`), manifest con íconos propios `any` y `maskable`, y `theme-color`.<br>• `ngsw.json` comprobado: `inicial` no tiene pantallas de las áreas.<br>• Aviso de versión nueva en `app.ts` con `SwUpdate`.<br>• `AvisosSistema`, llamado al llegar un INSERT, y `/cuenta/notificaciones` con **Activar avisos**.<br>• `CarteleraStore`: si falla `top_peliculas` sin conexión, la cartelera se muestra igual.<br>• Sin SQL nuevo: existen los jobs `recordatorios`, `alertas-venta` y `limpiar-retenciones`.<br><br>Build: inicial de 546 kB (el service worker suma unos 8 kB). Pruebas: 13 archivos, 118 en verde |
| F8 | cerrada el 08/10 (`95da9b9`) | Hecho:<br>• `CandyApi` y la regla `candy.ts`; `entradasConCombos` en `precio.ts`.<br>• Admin: `CategoriasAdmin`, `ProductosAdmin` y `CombosAdmin` (con `SelectorProductos`), con la botonera `.secciones`.<br>• `PasoCandy`: combos destacados, productos por categoría, un combo por entrada como máximo y aviso si sobran combos al volver a las butacas.<br>• El comprobante y el PDF muestran "con {combo}"; la validación en modo candy lista lo que hay que entregar.<br><br>Build: inicial de 547 kB. Pruebas: 14 archivos, 130 en verde |
| F9 | cerrada el 08/10 (`8282f60`) | Hecho:<br>• `CuponApi` (los sugeridos y la validación por RPC, porque el cliente no lee `cupones`) y la regla `cupones.ts`.<br>• `CuponesAdmin` en `/admin/cupones` (general o por edad, vigencia y límite de usos opcionales); `BIENVENIDA` aparte, con su porcentaje editable en `PreciosAdmin`.<br>• `PasoPago`: el mejor cupón sugerido con **Aplicar**, un código tipeado o **Quitar**; uno solo por compra. Si la base lo rechaza al pagar, lo saca y muestra el motivo.<br>• `Perfil`: "Tus cupones" mientras haya alguno.<br>• El comprobante y el PDF muestran "Descuento (cupón X)".<br><br>Build: inicial de 548 kB. Pruebas: 15 archivos, 149 en verde |
| F10 | cerrada el 08/10 (`fb1be95`) | Hecho, sin SQL nuevo (`cancelar_compra`, `saldos_credito` y el crédito de `confirmar_compra` ya estaban):<br>• `DetalleCompra`: sección "Cancelar la compra" solo si está vigente. Muestra el plazo, que sale de `configuracion.horas_limite_cancelacion`. Si no se puede, el botón queda deshabilitado con el motivo de `motivoNoCancelable`. Pide `confirm()` avisando que el cupón no vuelve y después muestra el crédito acreditado.<br>• `PasoPago`: "Crédito a favor" debajo del cupón, con **Usar** todo, un monto parcial o **Quitar**. Si no queda nada por pagar, aparece **Confirmar compra** sin tarjeta, que manda `p_pago` nulo.<br>• `Perfil`: "Tu crédito", con el saldo y los movimientos enlazados a su compra.<br>• `FidelizacionApi` (`saldoCredito`, `movimientosCredito`), `CompraApi.cancelar` y la regla `cancelacion.ts`.<br><br>Build: inicial de 548 kB. Pruebas: 16 archivos, 156 en verde |
| F11 | cerrada el 09/10 (`ebbcd8f`) | Hecho, sin SQL nuevo (las tres RPC de reporte y la política de lectura de `auditoria` ya estaban):<br>• `write-excel-file` 4.1.1 instalada (se importa de `write-excel-file/browser`).<br>• `ReporteApi` y `Reportes` en `/admin/reportes`: atajos Hoy, Últimos 7 días y Este mes, rango en dd/mm/aaaa, tabla de facturación por día con totales, **Exportar PDF** (`reporte-pdf.ts`) y **Exportar Excel** (`excel.ts`), los dos con `import()`; más vistas por semana o por mes y candy más vendido con `GraficoBarras`.<br>• `AuditoriaApi` y `Auditoria` en `/admin/auditoria`: filtros por email, acción y rango, los 200 más recientes, **Ver solo sus acciones** y "Ver datos" con los campos que cambiaron.<br>• `GraficoBarras` en `shared/components/` y la clase global `.solo-lectores`.<br>• Reglas `reportes.ts` y `auditoria.ts` con sus specs. Menú del admin con Reportes y Auditoría.<br><br>Build: inicial de 549 kB. Pruebas: 18 archivos, 179 en verde |
| Pedidos del 09/10 | hechos el 09/10: acceso (`e4af56a`), ver clave (`cb7aea7`) y barra (`cdb816e`) | Hecho, sin SQL nuevo:<br>• **Acceso para la corrección** en `/login`: completa la cuenta admin de la cátedra en el formulario (sección 2.22, D-26).<br>• **`VerClave`** (`app-ver-clave`): botón para mostrar u ocultar la clave, con `model()`. Va en el login, el registro, la clave nueva, la cuenta interna y el código de seguridad del pago.<br>• **`MenuPrincipal`** (`app-menu-principal`) con los grupos de `menu.model.ts`: en la computadora, Cartelera y Próximamente sueltos, el admin con cuatro desplegables (Cine, Ventas, Gestión y Reportes) y "Mi cuenta" a la derecha; en el celular, un panel lateral con los grupos como acordeón (sección 2.24) |
| F16 | cerrada el 09/10 (`9ba9bb7`; la quita de la demo va en `cdb816e`). El push real funciona desde las 18:54 del 09/10, probado en Android con la app cerrada | Hecho:<br>• `03_push.sql` aplicado: `pg_net`, `suscripciones_push` con RLS de solo lectura, las RPC `guardar_suscripcion_push` y `borrar_suscripcion_push`, el trigger `notificaciones_push` (`after insert`, llama a la Edge Function por pg_net) y `notificaciones_validacion` (`before insert`, suma sector, fecha y hora al aviso de QR validado). El secreto compartido va en Vault (`push_secreto`).<br>• Edge Function `enviar-push` (`npm:web-push@3.6.7`), desplegada desde el panel con `verify_jwt` apagado y cuatro secretos (`VAPID_PUBLIC_KEY`, `VAPID_PRIVATE_KEY`, `VAPID_SUBJECT` y `PUSH_SECRETO`). La prueba aislada respondió 201.<br>• Front: `vapidPublicKey` en los environments, `PushApi`, `AvisosSistema.suscribir()` (en **Activar avisos** y al entrar si el permiso ya estaba) y `desuscribir()` en `salir()`.<br>• Se armó `/admin/avisos` para disparar avisos en el momento y se sacó el mismo día porque no está en los requerimientos: `04_quita_demo_avisos.sql` (aplicado, verificado por lectura) borra `probar_recordatorio` y `disparar_alertas` y vuelve `alertas-venta` a 10 minutos.<br>• **Incidente del 09/10:** hasta las 18:54, `enviar-push` respondió 401 a todas las llamadas, porque `PUSH_SECRETO` no coincidía con el valor de Vault. Lo que parecía push eran avisos del sistema por Realtime con la app abierta o en segundo plano. Se detectó porque la alerta de venta, que salió con la app cerrada de verdad, no llegó. Se vio en `net._http_response` y en los logs de la función, y se arregló pegando en `PUSH_SECRETO` el valor de Vault, sin redesplegar.<br>• Después del arreglo: QR validado con la app cerrada en Android. La función respondió 200 y FCM, 201 para los dos dispositivos del cliente. Falta iPhone con la PWA instalada.<br><br>Build (con la barra nueva): inicial de 562 kB, sin avisos. Pruebas: 18 archivos, 179 en verde (F16 no agregó reglas puras) |
| F14 recortada | cerrada el 09/10 (`e3cff7c`): la alerta llegó a la bandeja a las 17:20, y como push con la app cerrada desde el arreglo del secreto (ver F16) | Hecho, sin SQL nuevo (`alertas`, su RLS y `procesar_alertas` ya estaban; se verificó por lectura que el alta exige rol cliente y película en `proximamente`):<br>• `AlertaApi` (`mias`, `activar` y `desactivar`) y `AlertasStore` en `root`, que trae las alertas una vez cuando entra un cliente, como `NotificacionesStore`.<br>• `BotonAlerta` (`app-boton-alerta`) debajo de cada tarjeta de Próximamente y en la ficha. Se muestra solo si la película está en `proximamente`, sin preventa abierta y sin alerta enviada. Sin sesión invita a iniciarla y vuelve a la misma pantalla. Si el permiso de avisos no está dado, sugiere activarlo.<br>• No va dentro de `TarjetaPelicula` (como decía el SPEC) porque la tarjeta es un enlace.<br>• Pendiente: la preventa en `PeliculaForm`. Desde la app, la alerta sale cuando la película pasa a cartelera con una función futura.<br><br>Build: inicial de 562 kB, sin avisos. Pruebas: 18 archivos, 179 en verde |
| F13 | código hecho el 09/10, falta probarla y commitearla (con la regresión de F5 y F10) | Sin SQL nuevo: la base suma los puntos desde F5 (`ganados = floor(total)`, con `disponible_desde` en el inicio de la función) y los anula al cancelar.<br>• `rules/puntos.ts`: `puntosGanados`, `saldoPuntos` (disponibles y pendientes según `disponible_desde`) y `textoPuntos` (punto de miles), con 6 pruebas.<br>• El Perfil muestra "Tus puntos": los disponibles, los pendientes con la fecha en que se liberan, los movimientos y las recompensas activas.<br>• `RecompensasAdmin` en `/admin/recompensas`: un producto del candy o una entrada 2D estándar, con el costo en puntos.<br>• Canje en `PasoPago`: "Canjeá tus puntos" con contadores; `CompraStore` manda `p_recompensas` (uno por canje), las entradas canjeadas cubren butacas después de los combos (`entradasConCombos` con 4.º parámetro) y, si la base rechaza el canje, se sacan y se recargan los puntos.<br>• El pago muestra "Con esta compra sumás N puntos" (RF-36) y el comprobante y el PDF marcan lo canjeado y los puntos.<br><br>Build: inicial de 565.27 kB, sin avisos. Pruebas: 20 archivos, 196 en verde |
| F12 | cerrada el 09/10 (`143c0f8`) | Sin SQL nuevo: `resenas` (RLS con `vio_pelicula`, `unique` y trigger del autor), la vista `mis_peliculas` y el promedio en `peliculas_cartelera` ya estaban; se verificó por lectura.<br>• `resena.model.ts`, `ResenaApi` (`dePelicula`, `mias`, `guardar` y `misPeliculas`) y `rules/resena.ts` (texto del promedio, cantidad y estrellas llenas).<br>• `Estrellas` (`app-estrellas`): control con `model()` para `[formField]`, y `soloLectura` para mostrar.<br>• `MisPeliculas` en `/cuenta/peliculas` (cliente, con `pendingChangesGuard`) y "Mis películas" en el menú: reseña en línea con Signal Forms (una o más estrellas y comentario de hasta 280).<br>• La ficha muestra la calificación en los detalles y la sección Reseñas; la tarjeta muestra ★ y el promedio.<br>• `mensajeError` traduce el `unique` de reseñas y el 42501 de RLS.<br><br>Build: inicial de 563.73 kB, sin avisos. Pruebas: 19 archivos, 188 en verde |
| F14 (preventa) | cerrada el 09/10 (`acd19fc`): probada en Vercel (alerta por preventa, precio con 15 % y vuelta al precio normal) | Sin SQL nuevo: se verificó por lectura que el admin escribe `peliculas` con la política `ALL` y que la base ya tiene los checks (0 a 100 y 1 a 60).<br>• `PeliculaForm` suma la sección Preventa: la casilla, el descuento (%) y los días antes del estreno, con `required`, `min`, `max` y un `validate` de días enteros.<br>• Un `computed` muestra el día en que abre (`aperturaPreventa` en `rules/pelicula.ts`, que usa `sumarDias`).<br>• `PeliculaDraft`, `PeliculaFila`, `borradorDesdePelicula` y `filaDesdeBorrador` llevan las tres columnas `preventa_*`.<br>• El precio y el cierre ya estaban en la base desde F5 (`en_preventa` y `precio_entrada`).<br><br>Build: inicial de 562.82 kB, sin avisos. Pruebas: 18 archivos, 183 en verde |

---

## 1. El sistema en un minuto

> Cineverso es una PWA para un cine de un solo edificio. Se puede comprar con o sin cuenta: se eligen función, butacas en un mapa en tiempo real, candy y descuentos, y se paga con una pasarela simulada. La compra genera un único QR que el empleado valida una vez en la puerta y una vez en el candy. El admin maneja catálogo, salas, programación con asignación automática de sala, precios, promociones, reportes y auditoría.
>
> Está hecha con Angular 22 (standalone, Signals, Signal Forms, carga diferida) y Supabase (Auth, Postgres con RLS, RPC, triggers, Realtime y Storage).
>
> **Las reglas que tocan salas, butacas, QR y dinero las garantiza la base de datos.** Angular solo las anticipa para que la experiencia sea buena. Así se cumplen aunque dos personas compren o programen a la vez.

---

## 2. Las decisiones que más te van a preguntar

Cada una tiene la decisión, por qué, qué descartaste y la frase para el oral.

### 2.1 Las reglas viven en la base y se anticipan en Angular

| | |
|---|---|
| Decisión | Cada regla crítica existe dos veces: en SQL, que es la que manda, y como función pura de TypeScript en `shared/rules/`, que la anticipa |
| Por qué | El documento lo exige (sección 5): la interfaz anticipa y la base garantiza. Las funciones de TypeScript se prueban con Vitest (RNF-23) |
| Descartado | Reglas solo en Angular (un usuario con DevTools se las saltea) y solo en SQL (la interfaz no podría avisar antes ni mostrar el precio en vivo) |
| Frase | "La base es la única que decide; Angular solo le muestra al usuario lo que la base va a decir." |

### 2.2 Una tabla `ocupaciones` con clave primaria `(funcion_id, butaca_id)`

| | |
|---|---|
| Decisión | Retenciones y ventas en la misma tabla. Cada butaca tiene como máximo una fila por función |
| Por qué | RN-04 queda resuelta por la propia clave primaria: si dos personas intentan la misma butaca, una gana el `insert` y la otra choca. El mapa escucha una sola tabla por Realtime |
| Descartado | Retenciones y entradas por separado: harían falta dos suscripciones, y `entradas` tendría que ser pública |
| Frase | "No hay forma de vender dos veces la misma butaca porque Postgres no permite dos filas con la misma clave." |

### 2.3 Restricción de exclusión contra funciones solapadas

| | |
|---|---|
| Decisión | `exclude using gist (sala_id with =, ocupacion with &&)` sobre un `tstzrange` que va del inicio al fin más 30 minutos |
| Por qué | RN-02 pide que lo garantice la base. Es la herramienta de Postgres para "no puede haber dos rangos superpuestos con la misma clave" |
| Descartado | Validarlo en un trigger o en la RPC: con dos admins programando al mismo tiempo, los dos verían la sala libre |
| Frase | "Es como un `unique`, pero para rangos de tiempo: dos funciones de la misma sala no pueden pisarse." |

### 2.4 RPC `security definer` con `search_path` vacío

| | |
|---|---|
| Decisión | `retener_butacas`, `confirmar_compra`, `validar_qr`, `cancelar_compra` y `programar_funciones` son funciones de Postgres que corren con permisos del dueño y en una sola transacción |
| Por qué | El cliente no puede escribir directo en `ocupaciones` ni en `compras`: solo puede pedirle a la función que lo haga, y la función valida todo. Si algo falla, la transacción se revierte entera (RNF-11). El `search_path` vacío evita que alguien suplante una tabla con otra del mismo nombre |
| Descartado | Insertar desde Angular con RLS: RLS controla filas, pero no puede validar contigüidad, precios ni saldos entre tablas |
| Frase | "El frontend no escribe ventas; le pide a la base que venda, y la base decide si puede." |

### 2.5 RLS, grants por columna y vistas `security_invoker`

| | |
|---|---|
| Decisión | RLS en todas las tablas (RNF-10), `grant update (columnas)` en `perfiles` y vistas con `security_invoker = true` |
| Por qué | RLS filtra **filas**. Para que un cliente edite su perfil pero no su `rol`, hace falta el grant por **columna**. Sin `security_invoker`, una vista se saltearía el RLS |
| Fecha de nacimiento | El grant la incluye solo para la primera carga. Después, el trigger `proteger_fecha_nacimiento` rechaza el cambio: si se pudiera editar, un menor podría comprar una +18 (RN-06) o alguien de 40 usar el cupón de mayores de 50 (RN-10) |
| Frase | "RLS dice qué filas ves; el grant por columna dice qué campos podés tocar; y la edad, una vez cargada, no la toca nadie desde la app." |

### 2.6 El tipo de sangre va en otra tabla

| | |
|---|---|
| Decisión | `datos_sensibles` es aparte, con una política "solo el titular" y sin excepción para el admin |
| Por qué | Ley 25.326 (S-16, RNF-22). Como RLS no oculta columnas, en `perfiles` el admin lo vería. Es opcional y requiere consentimiento |
| Frase | "Ni el admin puede verlo: la tabla solo deja leer al dueño." |

### 2.7 Sesión anónima para comprar sin cuenta

| | |
|---|---|
| Decisión | Cuando un visitante toca su primera butaca, se llama a `signInAnonymously()` |
| Por qué | La retención necesita un `usuario_id`, y RNF-09 pide sesiones anónimas. El anónimo no deja datos personales (S-04) |
| Detalle | El usuario anónimo usa el rol `authenticated` de Postgres con el claim `is_anonymous = true`. El rol `anon` corresponde a no tener sesión |
| Frase | "Supabase le da al visitante un usuario temporal sin datos, solo para que la base sepa de quién es cada retención." |

### 2.8 El pago se resuelve dentro de `confirmar_compra`

| | |
|---|---|
| Decisión | La RPC valida Luhn, vencimiento y CVV, y decide aprobado, rechazado o error según la tarjeta de prueba. En Angular, `PagoService` es un servicio común, como los de clase, que llama a la RPC |
| Por qué | S-08 pide que se resuelva en el servidor. En la misma transacción que la compra no puede pasar "pagó pero no se guardó". La pasarela queda aislada en `pago-service.ts`: para cambiarla se toca solo ese archivo |
| Seguridad | El número de tarjeta y el CVV no se guardan en ninguna tabla, ni en la auditoría, ni en los mensajes de error |
| Si se rechaza | La función borra las retenciones y **devuelve** el resultado en lugar de lanzar una excepción. Con una excepción, la liberación se revertiría junto con todo lo demás |
| Frase | "La compra y el pago son una sola transacción: o pasa todo o no pasa nada." |

### 2.9 Un solo código aleatorio para el QR

| | |
|---|---|
| Decisión | 10 caracteres de un alfabeto de 32 símbolos sin I, O, 0 ni 1, generados con `gen_random_bytes`. El QR codifica ese mismo código, que se imprime como `ABCD-EFGH-JK` |
| Por qué | RF-54 pide que el QR no tenga datos personales y que el mismo código sirva para el ingreso manual. Son unos 50 bits de azar, imposibles de adivinar, y además solo el personal puede validar |
| Descartado | Un JWT firmado (S-02): la validación consulta la base igual, así que la firma no aporta nada |
| Frase | "El QR no contiene nada de la compra: es una llave que solo la base sabe a qué compra corresponde." |

### 2.10 Todos los stores en `root`, con `reiniciar()` en el checkout

| | |
|---|---|
| Decisión | Los cinco stores son `@Injectable({ providedIn: 'root' })`, como en clase (Clase 5, lámina 5). `CompraStore` y `ButacasStore` tienen `reiniciar()`, y el `Checkout` lo llama en su `ngOnDestroy` |
| Qué hace `reiniciar()` | Libera las butacas retenidas, cierra el canal de Realtime con `removeChannel` (Clase 6, lámina 13), corta el contador y vacía los signals |
| Por qué | Es el patrón de clase. La limpieza queda explícita en un método que se puede leer |
| Descartado | Proveer los stores en el componente (`providers`): no se vio en clase |
| Frase | "Cuando salís del checkout, la pantalla le pide a los stores que se reinicien: se liberan las butacas y la próxima compra arranca de cero." |

### 2.11 Carga diferida con `loadComponent` y `canMatch` en cada ruta

| | |
|---|---|
| Decisión | Cada pantalla de `/cuenta`, `/empleado` y `/admin` se carga con `loadComponent` y lleva su guard en `canMatch`: `clienteGuard`, `empleadoGuard` o `adminGuard`. El menú de cada rol lo arma `MenuPrincipal` con `computed` sobre el rol (sección 2.24) |
| Origen | `loadComponent` es de Clase 3, lámina 6, y `canMatch` con `adminGuard`, de Clase 7, lámina 7 |
| Descartado | `loadChildren` con un archivo de rutas por área: no se vio en clase |
| Por qué | **Rendimiento**: cada usuario descarga solo las pantallas que usa. `canMatch` se evalúa antes de descargar el chunk, así que quien no tiene el rol no lo baja |
| Lo que **no** es | No es seguridad. "El guard no es seguridad" (Clase 6, lámina 17): el código de Angular llega al navegador y cualquiera puede leerlo |
| Qué da la seguridad | El **RLS** y las **RPC**: aunque alguien llegue a la pantalla del admin, la base le rechaza cada lectura y cada escritura |
| Service worker | Precachea solo lo inicial. Con `namedChunks: true`, cada pantalla tiene su chunk con nombre (`peliculas-admin-*.js`) y queda en `installMode: lazy`. Se comprueba en `ngsw.json`. Es un desvío a propósito respecto de Clase 8, lámina 21, que precachea todo el JS |
| Frase | "La carga diferida es para que la app arranque rápido; lo que protege los datos es la base, no el router." |

### 2.12 Pipes, directivas, validador y guards propios

| | |
|---|---|
| Decisión | Pipes `duracion`, `precioArs` y `edad`; directivas `appHighlightButaca` (de atributo) y `*appRestrictEdad` (estructural); `butacasContiguas` como regla de Signal Forms; `pendingChangesGuard` y `peliculaResolver` |
| Por qué | Los pide la cátedra (RNF-26 a RNF-28) y cada uno sigue el modelo de clase:<br>• `appHighlight`, de atributo con `host` (Clase 7, lámina 14).<br>• `*appIfOverdue`, estructural con `TemplateRef` y `ViewContainerRef` (Clase 7, lámina 15).<br>• `pendingChangesGuard` y el resolver (Clase 8, láminas 6 y 7). |
| `*appRestrictEdad` | Muestra el bloque Comprar solo si la edad alcanza. El motivo no lo muestra la directiva sino el componente, con `@if (motivoEdad(); as motivo)` y la misma regla `motivoRestriccionEdad` |
| Lógica en reglas puras | Los pipes llaman a `formatearDuracion`, `formatearPrecio` y `calcularEdad`. La directiva y el componente llaman a `motivoRestriccionEdad`, y el validador, a `motivoNoContiguas`. Esas funciones son las que se prueban con Vitest |
| Frase | "La lógica vive en una función pura; el pipe, la directiva y el validador son solo la forma de enchufarla en Angular, como lo vimos en clase." |

### 2.13 Auditoría desde la base

| | |
|---|---|
| Decisión | Un trigger genérico `auditar_cambio` guarda `to_jsonb(old)` y `to_jsonb(new)`. Las RPC registran validaciones, cancelaciones, canjes y usos de cupón |
| Por qué | RN-24 pide que no dependa del frontend. Hasta un cambio hecho desde el panel de Supabase queda registrado |
| Frase | "Aunque alguien cambie un precio sin pasar por la app, la base lo anota." |

### 2.14 Tareas programadas con pg_cron, pero solo donde hacen falta

| | |
|---|---|
| Decisión | Tres jobs: limpiar retenciones cada minuto, recordatorios cada 5 minutos y alertas cada 10 |
| Por qué | Un trigger no se dispara por el paso del tiempo (S-17) |
| Los puntos | No necesitan job: cada movimiento guarda `disponible_desde` y el saldo se calcula comparando con `now()` (S-07) |
| Frase | "Usé cron solo donde el tiempo tiene que provocar algo; donde alcanza con comparar fechas, comparo fechas." |

### 2.15 Avisos: tres canales que salen de un mismo INSERT

| | |
|---|---|
| Origen | Todo aviso es una fila nueva en `notificaciones`, que inserta la base: el job `recordatorios` (cada 5 minutos), `validar_qr` en el acto o `procesar_alertas` (cada 10 minutos). Nadie dispara un aviso a mano |
| Canal 1: bandeja | `NotificacionesStore` recibe el INSERT por Realtime y lo agrega a la bandeja. Es el canal garantizado (S-10): llega aunque no haya permiso, en vivo o al abrir la app |
| Canal 2: aviso del sistema | Con la app abierta y el permiso dado, `AvisosSistema.mostrar()` lo muestra con `showNotification` del service worker |
| Canal 3: push (F16) | Con la app cerrada. El trigger `notificaciones_push` manda la fila a la Edge Function `enviar-push`, que la cifra, la firma y la envía a cada dispositivo suscripto del usuario (sección 2.23) |
| Sin duplicados | Los canales 2 y 3 usan el mismo `tag` (`notificacion-{id}`): con la app abierta, uno reemplaza al otro |
| Al tocarlo | `onActionClick` con `navigateLastFocusedOrOpen`: el service worker de Angular enfoca la app o la abre en la `url` del aviso (el QR de la compra) |
| Historia | Hasta el 09/10 no había push (D-24). La cátedra lo exigió ese día y se hizo F16 (D-25), sin cambiar los dos primeros canales |
| Frase | "La base escribe un aviso y sale por tres lados: la bandeja, que siempre llega; el aviso del sistema con la app abierta; y el push con la app cerrada." |

### 2.16 Cuentas internas sin Edge Function: un segundo cliente y una RPC

| | |
|---|---|
| Decisión | El admin crea la cuenta con `signUp` desde un segundo cliente de Supabase que no guarda sesión. Después le asigna el rol con la RPC `asignar_rol`, que verifica que quien llama sea admin (D-23) |
| Por qué | Las Edge Functions no se vieron en clase y no están en el documento. Así no hace falta la clave `service_role` en ningún lado (RNF-10) |
| ¿Y `enviar-push`? | Es la única Edge Function del proyecto y llegó después, porque la cátedra exigió el push (sección 2.23). No crea cuentas, no lee la base y tampoco usa `service_role` |
| Por qué un segundo cliente | Si el admin llamara a `signUp` con su propio cliente, perdería su sesión. El segundo cliente tiene otra `storageKey` y no guarda nada |
| Seguridad | Todo registro nace como cliente; solo `asignar_rol` cambia el rol, y solo si quien llama es admin |
| Plan B | El empleado se registra solo y el admin lo promueve con la misma RPC |
| Frase | "El admin registra la cuenta como lo haría cualquiera, y después la base, que es la única que puede, le da el rol." |

### 2.17 QR, PDF y gráficos en el navegador

| | |
|---|---|
| Decisión | `qrcode` y `jspdf` generan el comprobante en el cliente. El reporte se exporta con `jspdf` y `write-excel-file`. Los gráficos son barras con CSS y `@for`, sin librería |
| Por qué | El comprobante tiene que verse sin conexión (RNF-15). Dos gráficos de barras no justifican una librería. `write-excel-file` escribe un `.xlsx` real con una sola función; `xlsx` se descartó porque la versión de npm está desactualizada y tiene vulnerabilidades |
| Cómo funciona offline | Las librerías se cargan con archivos propios (`qr.ts` y `comprobante-pdf.ts`). Con `namedChunks`, sus chunks tienen nombre fijo y `ngsw-config.json` los precachea, aunque las áreas sigan siendo diferidas. Desde F11, `jspdf` lo comparten el comprobante y el reporte, así que el build lo saca a un `chunk-*.js`, que también está en el grupo `inicial` |
| Frase | "El comprobante se arma en el teléfono, por eso funciona en modo avión." |

### 2.18 Scripts SQL locales, pegados en el SQL Editor

| | |
|---|---|
| Decisión | `supabase/sql/01_base.sql` (configuración y precios), `supabase/sql/tp1-sistema-cine.sql` (todo el resto) y `02_permisos_anon.sql` (le quita a `anon` las RPC que necesitan sesión), pegados en el SQL Editor como en Clase 6, lámina 5. Las fases 2 a 11 no necesitaron SQL nuevo: se verificó por lectura que lo de cada fase ya estaba. F16 sumó `03_push.sql` y `04_quita_demo_avisos.sql` |
| Por qué uno solo para el resto | Lo pediste así. Cada tabla nace con todas sus FK y cada función en su versión final, sin `alter table` ni `create or replace` entre fases |
| Por qué no están en el repo | Decisión tuya: la consigna no los pide y el docente no los mencionó. La carpeta está en `.git/info/exclude` |
| Copia | La carpeta está dentro de OneDrive, así que tiene copia en la nube aunque no esté en GitHub |
| Transacción | Cada script va entre `begin;` y `commit;`: si una sentencia falla, no se aplica nada y la base queda como estaba |
| Cambios | Un script aplicado no se edita. Un cambio va en un script nuevo numerado desde `02` |
| Cómo se probó | En una Postgres en memoria (PGlite) con imitaciones de `auth`, `storage` y `cron`: 151 controles con cliente, anónimo, empleado y admin |
| Si el docente pregunta | Mostrar la base en el panel de Supabase (Table Editor, Database → Functions y Authentication → Policies) y, si la quiere ver, el script local |
| Frase | "La base vive en Supabase. La armé con scripts que pegué en el SQL Editor, cada uno todo o nada." |

### 2.19 Modelos de TypeScript con los nombres de columna

| | |
|---|---|
| Decisión | `PeliculaModel.fecha_estreno` y no `fechaEstreno` |
| Por qué | Así `data as PeliculaModel` (el patrón de clase) es cierto sin escribir una capa de mapeo |
| Frase | "El tipo de TypeScript es el espejo de la fila." |

---

### 2.20 Identidad visual con contraste medido

| | |
|---|---|
| Decisión | Bree Serif en títulos y Poppins en el resto, en negrita en botones y precios. Fondo casi negro de tono cálido (`#12100D`), un solo violeta (`#8226EF`) y dos variantes más oscuras (`#660FCC` y `#4B0B98`). Sin aguamarina y sin bordes |
| Dónde va el violeta | Sobre el fondo da 3,1:1: alcanza para rellenos, líneas y texto grande como la marca, pero no para texto chico, que pide 4,5:1 |
| Variantes | `#4B0B98` es el mismo tono y la misma saturación que `#8226EF`, con menos luz; `#660FCC` queda en el medio. El hover usa la del medio y lo presionado y los chips, la más oscura |
| Por qué la cabecera usa el color del fondo | Sobre una superficie más clara el violeta baja a 2,9:1. Así el subrayado y la marca quedan sobre el fondo y cumplen |
| Foco del teclado | En el color del texto (16,8:1): un anillo violeta no llegaría a 3:1 sobre las tarjetas |
| Sin bordes | Sombra más una superficie apenas más clara: en un fondo casi negro la sombra sola casi no se ve |
| Tarjeta de película | Proporción 1:2 con dos filas de 3/4 y 1/4: el póster queda en 2:3 y la franja de abajo tiene el título y una línea de detalles |
| Navbar | La marca va a la izquierda y el menú en el centro (grilla `1fr auto 1fr`). El subrayado es un `::after` con `scaleX(0)` y `transform-origin: center`, que pasa a `scaleX(1)` en la sección activa, al pasar el mouse y con el foco del teclado |
| Frase | "Elegí los colores midiendo el contraste: el violeta va en botones, líneas y la marca, y nunca en texto chico." |

### 2.21 Ingreso: registro en dos pasos y guards que esperan el perfil

| | |
|---|---|
| Registro | `/registro` pide solo email y clave. Después lleva a `/cuenta/completar-perfil`, que es el mismo paso que hacen las cuentas de GitHub y Google. Así hay un solo formulario de perfil (`FormularioPerfil`), usado en `CompletarPerfil` y en `Perfil` |
| Por qué no todo en una pantalla | Juntar credenciales y perfil en un solo envío obligaba a pasar el formulario de un componente a otro con técnicas que no se vieron en clase. Con dos pasos, cada pantalla tiene su `form()`, como en clase |
| `FormularioPerfil` | Usa `model()`, como el `PriorityPicker` de clase: la pantalla le pasa su borrador con `[(perfil)]` y recibe `guardado` |
| `listo()` | Justo después de entrar, la sesión ya cambió pero el perfil (que trae el rol) todavía se está cargando. Los guards esperan `sesion.listo()`, que espera ese perfil; si esperaran solo `inicializada`, un admin recién logueado vería `rol() === null` y terminaría en la portada |
| Carga diferida del acceso | `Login`, `Registro`, `RecuperarClave` y `RestablecerClave` van con `loadComponent`. Con `component`, Signal Forms entraba en la carga inicial y el bundle llegaba a 566 kB (aviso de 500 kB). Así quedó en 487 kB |
| `onInvalid: 'touch'` | La lámina 24 de la Clase 4 lo muestra, pero en Angular 22.2 `submit` ya marca todos los campos solo y `onInvalid` pasó a ser una función. Escrito como en la lámina no compila: el comportamiento es el mismo sin escribir nada |
| Frase | "El registro tiene dos pasos para reutilizar el mismo formulario de perfil que usan las cuentas de Google y GitHub." |

### 2.22 Acceso para la corrección

| | |
|---|---|
| Qué es | En `/login`, abajo de todo, un botón con estilo de enlace: "Acceso para la corrección". Completa el email y la clave de `admincine123@gmail.com` (rol admin) en el formulario. Se entra con **Ingresar**, como cualquier usuario |
| Por qué | Lo pidió la cátedra el 09/10: el login tenía que permitir entrar como admin sin pedir datos |
| Cómo está hecho | `usarAccesoCorreccion()` hace `borrador.set(...)` sobre el modelo del `form()`: es escribir en la signal que Signal Forms ya vigila, como en Clase 4. No se agregó ninguna técnica |
| Riesgo | La clave queda dentro del JavaScript que se publica en Vercel. Aunque el repo sea privado, cualquiera que tenga la URL puede entrar como admin, cambiar datos del catálogo o incluso cambiarle la clave a esa cuenta |
| Qué lo limita | El RLS y las RPC tratan a esa cuenta como a cualquier admin: no ve claves de nadie, no toca Auth de otros usuarios y cada cambio queda en la auditoría con su nombre |
| Cómo lo restauro | Tengo otro admin privado, que no está publicado. Si le cambian la clave a la cuenta de corrección, la repongo desde el panel de Supabase (Authentication → Users) |
| Frase | "Es un riesgo aceptado porque lo pidió la cátedra: el router no protege nada, lo protege el RLS, y cualquier cambio queda auditado y lo puedo revertir con mi admin privado." |

### 2.23 Push con la app cerrada: trigger, pg_net y una Edge Function

| | |
|---|---|
| Decisión | `after insert on notificaciones` → `net.http_post` (pg_net) → Edge Function `enviar-push` → servicio de push del navegador (Google, Mozilla o Apple) → service worker del dispositivo (D-25) |
| Por qué una Edge Function | Un push hay que cifrarlo para cada dispositivo y firmarlo con la clave VAPID privada. Postgres no puede hacer ninguna de las dos cosas, y la clave privada no puede estar en Angular, porque todo lo de Angular llega al navegador |
| Por qué pg_net | Es asíncrono: el pedido sale después del commit. Si el push tarda o falla, `validar_qr` y los jobs no se frenan, y la bandeja igual tiene el aviso |
| Quién puede llamar a la función | Solo el trigger. Manda el encabezado `x-push-secreto` con el valor guardado en Vault, y la función responde 401 a cualquier otro. Por eso `verify_jwt` está apagado: el que llama es la base, no un usuario con sesión |
| Qué no hace la función | No lee la base ni usa `service_role`. Recibe la notificación y las suscripciones ya armadas por el trigger, y solo envía |
| Claves | VAPID pública en los environments (es pública por diseño). VAPID privada solo como secreto de la función: no está en el repo ni en la base. El secreto compartido, en Vault y como secreto de la función |
| Suscripción | `SwPush.requestSubscription` (mismo paquete que `SwUpdate`) y la RPC `guardar_suscripcion_push`, que exige cliente registrado (D-08). `suscripciones_push` tiene RLS: el cliente solo lee las suyas y no puede escribir directo |
| Equipo compartido | El `endpoint` es único: si otro usuario activa avisos en el mismo navegador, la fila pasa a su cuenta. Al salir, `desuscribir()` borra la fila y da de baja el navegador antes del `signOut` |
| RF-89 completo | El trigger `notificaciones_validacion` (`before insert`) suma "· Acceso · 09/10 21:34 h" al cuerpo del aviso de QR validado, sin reescribir `validar_qr`, que ya estaba aplicada |
| Descartado | Una función en Vercel (otro servidor y otra clave que cuidar), OneSignal o Firebase (un servicio externo con sus propias cuentas y SDK) y Postgres solo (no cifra ni firma) |
| Sin demo | Se armó `/admin/avisos` para disparar los avisos en el momento y se sacó el mismo día: no está en el documento y podía parecer que los avisos son manuales. Para la demo se crean funciones a 24 h y 5 min y a 2 h y 5 min, y el job hace el resto |
| Frase | "La base no puede firmar un push, así que le pasa la notificación a una función que solo sabe hacer eso y solo le hace caso a la base." |

### 2.24 Barra con grupos y botón para ver la clave

| | |
|---|---|
| Barra | `MenuPrincipal` reemplaza la lista plana de enlaces, que con las doce pantallas del admin no entraba. Los grupos son datos en `menu.model.ts` (`GRUPOS_ADMIN`: Cine, Ventas, Gestión y Reportes) y el componente elige qué mostrar con `computed` sobre el rol |
| Computadora y celular | Desde 768 px, desplegables que se abren con el mouse o con clic y se cierran con Escape o al sacar el mouse o el foco. Más angosto, un botón abre un panel lateral con "Mi cuenta" primero y los grupos como acordeón |
| Accesibilidad | `aria-expanded` en cada botón de grupo, manejo con teclado y sin movimiento con `prefers-reduced-motion` |
| Animaciones | `transition` de CSS; el acordeón pasa `grid-template-rows` de `0fr` a `1fr`. No se usa `@angular/animations`, que no se vio en clase |
| Salir | `MenuPrincipal` emite `salir` con `output()` y `app.ts` hace la baja del push, el `signOut` y la navegación: el menú no sabe de Supabase |
| `VerClave` | `visible = model(false)`, como el `PriorityPicker` de clase. La pantalla usa `[type]="verClave() ? 'text' : 'password'"`; Signal Forms lee el mismo `value` con los dos tipos |
| Presupuesto | El CSS de `MenuPrincipal` pasaba los 4 kB por componente. En `angular.json`, `anyComponentStyle` subió a 6 kB de aviso y 8 kB de error; es el único componente que lo necesita |
| Frase | "El menú es un componente que recibe el rol y decide qué grupos mostrar; los grupos son datos, no HTML repetido." |

### 2.25 Recuperación de clave y el correo de Supabase

| | |
|---|---|
| Flujo | `/recuperar-clave` pide el email y llama a `AuthApi.recuperarClave()`, que usa `supabase.auth.resetPasswordForEmail(email, { redirectTo: origen + '/restablecer-clave' })`. Supabase manda un mail con un enlace firmado, de un solo uso y con vencimiento. Al abrirlo, `supabase-js` lee el token de la URL y arma una sesión de recuperación. `/restablecer-clave` espera `sesion.inicializada`, pide la clave nueva dos veces y llama a `AuthApi.cambiarClave()` (`supabase.auth.updateUser({ password })`) (RF-02) |
| No revela cuentas | Exista o no la cuenta, la pantalla dice lo mismo: "Si hay una cuenta con ese email, te enviamos un enlace". Si dijera "ese email no existe", cualquiera podría averiguar quién está registrado |
| Enlace vencido o usado | Si al abrir `/restablecer-clave` no hay sesión, muestra "El enlace no es válido o ya venció" y ofrece pedir otro. El formulario solo aparece con la sesión que trajo el enlace |
| Guards | `/recuperar-clave` tiene `invitadoGuard`: quien ya entró cambia la clave desde su cuenta. `/restablecer-clave` no tiene guard, porque el enlace del mail inicia una sesión y `invitadoGuard` lo echaría |
| Validación | Signal Forms: `required`, `minLength(8)` y un `validate` que compara `repetir` con `clave`. La regla de largo también la controla Supabase: si no se cumple, devuelve `weak_password` y `mensajeError` lo traduce |
| Redirect URLs | `/restablecer-clave` entra por los comodines `http://localhost:4200/**` y `https://tp1-sistema-cine.vercel.app/**` de Authentication → URL Configuration, los mismos de OAuth. Si no estuviera cubierta, Supabase mandaría el enlace a la Site URL y el usuario caería en la portada |
| Limitación: el correo | El servidor de correo que trae Supabase es solo para probar. **Solo entrega a las direcciones de los miembros de la organización del proyecto** y permite muy pocos envíos por hora. A cualquier otra dirección no le llega nada, y Supabase no devuelve error, así que la app muestra el mismo mensaje. Si se pasa del límite, devuelve `over_email_send_rate_limit`, que la app muestra como "Se enviaron demasiados mails. Probá de nuevo en un rato." |
| Por qué no se resolvió | Hace falta un SMTP propio (Resend, SendGrid, Brevo o una cuenta de Gmail con clave de aplicación) cargado en Authentication → SMTP Settings. **No se toca código**: el flujo es el mismo y solo cambia quién envía el mail. Se dejó afuera porque suma una cuenta externa y credenciales que cuidar, y el documento no lo pide. Está en "Limitaciones conocidas" del README |
| Confirmación de email | Está desactivada por la misma razón. Si estuviera activa, quien se registra con un email que no es del equipo nunca recibiría el mail y no podría entrar. Por eso la cuenta queda activa al registrarse |
| Demo | Con una dirección que sea miembro del proyecto: pedir el enlace, abrirlo, elegir la clave nueva y entrar con ella. No repetirlo antes de la defensa, para no llegar al límite por hora |
| Frase | "El flujo está completo; lo que limita es el servidor de correo de prueba de Supabase, que solo le escribe al equipo. Con un SMTP propio funciona para cualquiera sin tocar una línea de Angular." |

## 3. Arquitectura en capas

```text
Componente  ──inyecta──►  Store  ──llama──►  Api  ──usa──►  supabase.client  ──►  Postgres
   │                        │                                                     (RLS, RPC,
   │                        └──usa──►  rules/ (funciones puras)                    triggers, cron)
   └── pipes, directivas, validador ──usan──► rules/
```

Reglas que nunca se rompen:

1. **Un componente no importa `supabase.client.ts`.** Siempre pasa por un store o una Api.
2. **Una Api no guarda estado.** Hace la llamada, lanza el error si lo hay y devuelve datos (como `TaskApi` en clase).
3. **Un store no sabe de HTML.** Expone signals; el componente decide cómo mostrarlos.
4. **Una regla pura no importa Angular.** Recibe datos y devuelve datos, por eso se prueba sin TestBed. Es lo único que se prueba con Vitest.

---

## 4. Recorrido por carpetas y archivos

Columnas: archivo, qué hace, requerimiento que cubre y por qué existe. Las carpetas de organización están en inglés, como en clase (`shared/`). El dominio está en español. Los nombres de archivo son los que genera el CLI 22.2. ✅ marca lo que ya existe; lo que falta lleva la fase en que llega.

### 4.1 Raíz del proyecto

| Archivo | Qué hace | Req. | Por qué existe |
|---|---|---|---|
| `SPEC.md` (local, no se sube) | Guía de implementación | — | Decide qué y cómo antes de programar |
| `README.md` ✅ | Arquitectura, decisiones, cómo correr, qué tiene la base, usuarios y tarjetas de prueba | RNF-25 | Lo pide la consigna; es lo que lee el docente |
| `ngsw-config.json` ✅ | Qué cachea el service worker y cómo. Grupos `inicial` (prefetch: `main`, `chunk-*`, `qr` y `comprobante-pdf`), `diferido` (lazy: las pantallas) y `recursos` (fuentes). Datos: `cartelera` (red primero) e `imagenes` (caché primero) | RNF-15 | Lo genera `ng add @angular/pwa` (F7). Las pantallas de las áreas quedan en `lazy` |
| `tsconfig.json` ✅ | `strict` y `strictTemplates` | RNF-08 | El proyecto se generó sin ellos y la cátedra los pide (Clase 2, lámina 15) |
| `angular.json` ✅ | Build, presupuestos, `fileReplacements` de producción, `namedChunks`, `serviceWorker`, `allowedCommonJsDependencies` (`qrcode` y los opcionales de `jspdf`) y `skipTests` en los generadores | RNF-24 | Configuración por entorno, como Clase 6, lámina 20 |
| `public/manifest.webmanifest` ✅ y `public/icons/` ✅ | Nombre, `lang`, colores e íconos `any` y `maskable` de la app instalada | RNF-14 | Sin manifest no hay instalación (Clase 8, lámina 22). Los íconos son propios: círculo violeta con un triángulo de play |

No hay `vercel.json` porque el preset Angular de Vercel ya manda cualquier ruta a `index.html` (Clase 6, errores típicos).

### 4.2 `src/`

| Archivo | Qué hace | Req. | Por qué existe |
|---|---|---|---|
| `main.ts` ✅ | `bootstrapApplication(App, appConfig)` y registro del locale `es-AR` | RNF-01, RNF-21 | Punto de entrada standalone (Clase 2, lámina 8) |
| `index.html` ✅ | `lang="es"`, título, manifest y `theme-color` | RNF-14, RNF-20 | El idioma correcto ayuda a los lectores de pantalla |
| `styles.css` ✅ | Tokens de color, tipografía y espaciado; estilos base, foco visible y clases comunes (`.panel`, `.boton`, `.chip`, `.campo`, `.secciones`) | RNF-17, RNF-20 | La identidad se cambia en un solo lugar |
| `environments/environment.ts` ✅ | Desarrollo: URL, `supabaseAnonKey` (la clave publicable) y `vapidPublicKey` (F16) | RNF-24 | Esquema de clase (Clase 6, lámina 20). La clave es pública por diseño; la seguridad es el RLS |
| `environments/environment.prod.ts` ✅ | Lo mismo, para producción | RNF-24 | Lo reemplaza `fileReplacements` en el build de producción |

### 4.3 `src/app/` (raíz)

| Archivo | Qué hace | Req. | Por qué existe |
|---|---|---|---|
| `app.ts`, `app.html`, `app.css` ✅ | Layout con `<router-outlet />`, `MenuPrincipal` (que emite `salir`; `app.ts` da de baja el push y cierra la sesión), la bandeja, el aviso de perfil incompleto y el aviso de versión nueva con `SwUpdate`, un signal y `@if` (Clase 8, lámina 23) | RNF-19, RF-77, RNF-15 | Componente raíz, como en clase |
| `app.config.ts` ✅ | Router con inputs desde la ruta, locale y zona horaria del `DatePipe`, y `provideServiceWorker` en producción | RNF-01, RNF-21 | Todos los providers globales en un lugar |
| `app.routes.ts` ✅ | Todas las rutas. Con `component` van solo `Inicio`, `Comprobantes` (tiene que abrir sin conexión) y `Privacidad`; el resto, incluidos el acceso, la ficha (con `peliculaResolver`) y el checkout, con `loadComponent`. Las de las áreas llevan su guard en `canMatch` y los formularios, `pendingChangesGuard` | RNF-06 | Mapa de navegación entero en un archivo (Clase 4, lámina 6) |

### 4.4 `shared/`

Todo lo de esta tabla existe salvo lo marcado con la fase en que llega.

| Archivo o carpeta | Qué hace | Req. | Por qué existe |
|---|---|---|---|
| `services/supabase.client.ts` ✅ | `createClient(url, supabaseAnonKey)` | — | Un único cliente para toda la app (Clase 6, lámina 6) |
| `models/*.model.ts` ✅ (`auth`, `butaca`, `candy`, `compra`, `configuracion`, `cupon`, `fidelizacion`, `funcion`, `notificacion`, `pago`, `pelicula`, `perfil`, `sala`, `validacion`; `auditoria` y `reporte`; `menu` con los grupos de la barra; falta `resena` en F12) | Interfaces por entidad, uniones de literales, etiquetas y borradores de formulario | RNF-08 | Tipado estricto: el compilador avisa si se usa un campo que no existe |
| `api/configuracion-api.ts` ✅ | Configuración y precios | RF-39 | Lo usan precios, programación, ficha, mapa y cancelación |
| `api/auth-api.ts` ✅ | Registro, login con clave, OAuth, anónimo, recuperación, cambio de clave y salida | RF-01, RF-02, RNF-09 | Aísla `supabase.auth` |
| `api/perfil-api.ts` ✅ | Leer y guardar el perfil y los datos sensibles | RF-04, RF-86, S-16 | Separa el acceso a datos del formulario |
| `api/pelicula-api.ts`, `genero-api.ts` ✅ | Cartelera, top, próximos estrenos, ficha y ABM | RF-08 a RF-15 | — |
| `api/sala-api.ts`, `funcion-api.ts` ✅ | ABM de salas y butacas; funciones y RPC de programación | RF-16 a RF-25 | — |
| `api/ocupacion-api.ts` ✅ | Leer, retener y liberar butacas | RF-26 a RF-29 | La única puerta al mapa |
| `api/compra-api.ts` ✅ | Confirmar (con candy, cupón y crédito), historial, comprobante y `cancelar`, que devuelve el crédito acreditado | RF-05, RF-37, RF-73 | — |
| `api/candy-api.ts`, `cupon-api.ts` ✅, `fidelizacion-api.ts` (✅ el crédito; puntos y recompensas en F13) | Candy, cupones, crédito, puntos y recompensas | RF-40 a RF-50, RF-64 a RF-66, RF-75 | El cliente no lee `cupones`: los sugeridos y la validación van por RPC. El saldo de crédito sale de la vista `saldos_credito`; sin movimientos no hay fila y el saldo es 0 |
| `api/notificacion-api.ts` ✅, `resena-api.ts` (F12), `alerta-api.ts` ✅ | Bandeja, reseñas y alertas | RF-61 a RF-63, RF-69, RF-87 a RF-89 | — |
| `api/validacion-api.ts` ✅ | RPC `validar_qr` | RF-55 a RF-59 | — |
| `api/push-api.ts` ✅ | RPC `guardar_suscripcion_push` (endpoint y las dos claves del navegador) y `borrar_suscripcion_push` | RF-87 a RF-89, RNF-16 | `suscripciones_push` solo se escribe por estas RPC (F16) |
| `api/imagen-api.ts`, `cuenta-interna-api.ts`, `reporte-api.ts`, `auditoria-api.ts` ✅ | Storage, alta de cuentas internas (segundo cliente y `asignar_rol`), las tres RPC de reporte y el log (hasta 200 registros con el perfil embebido; el usuario se busca por una parte del email) | RF-06, RF-79 a RF-85, RNF-13 | — |
| `stores/sesion-store.ts` ✅ | Sesión, perfil, rol, edad y perfil completo como signals; promesa `inicializada` y `listo()`, que además espera el perfil (sección 2.21) | RNF-02 | Lo leen los guards, la directiva de edad y el menú |
| `stores/cartelera-store.ts` ✅ | Películas, top, filtros y `filtradas` con `computed`. Si falla `top_peliculas` (sin conexión), la cartelera se muestra igual | RF-13, RF-14 | El mismo patrón que `TaskList` filtrando con `computed` (Clase 2, lámina 27) |
| `stores/butacas-store.ts` ✅ | Estado del mapa, selección, contador, diferencias de combo y Realtime | RF-26 a RF-29 | En `root`, con `reiniciar()` que libera, cierra el canal y vacía |
| `stores/compra-store.ts` ✅ | Paso actual, candy, cupón, crédito y `resumen`. `creditoDisponible` es el saldo sin pasarse del total, y `confirmar` manda `credito_usado` | RF-32, RF-36, RF-44 | En `root`, con `reiniciar()` |
| `stores/notificaciones-store.ts` ✅ | Bandeja con Realtime y `noLeidas`; abre el canal antes de traer la lista | RF-89, RNF-12 | Lo muestra la navegación en todas las páginas |
| `stores/alertas-store.ts` ✅ | Ids de las películas con alerta del cliente; `activar` y `desactivar` llaman a la Api y actualizan el signal | RF-69 | Lo comparten Próximamente y la ficha: una consulta por sesión y no una por tarjeta |
| `services/pago-service.ts` ✅ | `pagar()`: llama a la RPC con la tarjeta, o sin ella si el crédito o el descuento cubren el total | S-08, RNF-07 | Servicio común: la pasarela vive en un solo archivo |
| `services/comprobantes-locales.ts` ✅ | Los últimos 20 comprobantes en `localStorage` | RNF-15, S-04 | El anónimo no tiene cuenta donde guardarlos |
| `services/avisos-sistema.ts` ✅ | Pide permiso, muestra el aviso del sistema con la app abierta, suscribe el dispositivo al push con `SwPush` (`suscribir`) y lo da de baja al salir (`desuscribir`) | RNF-16, RF-87 a RF-89 | Canales 2 y 3 de la sección 2.15. `NotificacionesStore` llama a `suscribir()` al entrar un usuario, por si el permiso ya estaba dado |
| `rules/fechas.ts` ✅ | Hora de Buenos Aires, chips de días, fechas recurrentes, fechas `dd/mm/aaaa`, desfase del reloj del servidor y `textoFechaHora` | RF-22, RF-34, RNF-21 | — |
| `rules/formato.ts` ✅ | `formatearDuracion`, `formatearPrecio`, cuenta regresiva y código `ABCD-EFGH-JK` | RNF-26 | La lógica de los pipes, probada con Vitest |
| `rules/edad.ts` ✅ | `calcularEdad`, `edadMinima`, `motivoRestriccionEdad` y la leyenda del comprobante | RF-07, RN-06 | La usan el pipe, la directiva y el componente que muestra el motivo |
| `rules/perfil.ts` ✅ | Perfil completo, borrador desde el perfil, cambios a guardar y validación de la fecha de nacimiento | RF-01, RF-86, RN-06 | Toda la lógica del formulario de perfil, probada sin TestBed |
| `rules/pelicula.ts` ✅ | Borrador del formulario, fila a guardar, filtro de la cartelera (sin tildes) y orden por estreno | RF-08, RF-13, RF-14 | — |
| `rules/sala.ts` ✅ | `etiquetaSala` y `agruparButacas` (genérica: la usan el editor de sala y el mapa) | RF-16, RF-20 | — |
| `rules/precio.ts` ✅ | Precio de entrada, preventa, diferencia del combo, entradas cubiertas por combos y resumen | RN-07, RN-08, RN-11, RN-12, RN-15 | Anticipa lo que cobra la base |
| `rules/candy.ts` ✅ | Agrupar por categoría, detalle del combo, cantidades y líneas del pedido | RF-46 a RF-49 | — |
| `rules/cupones.ts` ✅ | Estado del cupón para el admin, motivo de rechazo (repite `motivo_cupon`) y mejor cupón | RN-09 a RN-11 | — |
| `rules/cancelacion.ts` ✅ | `limiteCancelacion` y `motivoNoCancelable`: repite las condiciones de `cancelar_compra` (ya cancelada, fuera de plazo, algo usado) | RF-73, RN-18 | El botón explica por qué no se puede cancelar sin llamar a la base |
| `rules/reportes.ts` ✅ | Fechas del rango (formato, orden y conversión), atajos Hoy, 7 días y mes, totales de la facturación y agrupación de las más vistas por período | RF-79 a RF-82 | La tabla, el PDF y el Excel suman con la misma función |
| `rules/auditoria.ts` ✅ | Campos que cambiaron, valor como texto, nombre de la fila afectada, quién actuó ("Sistema" sin usuario) y el rango en momentos de Buenos Aires | RF-83, RF-85 | El log muestra solo lo que cambió, no la fila entera |
| `rules/puntos.ts` (F13) | Puntos ganados y saldos | RN-12, RN-19 | — |
| `rules/asignacion.ts` ✅ | Vista previa de la asignación de salas | RF-24, RN-01, RN-03 | Se prueba con Vitest |
| `rules/contiguidad.ts` ✅ | `motivoNoContiguas` | RN-25 | La base del validador |
| `rules/tarjeta.ts` ✅ | Luhn, vencimiento y CVV | S-08 | Avisa antes de enviar el pago |
| `guard/invitado-guard.ts` ✅ | `invitadoGuard` (`canActivate`) | RF-02 | Quien ya entró no vuelve al login |
| `guard/cliente-guard.ts`, `empleado-guard.ts`, `admin-guard.ts` ✅ | Un `CanMatchFn` por rol; sin sesión mandan al login con `volver` | RNF-06 | Como el `adminGuard` de Clase 7, lámina 7 |
| `guard/puede-comprar-guard.ts` ✅ | Visitante o cliente con perfil completo | RF-86 | — |
| `guard/pending-changes-guard.ts` ✅ | `pendingChangesGuard` (`canDeactivate`) y la interfaz `ConCambiosPendientes` | — | Clase 8, lámina 6: nadie pierde un formulario ni deja butacas retenidas sin avisar |
| `guard/pelicula-resolver.ts` ✅ | `peliculaResolver` | RF-15 | Clase 8, lámina 7: la ficha nace con su película |
| `pipes/duracion-pipe.ts` ✅ | Minutos → "2 h 15 min" | RNF-26, RF-15 | — |
| `pipes/precio-ars-pipe.ts` ✅ | Monto → "$ 12.500,00" | RNF-26, RNF-21 | — |
| `pipes/edad-pipe.ts` ✅ | Fecha de nacimiento → años | RNF-26, RF-07 | — |
| `directives/highlight-butaca.ts` ✅ | De atributo: clases, `aria-label`, `aria-pressed` y `aria-disabled` de cada butaca | RNF-27, RF-30, RNF-20 | Como `appHighlight` (Clase 7, lámina 14). El mapa no depende solo del color |
| `directives/restrict-edad.ts` ✅ | Estructural: muestra el bloque Comprar solo si la edad alcanza | RNF-27, RF-35 | Como `*appIfOverdue` (Clase 7, lámina 15). El motivo lo muestra el componente con `@if` |
| `validators/butacas-contiguas.ts` ✅ | Regla de Signal Forms sobre `validate()` | RNF-28, RF-90 | — |
| `components/` ✅ (`tarjeta-pelicula`, `mapa-butacas`, `contador-retencion`, `selector-dias`, `selector-fecha`, `selector-hora`, `selector-horario`, `indicador-pasos`, `resumen-compra`, `vista-comprobante`, `formulario-perfil`, `bandeja-notificaciones`, `grafico-barras`, `menu-principal`, `ver-clave`, `boton-alerta`; falta `estrellas` en F12) | Tarjeta de película, mapa, selectores, contador, indicador de pasos, resumen, comprobante, formulario de perfil, bandeja, barra de navegación, botón para ver la clave y botón de alerta | RNF-03 | Se reutilizan en varias pantallas o áreas |
| `utils/errores.ts` ✅ y `errores.spec.ts` ✅ | Traduce los códigos de error de la base, las restricciones por nombre y Supabase Auth a mensajes | — | Un solo lugar para los textos de error. Es la primera prueba de función pura |
| `utils/rutas.ts` ✅ | `rutaSegura`: el `volver` de la URL solo puede ser una ruta interna | RF-86 | Evita que un enlace armado lleve a otro sitio después del login |
| `utils/archivos.ts` ✅ | `nombreSeguro` | RNF-13 | La normalización de `attachments.ts` de clase |
| `utils/qr.ts`, `comprobante-pdf.ts`, `reporte-pdf.ts`, `excel.ts` ✅ | QR, PDF del comprobante, PDF del reporte (A4 apaisado, con encabezado repetido en cada página) y Excel (montos como números con formato de pesos) | RF-51 a RF-54, RF-82 | Las librerías se cargan con `import()` solo cuando se usan |

### 4.5 `publico/`

Todas existen.

| Componente | Qué hace | Req. |
|---|---|---|
| `inicio/` ✅ | Top 3 (`top_peliculas`, por ventas) y cartelera con buscador y filtros por género, clasificación, formato e idioma | RF-13, RF-14 |
| `proximamente/` ✅ | Próximos estrenos con la fecha y `BotonAlerta` debajo de cada tarjeta | RF-68, RF-69 |
| `pelicula-detalle/` ✅ | Ficha y funciones agrupadas por día, con precios por formato. Recibe la película del resolver; Comprar con `*appRestrictEdad` y el motivo con `@if`; `BotonAlerta` si es un próximo estreno. Las reseñas llegan en F12 | RF-15, RF-35, RF-62, RF-69 |
| `login/` ✅ | Email y clave, GitHub y Google, como el `Login` de clase. Vuelve a la ruta del `volver` | RF-02 |
| `registro/` ✅ | Email y clave; después lleva a completar el perfil (sección 2.21) | RF-01 |
| `recuperar-clave/`, `restablecer-clave/` ✅ | Pide el mail sin revelar si el email tiene cuenta, y elige la clave nueva con la sesión del enlace | RF-02 |
| `privacidad/` ✅ | Qué datos guarda Cineverso según la base, para qué se usan, con quién se comparten, los derechos de la Ley 25.326 y un mail de contacto. Existe porque Google pide esa URL para pasar el login a producción | RNF-22 |
| `comprobantes/` ✅ | Comprobantes guardados en el dispositivo, sin conexión | RNF-15, S-04 |
| `compra/checkout/` ✅ | Contenedor del flujo con `@switch` sobre el paso, con `IndicadorPasos` y el `ContadorRetencion` arriba. Implementa `ConCambiosPendientes` y en `ngOnDestroy` llama a `reiniciar()` de los dos stores | RF-32 |
| `compra/paso-funcion/` ✅ | Chips de días y horarios | RF-34 |
| `compra/paso-butacas/` ✅ | Mapa, contador y `butacasContiguas` | RF-26 a RF-31, RF-90 |
| `compra/paso-candy/` ✅ | Combos destacados arriba y productos por categoría, con − y +. Un combo por entrada como máximo; sin nada elegido, el botón dice "Seguir sin candy" | RF-47, RF-49 |
| `compra/paso-pago/` ✅ | Cupón (sugerido o tipeado), crédito (todo o una parte), declaración de edad y tarjeta. Si no queda nada por pagar, reemplaza la tarjeta por **Confirmar compra**. Los canjes llegan en F13 | RF-37, RF-43 a RF-45, RF-76 |
| `compra/paso-comprobante/` ✅ | QR, código, PDF y guardado local | RF-51 a RF-54 |

### 4.6 `cliente/`, `empleado/` y `admin/`

Cada pantalla se carga con `loadComponent` y tiene el guard de su rol en `canMatch`. No hay `*.routes.ts` por área ni componente de layout: el menú de cada rol lo arma `MenuPrincipal`, que va en `app.html`.

| Componente | Qué hace | Req. |
|---|---|---|
| `cliente/perfil/` ✅ | Datos (fecha de nacimiento de solo lectura) y edad con el pipe. Cupones disponibles (F9) y crédito con sus movimientos (F10). Puntos y canjes llegan en F13 | RF-04, RF-66, RF-75 |
| `cliente/completar-perfil/` ✅ | Datos del perfil para quien se registró con email o entró por OAuth; vuelve a la ruta del `volver` | RF-86 |
| `cliente/mis-compras/`, `detalle-compra/` ✅ | Historial con el estado en un chip (`@switch`), QR, PDF y cancelación. El botón aparece solo si la compra está vigente, y si no se puede cancelar queda deshabilitado con el motivo | RF-05, RF-73 |
| `cliente/mis-peliculas/` (F12) | Películas vistas con acceso a reseñar | RF-72 |
| `cliente/notificaciones/` ✅ | Bandeja completa, "Marcar todos como leídos" y el botón **Activar avisos**, con el estado del permiso en un `@switch`. Desde F16, el mismo botón suscribe el dispositivo al push | RF-87 a RF-89 |
| `empleado/validacion/` ✅ | Modo acceso o candy, escáner, ingreso manual y resultado arriba. El admin también la ve en su menú | RF-55 a RF-57 |
| `empleado/escaner-qr/` ✅ | Cámara con `@zxing/browser`; emite el código leído. Necesita HTTPS: en el celular se prueba en Vercel | RF-55 |
| `empleado/resultado-validacion/` ✅ | Resultado grande con `@switch`, con las butacas o los productos a entregar | RF-58, RF-59 |
| `admin/peliculas-admin/`, `pelicula-form/`, `generos-admin/` ✅ | Catálogo, póster y preventa | RF-08 a RF-11, RF-70 |
| `admin/salas-admin/`, `sala-butacas/` ✅ | Salas y distribución (tipo de butaca y fuera de servicio) | RF-16, RF-20 |
| `admin/funciones-admin/`, `programar-funciones/`, `funcion-form/` ✅ | Funciones, programación recurrente y vista previa | RF-21 a RF-25, RF-78 |
| `admin/selector-generos/`, `selector-formatos/`, `selector-productos/` ✅ | Controles propios con `model()` (como el `PriorityPicker` de clase) para el formulario de película, de sala y de combo | RNF-03 |
| `admin/precios-admin/` ✅ | Precio por formato, recargo VIP y porcentaje del cupón de primera compra | RF-39, RF-41 |
| `admin/categorias-admin/`, `productos-admin/`, `combos-admin/` ✅ | Candy, con una botonera `.secciones` para pasar de una a otra. Borrar algo vendido falla por la FK y el mensaje sugiere marcarlo como no disponible | RF-46, RF-48 |
| `admin/cupones-admin/` ✅, `recompensas-admin/` (F13) | Promociones. `BIENVENIDA` aparece aparte, con un enlace a Precios | RF-40, RF-42, RF-65 |
| `admin/cuentas-admin/` ✅ | Crear empleados y admins sin cerrar la sesión propia; reintentar el rol si ese paso falla; lista del personal | RF-06 |
| `admin/reportes/` ✅ | Atajos y rango de fechas; facturación por día en tres líneas (entradas, candy y combos) con descuentos, total y fila de totales; exportación a PDF y Excel; más vistas por semana o por mes y candy más vendido con `GraficoBarras` | RF-79 a RF-82, D-13 |
| `admin/auditoria/` ✅ | Log con filtros opcionales por email, acción y rango; **Ver solo sus acciones** y "Ver datos" con los campos que cambiaron | RF-83 a RF-85 |

Los formularios del admin, el registro y el perfil implementan `ConCambiosPendientes` para el `pendingChangesGuard`.

### 4.7 `supabase/sql/` (local, no se sube)

Scripts que se pegan **en orden** en el SQL Editor. Cada uno va entre `begin;` y `commit;`, y un script ya aplicado no se edita.

| Script | Qué hace | Req. |
|---|---|---|
| `01_base.sql` ✅ aplicado | `configuracion` (fila única con los parámetros del negocio) y `precios_formato`, con RLS de solo lectura. Usa `text` + `check` y no `enum`, como la tabla `tasks` de clase | RF-39 |
| `tp1-sistema-cine.sql` ✅ aplicado | Todo el resto: perfiles y roles, catálogo, salas y funciones con exclusión, ocupaciones, compras, candy, cupones, crédito, puntos, reseñas, avisos, auditoría, reportes, RLS, Storage, Realtime y pg_cron | Ver `SPEC.md`, secciones 5 y 6 |
| `02_permisos_anon.sql` ✅ aplicado | Quita a `anon` las 19 RPC que necesitan sesión; sin sesión quedan `es_admin` y `top_peliculas` | RNF-10 |
| `03_push.sql` ✅ aplicado | `pg_net`, `suscripciones_push` con RLS, las dos RPC de suscripción, el trigger que llama a `enviar-push` y el que completa el aviso de QR validado. Antes de correrlo se guardó el secreto en Vault con una consulta aparte, que no está en ningún archivo. También traía la demo (`probar_recordatorio`, `disparar_alertas` y alertas cada minuto) | RF-87 a RF-89, RNF-16 |
| `04_quita_demo_avisos.sql` ✅ aplicado | Borra las dos RPC de la demo y vuelve `alertas-venta` a 10 minutos. Verificado por lectura: las RPC no existen y el job corre `*/10` | Alcance |

`supabase/functions/enviar-push/index.ts` (local, no se sube) es el código de la Edge Function. Se desplegó pegándolo en el panel (Edge Functions → Deploy a new function), igual que el SQL va por el SQL Editor: sin CLI.

---

## 5. La base de datos explicada

**Tablas por grupo:**

| Grupo | Tablas |
|---|---|
| Usuarios | `perfiles` y `datos_sensibles` |
| Catálogo | `generos`, `peliculas` y `peliculas_generos` |
| Salas | `plantilla_butacas`, `salas` y `butacas` |
| Programación | `funciones` |
| Venta | `ocupaciones`, `compras`, `entradas` y `compras_items` |
| Candy | `categorias`, `productos`, `combos` y `combos_productos` |
| Promociones | `cupones`, `recompensas`, `puntos_movimientos` y `creditos_movimientos` |
| Comunidad | `resenas`, `alertas`, `notificaciones` y `suscripciones_push` (F16) |
| Control | `configuracion`, `precios_formato` y `auditoria` |

**Conceptos que tenés que poder explicar en una frase:**

| Concepto | En una frase |
|---|---|
| Restricción de exclusión | Prohíbe que dos filas con la misma sala tengan rangos de tiempo superpuestos. Necesita `btree_gist` |
| Rango `tstzrange` con `[)` | El inicio se incluye y el fin no: una función que libera la sala a las 20:30 deja empezar otra a las 20:30 |
| `on conflict … do update … where` | "Insertá; si ya existe, actualizá solo si la retención venció o es mía". Así se reemplazan las retenciones vencidas sin esperar al cron |
| `for update` | Bloquea las filas leídas hasta que termina la transacción, para que otra compra no las use al mismo tiempo |
| `security definer` | La función corre con los permisos de su dueño. Por eso valida quién la llama con `auth.uid()` |
| `security_invoker` en vistas | La vista respeta el RLS de quien consulta |
| Grant por columna | Permiso de `update` solo sobre ciertas columnas |
| `revoke execute` | Las funciones internas no se pueden llamar desde la API |
| Trigger | Código que corre solo ante un `insert`, `update` o `delete` |
| pg_cron | Un `cron` dentro de Postgres |
| `begin; … commit;` | Encierra el script en una transacción: o se aplica todo o no se aplica nada |
| Publicación `supabase_realtime` | Lista de tablas cuyos cambios se transmiten a los clientes suscriptos |
| pg_net | Extensión que hace pedidos HTTP desde Postgres. Es asíncrona: encola el pedido y lo manda después del commit |
| Vault | Lugar cifrado de la base para guardar secretos. Se leen con la vista `vault.decrypted_secrets`, que solo usa el dueño de la base |
| Trigger `before` y `after` | `before insert` puede cambiar la fila antes de guardarla (el cuerpo del aviso de QR); `after insert` reacciona a la fila ya guardada (el push) |

---

## 6. Flujos de punta a punta

### 6.1 Registro con email

1. `Registro` valida email y clave con Signal Forms.
2. `AuthApi.registrar` llama a `signUp`.
3. El trigger `crear_perfil` crea el perfil con rol `cliente`.
4. `Registro` espera `sesion.listo()` y lleva a `/cuenta/completar-perfil`.
5. `FormularioPerfil` valida los datos; `SesionStore.guardarPerfil` los guarda y, si hubo consentimiento, guarda el tipo de sangre en `datos_sensibles`.

### 6.2 OAuth con GitHub o Google

1. `signInWithOAuth` lleva al proveedor.
2. El proveedor vuelve a `https://xrbjzgqosscoowmkwnxf.supabase.co/auth/v1/callback`.
3. Supabase redirige a la app (localhost o Vercel, si están en Redirect URLs).
4. `supabase-js` toma la sesión de la URL y el trigger ya creó el perfil vacío.
5. Antes de comprar, `puedeComprarGuard` manda a completar el perfil (RF-86).

### 6.2 bis Recuperación de clave

1. `RecuperarClave` valida el email con Signal Forms y llama a `AuthApi.recuperarClave` (`resetPasswordForEmail` con `redirectTo` a `/restablecer-clave`).
2. La pantalla muestra el mismo mensaje exista o no la cuenta.
3. Supabase manda el mail (solo a direcciones del equipo, con el correo de prueba).
4. El enlace pasa por Supabase, que valida el token y redirige a `/restablecer-clave` con la sesión en la URL.
5. `supabase-js` toma esa sesión y `RestablecerClave` espera `sesion.inicializada`. Sin sesión, avisa que el enlace venció.
6. `AuthApi.cambiarClave` llama a `updateUser({ password })`. El usuario queda adentro con la clave nueva.

### 6.3 Compra anónima

1. El visitante elige la película y entra a `/compra/:id`.
2. Elige la función con chips de día y botones de horario.
3. Al tocar la primera butaca se crea la sesión anónima y se llama a `retener_butacas`.
4. Arranca el contador de 10 minutos.
5. Si la película es +13 o +18, en el pago marca la declaración de edad.
6. Paga y ve el comprobante con QR. El comprobante se guarda en el dispositivo.

### 6.4 El mapa en tiempo real

1. `ButacasStore` carga las butacas de la sala y las ocupaciones de la función.
2. Se suscribe al canal `ocupaciones-{id}`.
3. Cuando otra persona retiene, llega un INSERT y la butaca se pinta como "retenida por otro".
4. Cuando confirma, llega un UPDATE a `vendida`.
5. Cuando vence o se libera, llega un DELETE (con la clave primaria) y la butaca vuelve a libre.

### 6.5 `confirmar_compra`

Valida comprador, función y edad. Bloquea las retenciones y valida contigüidad. Calcula precios, candy, cupón y crédito, y valida los puntos. Simula el pago. Si aprueba, registra compra, entradas, ocupaciones vendidas, usos, movimientos y auditoría. Si rechaza, libera las butacas y devuelve el motivo.

### 6.6 Validación de QR

1. El empleado elige el modo (acceso o candy) y escanea o tipea el código.
2. `validar_qr` bloquea la compra y revisa si existe, si está cancelada, si ya se usó y la ventana horaria.
3. Si todo está bien, marca el sector, audita y notifica al cliente.
4. La pantalla muestra el resultado con `@switch`.

### 6.7 Cancelación

1. `DetalleCompra` trae el plazo de `configuracion` y calcula `motivoNoCancelable`. Si hay motivo (fuera de plazo o algo usado), el botón queda deshabilitado y el motivo se muestra al lado.
2. El cliente toca **Cancelar compra** y confirma; el `confirm()` avisa cuánto vuelve y que el cupón no se devuelve.
3. `cancelar_compra` bloquea la compra y verifica de nuevo las condiciones: dueño registrado, sigue confirmada, falta más de 2 h y nada usado.
4. Libera las butacas (los mapas abiertos se actualizan solos).
5. Acredita el total como crédito, incluida la parte que se había pagado con crédito. Anula los puntos pendientes y devuelve los canjeados.
6. El cupón no vuelve. La pantalla muestra el crédito acreditado y vuelve a traer el comprobante, que queda guardado como "Cancelada".

### 6.7 bis Uso del crédito

1. `PasoPago` trae el saldo una vez por compra (`saldos_credito`).
2. El cliente usa todo con **Usar** o tipea una parte. El resumen resta el crédito después del cupón y muestra "A pagar con tarjeta".
3. `confirmar_compra` recibe `p_credito`. Si supera el saldo, lanza `CREDITO_INSUFICIENTE`. Si no queda resto, no pide tarjeta (`p_pago` nulo).
4. Registra un movimiento `uso` negativo, así el saldo baja solo.

### 6.7 ter Reporte de facturación y exportación

1. `Reportes` arranca con el día de hoy. El admin puede usar un atajo (Hoy, Últimos 7 días, Este mes) o escribir el rango en dd/mm/aaaa.
2. `ReporteApi` llama a las tres RPC a la vez (`Promise.all`). Cada una verifica `es_admin()` y cuenta solo las compras con `estado = 'confirmada'`: las canceladas quedan afuera.
3. La facturación se agrupa por el día de la compra; las más vistas, por el día de la función (S-24).
4. La tabla suma la fila de totales con `totalFacturacion`, la misma función que usan el PDF y el Excel.
5. **Exportar PDF** y **Exportar Excel** cargan `reporte-pdf.ts` o `excel.ts` con `import()` y descargan el archivo del rango que se ve, no del que se está escribiendo.

### 6.7 quater Filtro de la auditoría

1. `Auditoria` muestra los 200 registros más recientes.
2. Con un email (o una parte), `AuditoriaApi` busca primero los perfiles que coinciden y filtra `auditoria` por esos ids. Suma la acción y el rango (desde las 00:00 del primer día hasta las 00:00 del día siguiente al último, en hora argentina).
3. El RLS deja leer solo al admin. Nadie puede escribir: la tabla no tiene políticas de alta, cambio ni baja.
4. Cada registro muestra quién actuó; **Ver solo sus acciones** filtra por su email. "Ver datos" lista los campos con `cambiosAuditoria`.

### 6.8 Recordatorio de 24 h y 2 h

1. Cada 5 minutos, el job `recordatorios` (pg_cron) inserta las notificaciones de 24 h y 2 h en `notificaciones`. Solo para clientes registrados (D-08) y sin avisar a compras hechas con menos de 24 h o 2 h de anticipación.
2. Realtime las entrega a `NotificacionesStore`, que las agrega a la bandeja.
3. Si la app está abierta y el usuario dio permiso, `AvisosSistema` muestra el aviso del sistema con `showNotification` del service worker.
4. En paralelo, el trigger `notificaciones_push` junta las suscripciones del usuario. Si tiene alguna, lee el secreto de Vault y hace `net.http_post` a `enviar-push`.
5. La función verifica el secreto, arma el mensaje (título, cuerpo, ícono, `tag` y `onActionClick`) y lo envía con `web-push` a cada dispositivo, con 24 h de vida y urgencia alta.
6. El service worker de Angular lo muestra aunque la app esté cerrada. Si la app estaba abierta, el `tag` repetido hace que el push reemplace al aviso del paso 3.
7. Al tocarlo, se abre el QR de la compra (`/cuenta/compras/{id}`).

### 6.8 bis Activar avisos, QR validado y cierre de sesión

1. El cliente toca **Activar avisos**. `AvisosSistema.activar()` pide el permiso desde ese clic (el navegador no lo pide sin un gesto) y llama a `suscribir()`.
2. `SwPush.requestSubscription` con la VAPID pública devuelve el endpoint y las claves del navegador. `PushApi.guardar` los manda a `guardar_suscripcion_push`.
3. Al entrar otra vez, si el permiso ya estaba dado, `NotificacionesStore` vuelve a suscribir sin preguntar.
4. El empleado valida el QR. `validar_qr` inserta el aviso y el trigger `before insert` le suma "· Acceso · 09/10 21:34 h". El push sale como en 6.8, pasos 4 a 7.
5. Al tocar **Salir**, `desuscribir()` borra la fila y da de baja el navegador. Recién después `signOut`. Si la baja falla, la salida sigue igual.

### 6.9 Programación recurrente

1. El admin elige película, formato, idioma, días (chips), hora (botones), inicio y semanas.
2. Angular expande las fechas y muestra la vista previa con `asignarSalas`.
3. Al confirmar, `programar_funciones` asigna y crea cada función.
4. La exclusión garantiza que no haya solapes aunque otro admin programe a la vez.
5. La pantalla muestra lo que se creó de verdad.

---

## 7. Conceptos de Angular para explicar

Todo lo de esta tabla se vio en clase; entre paréntesis va la clase y la lámina, para repasar. La sección 12 recorre cada decisión del proyecto con su lámina, o con el motivo si se apartó de clase.

| Concepto | En una o dos líneas | Dónde está |
|---|---|---|
| Standalone y `bootstrapApplication` (2·8) | No hay `NgModule`: cada componente declara sus `imports` y la app arranca desde el componente raíz | `main.ts` |
| Zoneless y OnPush por defecto (1·6, 2·21) | La vista se actualiza cuando cambia un signal; no hace falta zone.js | toda la app |
| `signal` / `computed` / `effect` (2·21, 4·25) | Estado / valor derivado que se recalcula solo / reacción a cambios (cargar el perfil, copiar la selección de butacas al formulario) | stores y `paso-butacas` |
| `input()`, `input.required()`, `output()`, `model()` (2·19-20, 4·10-13) | Entradas, salidas y enlace bidireccional como signals. `model()` hace que un componente funcione como control de formulario | `shared/components/` |
| `@if`, `@for` con `track`, `@switch`, `@let` (2·26, 4·18) | Control de flujo integrado; `track` le dice a Angular cómo identificar cada ítem | todas las plantillas |
| Signal Forms (3·18-19, 4·20-25) | `form(signal, validadores, { submission: { action } })`, `[formField]` y `[formRoot]`. Al enviar, `submit` marca todos los campos (lo que la lámina escribe como `onInvalid: 'touch'`) | registro, `paso-pago`, admin |
| Validador propio (3·19) | Una función que llama a `validate()` y devuelve `{ kind, message }` o `null` | `butacas-contiguas.ts` |
| `withComponentInputBinding` (3·10, 4·8) | Los parámetros de ruta y los datos del resolver llegan como `input()` del componente | `app.config.ts` |
| `loadComponent` (3·5-6, 7·7) | Carga diferida de una pantalla | `app.routes.ts` |
| `canActivate`, `canMatch`, `canDeactivate` y `resolve` (7·5-7, 8·5-8) | ¿Puede entrar? / ¿la ruta existe para este usuario (no baja el chunk)? / ¿puede salir? / ¿qué datos espera la pantalla? | `shared/guard/` |
| Guard funcional (6·17, 7·5) | Una función que usa `inject()` y devuelve `true` o un `UrlTree` con `parseUrl` | `admin-guard.ts` y los demás |
| `inject()` y `providedIn: 'root'` (5·5) | Inyección de dependencias; `root` comparte una instancia en toda la app | servicios y stores |
| `ngOnDestroy` (2·21, 6·13) | Punto de limpieza: el checkout llama a `reiniciar()` y el store cierra el canal con `removeChannel` | `checkout`, stores |
| Pipe puro (7·9-11, 8·10-13) | Solo se recalcula si cambia la entrada; acá cada pipe llama a una regla pura | `shared/pipes/` |
| Directiva de atributo con `host` (7·14, 8·15) | Agrega clases y atributos al elemento anfitrión sin tocar su plantilla | `highlight-butaca.ts` |
| Directiva estructural (7·15, 8·17-18) | `TemplateRef` es el molde y `ViewContainerRef`, el lugar; un `effect` decide si se crea la vista | `restrict-edad.ts` |
| `SwUpdate` (7·19, 8·23) | En `app.ts`: signal `actualizacionDisponible` y `@if` en `app.html` con el botón de recargar | `app.ts` |
| Service worker y `ngsw-config` (7·17-18, 8·20-21) | Proxy local que cachea la app y las respuestas de la API | PWA |

---

## 8. Preguntas probables del oral

1. **¿Qué pasa si dos personas tocan la misma butaca al mismo tiempo?**
   Las dos llaman a `retener_butacas`. Una inserta la fila y la otra choca contra la clave primaria `(funcion_id, butaca_id)`. Como la retención de la primera no venció, el `on conflict` no la reemplaza: la segunda recibe `BUTACA_NO_DISPONIBLE`, su selección vuelve atrás y ve la butaca como retenida por Realtime.

2. **¿Y si alguien retiene y cierra la pestaña?**
   La retención vence a los 10 minutos. Si otro la pide, `retener_butacas` la reemplaza. Si nadie la pide, el job de cada minuto la borra y Realtime avisa a todos.

3. **¿Por qué no validás la contigüidad al retener?**
   Porque el usuario todavía está eligiendo y puede tener dos butacas sueltas un momento. Se valida en la interfaz antes de continuar y otra vez en la base al confirmar (RN-25).

4. **¿Cómo evitás dos funciones solapadas si dos admins programan a la vez?**
   Con la restricción de exclusión: la segunda inserción falla. `programar_funciones` captura el error y prueba otra sala.

5. **¿Cómo sé que un cliente no se puede hacer admin?**
   Todo registro nace con rol `cliente`, y el cliente no puede tocar la columna `rol`. El único camino para cambiarla es la RPC `asignar_rol`, que primero verifica que quien la llama sea admin. Las cuentas internas las da de alta el admin con `signUp` desde un segundo cliente sin sesión, y después les asigna el rol con esa RPC.

6. **¿Dónde está la clave `service_role`?**
   En ningún lado: el proyecto no la usa. En el frontend está la clave publicable (en `supabaseAnonKey`), que no da permisos por sí sola: lo que se puede hacer lo deciden el RLS y las RPC (Clase 5, lámina 19).

7. **¿Por qué el pago está en una función de Postgres?**
   Para que aprobar y registrar la compra sean una sola transacción en el servidor (S-08). Si se aprobara en el navegador, cualquiera podría confirmar sin pagar.

8. **¿Se guarda la tarjeta?**
   No. Solo existe como parámetro de la llamada. No va a ninguna tabla, ni a la auditoría, ni a los mensajes de error.

9. **¿Qué tiene el QR?**
   Un código aleatorio de 10 caracteres sin datos de la compra. Sin la base no significa nada, y solo el personal puede validarlo.

10. **¿Cómo se valida dos veces el mismo QR (sala y candy)?**
    La compra tiene dos marcas: `acceso_validado_en` y `candy_validado_en`. Cada modo consume la suya; con las dos usadas, el QR queda inválido. Si se escanea una parte ya usada, muestra cuándo se usó.

11. **¿Cómo funciona el precio en preventa?**
    Si el momento de la compra cae entre las 00:00 de N días antes del estreno y las 00:00 del estreno (hora de Buenos Aires), se descuenta el porcentaje sobre el precio base. El recargo VIP se suma completo. Cuando llega el estreno, la condición deja de cumplirse y el precio vuelve solo. El admin la configura en el formulario de la película (casilla, porcentaje y días), y el formulario muestra el día en que abre con un `computed`. El combo no lleva descuento (D-02).

12. **¿Por qué los puntos no necesitan un cron?**
    Cada movimiento guarda desde cuándo está disponible: el inicio de la función. El saldo disponible suma lo que tiene esa fecha en el pasado.

13. **¿Qué pasa con los puntos si cancelo?**
    Se anulan los pendientes de esa compra con un movimiento negativo y se devuelven los que canjeaste. Nunca hay saldo negativo, porque la cancelación cierra 2 h antes de que los puntos se liberen.

14. **¿Por qué el tipo de sangre no está en `perfiles`?**
    Porque RLS no oculta columnas: el admin, que lee todos los perfiles, lo vería. Tabla aparte, solo el titular (Ley 25.326).

15. **¿Qué es un signal y en qué se diferencia de una variable común?**
    Es un valor que avisa cuando cambia. Angular sabe qué partes de la vista y qué `computed` dependen de él y actualiza solo esas.

16. **¿Para qué usás `effect`?**
    Para reaccionar a un cambio con algo que no es un valor: cargar el perfil cuando cambia el usuario, copiar la selección de butacas al formulario o decidir si la directiva de edad muestra el bloque.

17. **Si los stores son globales, ¿cómo no queda la compra anterior colgada?**
    El `Checkout` llama a `reiniciar()` de `CompraStore` y `ButacasStore` en su `ngOnDestroy`. Ahí se liberan las butacas, se cierra el canal con `removeChannel` (Clase 6, lámina 13), se corta el contador y se vacían los signals.

18. **¿Qué diferencia hay entre `canMatch` y `canActivate`? ¿Con eso está seguro el admin?**
    `canMatch` se evalúa antes de cargar el chunk diferido: si devuelve `false`, el código no se descarga. `canActivate` se evalúa con el código ya cargado.

    Eso es rendimiento, no seguridad: "el guard no es seguridad" (Clase 6, lámina 17). Cualquiera puede leer el JavaScript o llamar a la API a mano. Lo que protege los datos es el RLS (qué filas puede leer o escribir cada rol) y las RPC (que validan quién llama con `auth.uid()` y su rol).

19. **Si un visitante instala la PWA, ¿se descarga el código del admin?**
    No. `ngsw-config.json` precachea solo `main`, lo compartido (`chunk-*.js`) y los dos chunks del comprobante offline (`qr-*.js` y `comprobante-pdf-*.js`). Las pantallas de las áreas, como `peliculas-admin-*.js`, quedan en `installMode: lazy` y se guardan recién cuando alguien con el rol entra.

    Es un desvío a propósito respecto de Clase 8, lámina 21, que precachea todo el JS.

20. **¿Puede un cliente cambiarse la fecha de nacimiento para comprar una +18?**
    No. La fecha se carga una sola vez. El trigger `proteger_fecha_nacimiento` rechaza cualquier cambio que llegue por la API si ya tenía valor. Una corrección real se hace desde el SQL Editor y queda auditada.

21. **¿Cómo se comunica el mapa con el checkout?**
    `MapaButacas` recibe las butacas con `input.required()` y emite la butaca tocada con `output()`. No sabe nada de Supabase: el paso de butacas llama al store.

22. **¿Qué hace `appHighlightButaca`?**
    Es de atributo, como `appHighlight` de clase. Le pone a cada butaca las clases según tipo y estado y los atributos de accesibilidad con `host`. La forma y el ícono (estrella, cruz, rayado) hacen que no dependa solo del color.

23. **¿Por qué `appRestrictEdad` es estructural y quién muestra el motivo?**
    Porque decide si el bloque Comprar existe, que es el criterio de clase para una estructural (Clase 7, lámina 13). Funciona como `*appIfOverdue`: un `effect` calcula la regla y crea o borra la vista. El motivo lo muestra el componente con `@if (motivoEdad(); as motivo)`, usando la misma regla `motivoRestriccionEdad`, así que no se pueden contradecir.

24. **¿Qué hace el `pendingChangesGuard` en el checkout?**
    Si hay butacas retenidas y el usuario quiere salir, pregunta "Tenés butacas retenidas. Si salís, se liberan. ¿Salir igual?", con `confirm()` como en Clase 8, lámina 6. Si acepta, el `ngOnDestroy` las libera.

25. **¿Para qué el `peliculaResolver`?**
    Para que la ficha nazca con su película y no muestre un estado vacío (Clase 8, lámina 7). Con `withComponentInputBinding`, la película llega como `input`.

26. **¿Cómo funciona offline?**
    El service worker cachea la app, la cartelera (red primero) y las imágenes (caché primero). Los comprobantes se guardan en el dispositivo, y el QR y el PDF se generan en el teléfono.

27. **¿Cómo llega un recordatorio o el aviso de QR validado?**
    La base inserta la notificación y Realtime la lleva a la bandeja, que es el canal garantizado. Con la app abierta y el permiso dado, además se muestra como aviso del sistema, y al tocarlo abre la URL.

    Y con la app cerrada llega como push: el trigger `notificaciones_push` se lo pasa a la Edge Function `enviar-push`, que lo envía a cada dispositivo suscripto (secciones 2.15 y 2.23).

28. **¿Qué probaste con Vitest?**
    Solo funciones puras, sin TestBed: precio, candy, cupones, cancelación, reportes, auditoría, edad, perfil, película, sala, asignación, contigüidad, fechas, tarjeta, formato (lo que usan los pipes), nombres de archivo, rutas seguras y el traductor de errores (18 archivos, 179 pruebas al cierre de F11, igual al cierre de F16). Los puntos se suman en F13. Los pipes, la directiva de edad y el validador llaman a esas mismas funciones, así que su lógica queda cubierta.

29. **¿Por qué no usaste una librería de gráficos?**
    Son dos gráficos de barras: con `@for` y un `[style.width.%]` relativo al valor más alto alcanza. Las barras tienen `aria-hidden` y debajo hay una tabla con la clase `.solo-lectores`: no se ve, pero el lector de pantalla la lee con su título. La animación de las barras es un `@keyframes` de CSS que arranca en `width: 0` y termina en el ancho del valor, con un retraso por `$index` para que crezcan una tras otra; no hace falta el módulo de animaciones de Angular. Con `prefers-reduced-motion` se apaga.

30. **¿Qué cambiaste respecto del proyecto de clase?**
    Ver la sección 9.

31. **¿Dónde está el SQL de la base? No lo veo en el repo.**
    La base vive en Supabase y la armé con cinco scripts que pegué en el SQL Editor, como en Clase 6: la base de clase, todo el resto, el ajuste de permisos para `anon`, el push (F16) y la quita de la demo de avisos. No los subí porque la consigna no los pide. Los tengo locales y los puedo mostrar, igual que las tablas, funciones y políticas en el panel.

32. **¿Qué pasa si alguien sin sesión llama a una RPC?**
    Recibe `permission denied` antes de que la función arranque: `02_permisos_anon.sql` le quitó a `anon` el permiso de ejecutar las 19 RPC que necesitan sesión. Sin sesión solo puede llamar a `top_peliculas` (la portada) y a `es_admin`, que usa la política de lectura de películas. Igual, cada RPC controla adentro quién la llama (`SOLO_ADMIN`, `NO_PUEDE_COMPRAR`…), así que el permiso es una segunda barrera. El que compra sin cuenta no es `anon`: abre una sesión anónima y pasa a ser `authenticated`.

33. **El panel de Supabase muestra avisos. ¿Por qué no los resolviste?**
    Porque son decisiones, no errores:
    - **23 de seguridad para `authenticated`** ("Signed-In Users Can Execute SECURITY DEFINER Function"; eran 21 hasta F16, que sumó las dos RPC de suscripción): son las RPC de la app y los usuarios con sesión tienen que poder llamarlas. Son `security definer` porque escriben en tablas que el usuario no puede tocar directo (ocupaciones, compras, auditoría, suscripciones), y cada una verifica el rol adentro. Los 2 que quedan para `anon` (`es_admin` y `top_peliculas`) son a propósito.
    - **"Anonymous Access Policies"** en casi todas las tablas: el visitante que compra sin cuenta tiene una sesión anónima con el rol `authenticated`, así que el panel avisa que esas políticas también lo alcanzan. Es lo buscado: lee el catálogo y sus propias compras, y lo que exige cuenta lo cortan `es_registrado()` y `puede_comprar()`.
    - **`pg_net` en el esquema `public`** (desde F16): el script la creó sin indicar esquema. Sus funciones viven en el esquema `net`, que la API no expone, así que no queda nada nuevo para llamar desde afuera.
    - **"Leaked Password Protection Disabled"**: es el control de claves filtradas contra HaveIBeenPwned, que se configura en Auth. No lo activé.
    - **10 de rendimiento, "Multiple Permissive Policies"**: en las tablas del catálogo hay una política de lectura para todos y otra `for all` del admin, así que al leer con sesión Postgres evalúa las dos. Mantengo el `for all`, como la política "own tasks" de clase. Con el volumen de un cine no se nota.
    - **3 índices "sin uso"** (`auditoria_creado_en`, `funciones_pelicula_inicio` y `funciones_sin_solapamiento`): nunca se usaron porque la base está vacía. El último es el de la restricción de exclusión: si se borra, se pierde la regla que impide superponer funciones.
    - **30 informativos de claves foráneas sin índice**: con pocas filas no cambian nada.

34. **¿Qué pasa si pido más crédito del que tengo?**
    `confirmar_compra` lanza `CREDITO_INSUFICIENTE` en lugar de usar lo que haya con `least(...)`. Si no, se cobraría en silencio a la tarjeta más de lo que el cliente vio en el resumen. Angular igual no deja pedir más: `creditoDisponible` es el saldo sin pasarse del total. Si el saldo cambió en otra pestaña, el store saca el crédito y vuelve a traer el saldo.

35. **¿Y si el crédito cubre todo?**
    `PasoPago` cambia el formulario de la tarjeta por **Confirmar compra**, que manda `p_pago` nulo. La base solo llama a la pasarela si queda resto. Si alguien manda nulo con resto, recibe `PAGO_REQUERIDO`.

36. **¿Por qué el crédito son movimientos y no un campo saldo?**
    Porque el perfil muestra los movimientos (RF-75) y la cancelación se resuelve agregando una fila, sin recalcular nada. El saldo es una vista que suma (`saldos_credito`), y el cliente solo lee sus filas por RLS. Las escriben únicamente las RPC.

37. **Si la base ya controla la cancelación, ¿para qué la regla en Angular?**
    Para que el botón explique por qué no se puede cancelar sin tener que intentarlo (RN-18). Es la misma idea que `motivoRestriccionEdad`: Angular anticipa y la base decide. Si alguien deja la página abierta hasta que pasa el plazo, el botón sigue activo, pero la base responde `FUERA_DE_PLAZO` y se muestra ese mensaje.

38. **¿Por qué la cancelación devuelve también lo que pagué con crédito?**
    Porque `cancelar_compra` acredita el `total` de la compra, que incluye la parte pagada con crédito. Así no se pierde nada: lo pagado con tarjeta y lo pagado con crédito vuelven como crédito. El cupón sí se pierde (S-20), igual que su uso, que cuenta para el límite global.

39. **¿Cómo sabés que el reporte no cuenta las compras canceladas?**
    Las tres RPC filtran `estado = 'confirmada'`, y `cancelar_compra` pasa la compra a `cancelada`. Lo comprobé con la base: el 08/10 hay 10 compras vigentes y una cancelada de $ 16.200, y el reporte del día da $ 387.750, sin la cancelada.

40. **¿Por qué la facturación va por día de compra y las más vistas por día de función?**
    Es S-24. La plata entra el día que se paga, aunque la función sea la semana siguiente. En cambio, una película se "ve" el día de la función: si se agrupara por la compra, la preventa inflaría semanas en las que la película ni se proyectó.

41. **¿Por qué la suma la hace la base y no Angular?**
    Con una RPC viaja una fila por día en lugar de todas las compras, y la regla (qué cuenta y qué no) está en un solo lugar. Además, el cliente no puede leer las compras de otros: solo el admin, y por la RPC que verifica `es_admin()`. Angular solo suma la fila de totales de lo que ya llegó.

42. **¿Cómo se filtra la auditoría por usuario si la tabla solo tiene el id?**
    En dos pasos: busco en `perfiles` los ids cuyo email contiene lo que se tipeó y después filtro `auditoria` con `in('usuario_id', ids)`. El nombre y el email de cada registro llegan embebidos con `perfiles(nombre, apellido, email)`, por la FK. Si nadie coincide, ni consulto la auditoría.

43. **¿Puede alguien borrar o falsear la auditoría desde la app?**
    No. La tabla no tiene políticas de INSERT, UPDATE ni DELETE para nadie; solo una de SELECT para el admin. La escriben los triggers y las RPC, que son `security definer`. Quedaría el SQL Editor, que es acceso de dueño del proyecto, no de la app.

44. **¿Por qué el Excel con `write-excel-file` y no un CSV?**
    El CSV no es un Excel: según la configuración regional, Excel lo abre con las columnas pegadas o los montos como texto. El `.xlsx` lleva los montos como números con formato de pesos, así se pueden sumar, y se abre igual en Excel y en LibreOffice.

45. **¿Cómo llega un aviso con la app cerrada?**
    La base inserta la notificación. El trigger `notificaciones_push` busca las suscripciones del usuario y, con pg_net, le pasa la notificación a la Edge Function `enviar-push`. La función la cifra para cada dispositivo, la firma con la clave VAPID y la manda al servicio de push del navegador. El service worker la muestra aunque la app esté cerrada.

46. **¿Dónde está la clave VAPID privada?**
    Solo como secreto de la Edge Function. No está en el repo, ni en la base, ni en Angular. En Angular va la pública, que es pública por diseño: el navegador la usa para saber que el push viene de mi servidor.

47. **¿Cualquiera puede llamar a `enviar-push` y mandar avisos falsos?**
    No. La función solo acepta el encabezado `x-push-secreto` con el valor que la base tiene en Vault, y a todo lo demás le responde 401. `verify_jwt` está apagado porque el que llama es un trigger, no un usuario con sesión: el secreto reemplaza al JWT.

48. **¿Por qué no lo hiciste con Firebase u OneSignal?**
    Son servicios externos con sus propias cuentas, SDK y paneles, y no aportan nada que no haga `web-push`. Con la Edge Function todo queda en Supabase, que ya es el backend, y el service worker de Angular muestra el mensaje sin código extra porque le llega con el formato que espera (`notification` con `onActionClick`).

49. **Si el push falla, ¿se pierde el aviso?**
    No. pg_net manda el pedido después del commit, así que la notificación ya está guardada y aparece en la bandeja (por Realtime o al abrir la app). Por eso la bandeja es el canal garantizado (S-10) y el push es un canal más. Si el pedido falla, tampoco se frena `validar_qr` ni el job.

50. **¿Qué pasa si dos personas usan la misma computadora?**
    Al tocar **Salir**, `desuscribir()` borra la suscripción de ese navegador y la da de baja antes del `signOut`. Y si igual quedara, el `endpoint` es único: cuando el siguiente usuario activa avisos, `guardar_suscripcion_push` pasa la fila a su cuenta. Nunca recibe avisos de la cuenta anterior.

51. **¿Funciona en iPhone?**
    Solo con la PWA instalada (Safari → Compartir → Agregar a inicio) y en iOS 16.4 o superior. La PWA instalada no comparte sesión con Safari: hay que iniciar sesión y tocar **Activar avisos** adentro de la app. Lo probé en Android con la app cerrada; el iPhone queda para la demo.

52. **Con la app abierta, ¿no llega el aviso dos veces?**
    No. El aviso del sistema (por Realtime) y el push usan el mismo `tag`, `notificacion-{id}`. El sistema operativo reemplaza un aviso por otro con el mismo `tag`, así que se ve uno solo.

53. **¿Cómo le agregaste sector, fecha y hora al aviso de QR validado sin tocar `validar_qr`?**
    Con un trigger `before insert` sobre `notificaciones`, que solo corre para los tipos `validacion_acceso` y `validacion_candy`. Cambia `new.cuerpo` antes de guardar la fila. Así no reescribí una RPC que ya estaba aplicada y probada.

54. **¿Cómo demostrás los recordatorios sin esperar 24 horas?**
    Con el camino real: creo una función que empieza dentro de 24 h y 5 min y otra dentro de 2 h y 5 min, el cliente compra las dos y cierra la app. En el siguiente ciclo del job (cada 5 minutos) llegan los dos. Había armado una pantalla para dispararlos a mano y la saqué: no está en los requerimientos y podía parecer que los avisos no son automáticos.

55. **La Edge Function no se vio en clase. ¿Por qué la usaste?**
    Porque la cátedra exigió el push con la app cerrada, y no hay otra forma de firmar un push sin exponer la clave privada. Lo de Angular sí es de clase: `SwPush` es del mismo paquete que `SwUpdate` (Clase 8) y el mensaje lo muestra el service worker que ya genera `ng add @angular/pwa`. En la sección 12 está cada técnica no vista y por qué se usó.

56. **¿Cómo funciona la alerta de un próximo estreno?**
    El cliente toca **Avisame cuando salga** y se inserta una fila en `alertas`. La política de alta exige que sea cliente y que la película esté en Próximamente. Cada 10 minutos, `procesar_alertas` marca las películas que ya tienen entradas a la venta (en cartelera con una función futura, o con la preventa abierta) y les inserta un aviso a quienes tenían la alerta. Ese aviso sale por los tres canales, y `alertas_enviadas_en` garantiza que cada película avise una sola vez (S-10).

57. **¿Por qué el botón de alerta está debajo de la tarjeta y no adentro?**
    Porque la tarjeta entera es un enlace a la ficha, y el HTML no admite un botón dentro de un enlace: el lector de pantalla y el teclado no sabrían cuál de los dos se activa. Es la misma corrección que le hice a la tarjeta de taskflow (sección 9).

58. **¿Por qué la barra no usa el módulo de animaciones de Angular?**
    Porque no se vio en clase y no hace falta: los desplegables y el acordeón son `transition` de CSS (el acordeón pasa `grid-template-rows` de `0fr` a `1fr`). Con `prefers-reduced-motion` no hay movimiento.

59. **¿Cómo sabés que llegó un push y no el aviso de la app abierta?**
    Los dos se ven igual en el teléfono. La prueba es la respuesta de la función, que pg_net guarda en `net._http_response`: 200, y adentro el código del servicio de push para cada dispositivo (201 es entregado; 404 o 410, suscripción vencida). Me pasó: durante unas horas el secreto de la función no coincidía con el de Vault, respondía 401 y yo veía avisos solo porque la app estaba en segundo plano. Lo descubrí porque la alerta de venta, que salió con la app cerrada de verdad, no llegó.

60. **¿Cómo controlás que solo reseñe quien vio la película?**
    En dos capas. En la app, el botón de reseñar está solo en Mis películas, que es la vista `mis_peliculas`: compras confirmadas propias con el acceso validado o la función terminada (S-13). En la base, la política de alta de `resenas` exige `usuario_id = auth.uid()` y `vio_pelicula(pelicula_id)`, así que un INSERT armado a mano falla por RLS. El `unique (pelicula_id, usuario_id)` deja una sola por película, y la política de cambio deja editar solo la propia.

61. **¿Cómo hiciste el control de estrellas para que funcione con Signal Forms?**
    `Estrellas` tiene `value = model<number>(0)`, igual que `SelectorGeneros`. Con eso, `[formField]="resenaForm.estrellas"` lo conecta al formulario como un input más, y el `validate` exige al menos una estrella. El mismo componente, con `soloLectura`, dibuja el promedio en la ficha y la calificación de cada reseña; un promedio de 4,5 se redondea a la estrella más cercana (`estrellasLlenas`).

62. **¿De dónde sale el promedio?**
    Lo calcula la vista `peliculas_cartelera` (`round(avg(estrellas), 1)` y `count(*)`), así que la tarjeta y la ficha no hacen cuentas. Angular solo le da formato con coma (`textoPromedio`).

63. **Con la entrada validada, ¿no puede reseñar en mitad de la función?**
    Sí, y es a propósito: lo fija el documento. S-13 dice que la película entra en Mis películas cuando se valida la entrada o, si no se validó, cuando termina la función. S-14 dice que puede reseñar lo que figura en Mis películas, y RF-72 pide el acceso directo a reseñar desde ahí. Exigir que termine la función contradiría esas tres cosas. La validación en la puerta prueba que fue al cine, que es lo que S-14 busca: que la reseña refleje la experiencia en este cine.

64. **¿Cuándo pasan los puntos de pendientes a disponibles? ¿Hay un cron?**
    No hay cron. Al confirmar la compra, `confirmar_compra` inserta en la misma transacción un movimiento `ganados` por `floor(total)` con `disponible_desde` igual al inicio de la función. Pendiente es `disponible_desde > now()` y disponible es `<= now()`: cuando empieza la función la condición se cumple sola (S-07), igual que la preventa se cierra sola con la fecha (RN-08).

65. **¿Cómo evitás canjear puntos que no tenés?**
    En la app, `puedeCanjear` deshabilita el + si el costo supera los puntos que quedan o si no hay butaca libre para una entrada. Pero la autoridad es la base: `confirmar_compra` bloquea el perfil (`for update`), suma el costo de los canjes y lanza `PUNTOS_INSUFICIENTES` si supera los disponibles. Si eso pasa, la app saca los canjes y recarga el saldo. Lo canjeado vale 0, así que no suma puntos (RN-13).

66. **¿Cómo funciona la recuperación de la clave?**
    `/recuperar-clave` llama a `resetPasswordForEmail` con `redirectTo` a `/restablecer-clave`. Supabase manda un enlace firmado, de un solo uso. Al abrirlo, `supabase-js` arma una sesión de recuperación a partir de la URL, y con esa sesión `updateUser({ password })` cambia la clave. Angular nunca ve ni guarda el token: lo maneja `supabase-js`.

67. **Probé con mi email y no me llegó el mail. ¿Está roto?**
    No, el flujo está completo. El servidor de correo que trae Supabase es de prueba: solo les manda mails a los miembros de la organización del proyecto y permite muy pocos por hora. A otra dirección no le llega nada, y Supabase tampoco devuelve un error. Para que funcione con cualquier email hay que cargar un SMTP propio en Authentication → SMTP Settings, sin tocar código. Lo dejé documentado en "Limitaciones conocidas" del README y lo demuestro con una dirección del equipo.

68. **¿Por qué la pantalla dice "si hay una cuenta con ese email" y no te avisa si el email no existe?**
    Para no revelar quién está registrado. Si respondiera distinto según exista o no la cuenta, cualquiera podría probar emails y armar una lista de clientes. Es la práctica recomendada y es lo mismo que hace Supabase: no devuelve error si el email no existe.

69. **¿Por qué `/restablecer-clave` no tiene guard?**
    Porque el enlace del mail inicia sesión. Con `invitadoGuard`, que echa a quien ya está logueado, el usuario no podría ver el formulario. La protección es la sesión de recuperación: sin ella el componente no muestra el formulario, y `updateUser` sin sesión falla en Supabase.

70. **¿Por qué no está activa la confirmación de email al registrarse?**
    Por la misma limitación del correo. Si estuviera activa, quien se registra con un email que no es del equipo nunca recibiría el mail y su cuenta quedaría bloqueada. Con un SMTP propio se activa desde el panel. En la app habría que sumar un aviso de "revisá tu mail" después de registrarse, porque hoy `Registro` espera la sesión y pasa directo a completar el perfil. El login ya traduce `email_not_confirmed` ("Todavía no confirmaste tu email.").

71. **¿Qué pasa si alguien pide muchos mails seguidos?**
    Supabase corta con `over_email_send_rate_limit` y la app lo muestra como "Se enviaron demasiados mails. Probá de nuevo en un rato." Eso frena el abuso. Además, el botón se deshabilita mientras se envía (`enviando()`), así que no se dispara dos veces con un doble clic.

---

## 9. Diferencias con el proyecto de clase y por qué

Diferencias con el código de `taskflow`. Las decisiones contra las láminas están en la sección 12.

| En `taskflow` | Acá | Por qué |
|---|---|---|
| Código en inglés | Carpetas en inglés como en clase (`shared/`, `api/`, `pipes/`…); dominio en español | Lo piden los nombres de la cátedra (`duracion`, `butacasContiguas`) y el material de clase usa `shared/` |
| `AuthService` con sesión y login juntos, y acoplado a `TaskStore` | `AuthApi` (acceso) y `SesionStore` (estado); cada store lee la sesión | Capas separadas (RNF-07), sin dependencias cruzadas |
| Solo GitHub | Email, GitHub, Google y anónimo | RF-01, RF-02, RNF-09 |
| Todas las rutas se cargan al inicio, con `canActivate` | `loadComponent` y `canMatch` por rol en cada ruta, más `canDeactivate` y `resolve` | RNF-06 y Clases 7 y 8 |
| Escritura directa en la tabla | Lo crítico por RPC transaccional | RN y RNF-11 |
| Bucket público para adjuntos del usuario | Buckets públicos solo para imágenes del catálogo | No hay datos personales en Storage |
| `supebaseUrl`, environments duplicados, sin `fileReplacements` | `environment.ts` y `environment.prod.ts` con `fileReplacements` y `supabaseAnonKey` | RNF-24 y Clase 6, lámina 20 |
| Colores hex en cada CSS | Tokens en `styles.css` | RNF-17 |
| Specs de "should create" | Solo pruebas de funciones puras | RNF-23; lo más simple de mantener y explicar |
| Comentarios mezclados | Una línea, tercera persona, minúscula, sin punto | Tu regla (sección 3.1 del SPEC) |

---

## 10. Lo que exigió el docente: dónde está

**Verificado en Vercel el 09/10:** OAuth con los dos proveedores y completar el perfil, los tres pipes, `appHighlightButaca`, `butacasContiguas` (los tres mensajes y el caso válido), `*appRestrictEdad` (un menor en una +18, una ATP y sin sesión), los guards (sin sesión, cliente y empleado), la carga diferida en Network con incógnito y `ngsw.json` sin chunks de las áreas en `inicial`. La alerta de venta (F14) llegó a la bandeja a las 17:20. Al probarla con la app cerrada apareció que el push real nunca había salido (401 por el secreto); se arregló y a las 18:54 llegó en Android con la app cerrada (200, y 201 de FCM).

| Punto | Archivo o lugar | Cómo lo mostrás |
|---|---|---|
| OAuth Google y GitHub | `publico/login/`, `shared/api/auth-api.ts` | Entrar con cada uno en Vercel |
| Recordatorios de 24 h y 2 h | `procesar_recordatorios` en `tp1-sistema-cine.sql`, bandeja, `AvisosSistema` y push (`03_push.sql`, `enviar-push`) | Funciones creadas a 24 h y 5 min y a 2 h y 5 min; el cliente compra, cierra la app y en 5 a 10 minutos llegan los dos (plan de demo de F16 en el SPEC) |
| Aviso de QR validado | `validar_qr`, trigger `notificaciones_validacion`, bandeja y push | El empleado valida desde su celular con la app del cliente cerrada: llega con sector, fecha y hora |
| Push con la app cerrada | `03_push.sql`, `supabase/functions/enviar-push/`, `avisos-sistema.ts`, `push-api.ts` | **Activar avisos** en el celular del docente (en iPhone, dentro de la PWA instalada). Si no llega, la bandeja lo muestra igual |
| Alerta de venta | `BotonAlerta`, `AlertasStore`, `procesar_alertas` (job cada 10 minutos) y push | El docente activa la alerta de un estreno; el admin le crea una función y la pasa a cartelera; en hasta 10 minutos llega el aviso, también con la app cerrada |
| Entrar como admin sin pedir datos | `publico/login/`, "Acceso para la corrección" | Tocar el enlace en `/login` y después **Ingresar** |
| `butacasContiguas` | `shared/validators/butacas-contiguas.ts`, `shared/rules/contiguidad.ts` | Elegir A4 y A5 |
| Pipes `duracion`, `precioArs` y `edad` | `shared/pipes/` y `shared/rules/formato.ts` | Ficha, resumen y perfil |
| `appHighlightButaca` y `*appRestrictEdad` | `shared/directives/` | Mapa, y botón Comprar con un cliente de 16 |
| Carga diferida de `/admin` y `/empleado` con guards | `app.routes.ts`, `shared/guard/`, `ngsw-config.json` | DevTools → Network con un cliente: no baja ningún chunk del admin, y `ngsw.json` tampoco lo precachea. Si preguntan por seguridad: la dan el RLS y las RPC, no el router |

---

## 11. Glosario rápido

| Término | Significado |
|---|---|
| RLS | Row Level Security: reglas por fila que Postgres aplica a cada consulta |
| RPC | Función de Postgres que se llama desde el cliente (`supabase.rpc`) |
| PostgREST | La capa de Supabase que convierte las tablas y funciones en una API |
| JWT | El token de sesión; trae el `sub` (id del usuario) y claims como `is_anonymous` |
| Service worker | Script que corre aparte de la página y maneja la caché, la instalación y los avisos del sistema |
| Chunk | Archivo JS separado que se descarga solo cuando se necesita |
| Luhn | Dígito verificador de las tarjetas: detecta errores de tipeo |
| Idempotente | Llamarlo dos veces da el mismo resultado; `retener_butacas` recibe siempre la selección completa |
| Push | Mensaje que un servidor manda al navegador a través del servicio de push (Google, Mozilla o Apple); lo recibe el service worker aunque la app esté cerrada |
| Suscripción push | Lo que entrega el navegador al aceptar: un `endpoint` (la dirección de ese dispositivo) y dos claves para cifrarle los mensajes |
| VAPID | Par de claves que identifica al servidor que envía. La pública va en la app; la privada firma cada push y nunca sale del servidor |
| Edge Function | Función de TypeScript que Supabase corre en sus servidores (Deno). Acá, solo `enviar-push` |

---

## 12. Lo visto en clase, decisión por decisión

Criterio acordado: **en Angular, todo como en clase; la base queda como está diseñada.** Si hacía falta una técnica que no se vio, se frenaba y se elegía entre la alternativa de clase y la nueva. Entre paréntesis va la clase y la lámina (`4·25` es Clase 4, lámina 25); "taskflow" es el proyecto de clase cuando no hay lámina anotada. La fuente es la sección 13.4 del SPEC.

Frase para el oral: "En Angular hice todo como en clase. Lo que no se vio vive en la base o lo pidió la cátedra, y cada caso tiene su motivo."

### 12.1 Decisiones que salen de una lámina

| Decisión en Cineverso | Concepto de clase | Clase · lámina | Dónde está |
|---|---|---|---|
| App sin `NgModule`, arranca con `bootstrapApplication(App, appConfig)` | Componentes standalone | 2·8 | `main.ts` |
| `strict` y `strictTemplates` activados | Configuración estricta de TypeScript | 2·15 | `tsconfig.json` |
| Estado en signals y valores derivados con `computed`: filtros de la cartelera, resumen de la compra, grupos del menú | `signal`, `computed`, filtrar con `computed` como `TaskList` | 2·21, 2·27 | stores, `MenuPrincipal` |
| `effect` para reaccionar: cargar el perfil, copiar la selección de butacas al formulario, la directiva de edad. Sin `linkedSignal` | `signal` + `effect` | 4·25 | `sesion-store`, `paso-butacas`, `restrict-edad` |
| Componentes que reciben con `input()` y avisan con `output()`: el mapa emite la butaca tocada y el menú emite `salir` | Signal inputs y outputs | 2·19-20 | `mapa-butacas`, `menu-principal` |
| Controles propios con `model()`: selectores de géneros, formatos, productos, días, hora y fecha; `FormularioPerfil`; `VerClave` | `model()`, como el `PriorityPicker` | 4·10-13 | `admin/selector-*`, `shared/components/` |
| Plantillas con `@if`, `@for` con `track`, `@switch` y `@let` | Control de flujo integrado | 2·26, 4·18 | todas |
| Formularios con `form()`, `[formField]`, `[formRoot]` y `submission` | Signal Forms | 3·18-19, 4·20-25 | login, registro, pago, admin |
| `butacasContiguas` como regla sobre `validate()` | Validador propio | 3·19 | `validators/butacas-contiguas.ts` |
| "Acceso para la corrección" hace `borrador.set(...)` | El modelo del formulario es un signal | Clase 4 | `publico/login/` |
| El id de la ruta y la película del resolver llegan como `input()` | `withComponentInputBinding` | 3·10, 4·8 | `app.config.ts` |
| Cada pantalla con `loadComponent` | Carga diferida | 3·5-6 | `app.routes.ts` |
| `adminGuard`, `empleadoGuard` y `clienteGuard` en `canMatch`, funcionales con `inject()` y `parseUrl` | Guard funcional y `canMatch` | 6·17, 7·5-7 | `shared/guard/` |
| Aviso al salir de un formulario o del checkout con butacas | `canDeactivate` con `confirm()` | 7·6, 8·6 | `pending-changes-guard.ts` |
| La ficha nace con su película | `resolve` | 7·7, 8·7 | `pelicula-resolver.ts` |
| Stores y servicios con `@Injectable({ providedIn: 'root' })` e `inject()` | Inyección de dependencias | 5·5 | `shared/stores/`, `shared/services/` |
| Api delgada: llama, `if (error) throw error`, devuelve datos | `TaskApi` | taskflow | `shared/api/` |
| Un solo cliente de Supabase como constante de módulo | Cliente de Supabase | 6·6 | `supabase.client.ts` |
| Mapa y bandeja con `channel().on('postgres_changes')`, filtrando en el callback | Realtime | 6·10 | `butacas-store`, `notificaciones-store` |
| `ngOnDestroy` del checkout llama a `reiniciar()`, que cierra el canal con `removeChannel` | Limpieza al destruir | 6·13 | `checkout`, stores |
| La clave publicable va en el front; lo que protege es el RLS | Clave pública de Supabase | 5·19 | environments |
| Los guards son comodidad; la seguridad la dan el RLS y las RPC | "El guard no es seguridad" | 6·17 | sección 2.11 |
| `text` + `check (... in (...))` en lugar de `enum` | Tabla `tasks` | 5·10 | todos los scripts |
| Políticas "dueño de la fila" más un `for all` del admin | Política "own tasks" | taskflow | RLS |
| La base se arma pegando scripts en el SQL Editor | SQL Editor | 6·5 | `supabase/sql/` |
| `environment.ts` y `environment.prod.ts` con `fileReplacements` | Configuración por entorno | 6·20 | `angular.json` |
| Sin `vercel.json`: el preset Angular de Vercel resuelve el refresco | Despliegue | Clase 6, errores típicos | Vercel |
| Pósters en `{id}/{nombre normalizado}` | `attachments.ts` | taskflow | `imagen-api.ts`, `archivos.ts` |
| Pipes puros `duracion`, `precioArs` y `edad` | Pipe puro | 7·9-11, 8·10-13 | `shared/pipes/` |
| `appHighlightButaca` pone clases y `aria-*` con `host` | Directiva de atributo, como `appHighlight` | 7·14, 8·15 | `highlight-butaca.ts` |
| `*appRestrictEdad` crea o borra el bloque Comprar | Directiva estructural, como `*appIfOverdue` | 7·15, 8·17-18 | `restrict-edad.ts` |
| `ng add @angular/pwa`, manifest e íconos propios | PWA | 7·17-18, 8·20-22 | `ngsw-config.json`, `public/` |
| Aviso de versión nueva con un signal y `@if` | `SwUpdate` | 7·19, 8·23 | `app.ts` |
| `shared/` con subcarpetas en inglés | Estructura de carpetas | 8·3 | `src/app/shared/` |
| El login es `publico/login/` (`Login`, `/login`) | Login | 6·16, 8·8 | `publico/login/` |

### 12.2 Había otra opción y quedó la de clase

| Lo que había pensado | Lo que quedó | Clase · lámina | Qué se ganó |
|---|---|---|---|
| `loadChildren` con un `*.routes.ts` por área | `loadComponent` con `canMatch` en cada ruta | 3·6, 7·7 | Mismo ahorro de descarga, con la técnica de clase |
| Guard fábrica `rolGuard(roles)` | Un guard por rol | 7·7 | Cada guard se lee como el `adminGuard` de la lámina |
| Stores provistos en el componente (`providers`) | Stores en `root` con `reiniciar()` en `ngOnDestroy` | 5·5, 6·13 | La limpieza queda en un método que se puede mostrar |
| `DestroyRef` y `effect` con `onCleanup` | `ngOnDestroy` con `removeChannel` | 6·13 | Un solo punto de limpieza, el de clase |
| `linkedSignal` | `signal` + `effect` | 4·25 | Sin API nueva |
| Clase abstracta `PagoService` con `useClass` | Un `PagoService` común | 5·5 | Igual se cambia la pasarela tocando un archivo |
| Realtime con `filter` en la suscripción | Toda la tabla, filtrada en el callback | 6·10 | Como el canal de `tasks` |
| `appRestrictEdad` de atributo con `exportAs` | Estructural, y el motivo con `@if` en el componente | 7·15, 8·17 | La directiva hace una sola cosa, como `*appIfOverdue` |
| Componente `AvisoVersion` con `toSignal` | `SwUpdate` en `app.ts` con un signal | 8·23 | Copia directa de la lámina |
| `environment.development.ts` y `supabaseKey` | `environment.ts`, `environment.prod.ts` y `supabaseAnonKey` | 6·20 | Los nombres de la lámina |
| `compartido/` con subcarpetas en español | `shared/` en inglés | 8·3 | Los nombres de la lámina |
| `publico/ingresar/` | `publico/login/` | 6·16, 8·8 | Los nombres de la lámina |
| Menú plano en `app.html` | `MenuPrincipal` con `input`/`output`, `computed` y grupos como datos | 2·19-21 | El menú del admin entra en pantalla y cada rol se arma con lo de clase |
| `@angular/animations` para los desplegables | `transition` de CSS | — | No suma una técnica no vista |
| Agregados por ser de clase: `pendingChangesGuard`, `peliculaResolver` y marcar todos los campos al enviar | — | 7·6, 7·7, 8·6, 8·7, 4·24 | Usan lo de la Clase 8 aunque taskflow no lo tenga |

### 12.3 Desvíos a propósito de una lámina

| Lámina | Qué muestra | Qué hice | Por qué |
|---|---|---|---|
| 8·21 | El grupo `app` precachea todo el JS con `/*.js` | `namedChunks` y las pantallas de las áreas en `installMode: lazy` | Si se precacheara todo, cualquier visitante descargaría el código del admin y `canMatch` dejaría de ahorrar esa descarga (sección 2.11) |
| 4·24 | `onInvalid: 'touch'` en `submission` | Nada: en Angular 22.2, `submit` ya marca todos los campos y `onInvalid` es una función | Escrito como en la lámina no compila; el comportamiento es el mismo |
| taskflow | Specs "should create" con TestBed | Solo pruebas de funciones puras | RNF-23; se prueba la lógica, que es lo que se puede explicar (sección 9) |

### 12.4 Técnicas no vistas en clase y por qué se usaron

| Técnica | Quién la pide | Cómo lo explico |
|---|---|---|
| Funciones SQL y RPC (`security definer`, `search_path` vacío, `for update`, `on conflict`) | RNF-11 y la sección 5 del documento | Las reglas de dinero y butacas tienen que pasar en una transacción en el servidor (sección 2.4) |
| Triggers (auditoría, perfil nuevo, fecha de nacimiento fija, push y aviso de QR) | RN-24 y RNF-11; los demás por decisión | Lo que tiene que pasar siempre, pase por donde pase el cambio |
| pg_cron | S-17 | Un trigger no se dispara por el paso del tiempo (sección 2.14) |
| Restricción de exclusión con `btree_gist` | RN-02 | Es la herramienta de Postgres para rangos que no se pisan (sección 2.3) |
| Sesión anónima | RNF-09 | Comprar sin cuenta necesita un dueño para la retención (sección 2.7) |
| Vistas `security_invoker` y `grant` por columna | Decisión de diseño de la base | RLS filtra filas; el grant, columnas (sección 2.5) |
| Vitest solo con funciones puras | RNF-23 | Las reglas se prueban sin Angular ni red |
| `LOCALE_ID`, `registerLocaleData` y `DATE_PIPE_DEFAULT_OPTIONS` | RNF-21 | Fechas y pesos en formato argentino y hora de Buenos Aires |
| `localStorage` para los comprobantes | RNF-15 y S-04 | El anónimo no tiene cuenta donde guardarlos y tienen que abrir sin red |
| Aviso del sistema con `showNotification` | RNF-16; confirmado al cerrar F7 | En Chrome para Android `new Notification()` no está permitido |
| Edge Function `enviar-push`, `web-push`, pg_net, Vault y `SwPush` | La cátedra, el 09/10 (D-25) | Sin servidor no se firma un push. `SwPush` es del mismo paquete que `SwUpdate` (sección 2.23) |
| Segundo cliente de Supabase y la RPC `asignar_rol` | RF-06, sin Edge Functions (D-23) | El admin no pierde su sesión y nadie usa `service_role` (sección 2.16) |
| `namedChunks` y `installMode: lazy` | RNF-06 | Desvío de la lámina 8·21 (sección 12.3) |
| `qrcode`, `@zxing/browser`, `jspdf` y `write-excel-file` | RF-51, RF-53, RF-55 y RF-82 | QR, escáner, PDF y Excel; todas con `import()` solo cuando se usan |
| `window.matchMedia` en `MenuPrincipal` | Pedido del 09/10 (barra) | API del navegador: en la computadora el grupo se abre con el mouse y en el celular no |
