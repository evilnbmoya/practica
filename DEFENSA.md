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

Cada una tiene la decisión, por qué, qué descartaste y la frase para el oral. Las mismas decisiones, ordenadas por tema y separadas entre lo que se vio en la cátedra y lo que no, están en la sección 15. El push, paso a paso, en la 16.

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

Explicada a fondo, con el código de cada paso, en la sección 17.

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

El árbol completo, cómo se comunican las carpetas y la matriz de trazabilidad están en la sección 13. Columnas: archivo, qué hace, requerimiento que cubre y por qué existe. Las carpetas de organización están en inglés, como en clase (`shared/`). El dominio está en español. Los nombres de archivo son los que genera el CLI 22.2. ✅ marca lo que ya existe; lo que falta lleva la fase en que llega.

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
| `models/*.model.ts` ✅ (`auth`, `butaca`, `candy`, `compra`, `configuracion`, `cupon`, `fidelizacion`, `funcion`, `notificacion`, `pago`, `pelicula`, `perfil`, `sala`, `validacion`; `auditoria` y `reporte`; `menu` con los grupos de la barra; `resena`) | Interfaces por entidad, uniones de literales, etiquetas y borradores de formulario | RNF-08 | Tipado estricto: el compilador avisa si se usa un campo que no existe |
| `api/configuracion-api.ts` ✅ | Configuración y precios | RF-39 | Lo usan precios, programación, ficha, mapa y cancelación |
| `api/auth-api.ts` ✅ | Registro, login con clave, OAuth, anónimo, recuperación, cambio de clave y salida | RF-01, RF-02, RNF-09 | Aísla `supabase.auth` |
| `api/perfil-api.ts` ✅ | Leer y guardar el perfil y los datos sensibles | RF-04, RF-86, S-16 | Separa el acceso a datos del formulario |
| `api/pelicula-api.ts`, `genero-api.ts` ✅ | Cartelera, top, próximos estrenos, ficha y ABM | RF-08 a RF-15 | — |
| `api/sala-api.ts`, `funcion-api.ts` ✅ | ABM de salas y butacas; funciones y RPC de programación | RF-16 a RF-25 | — |
| `api/ocupacion-api.ts` ✅ | Leer, retener y liberar butacas | RF-26 a RF-29 | La única puerta al mapa |
| `api/compra-api.ts` ✅ | Confirmar (con candy, cupón y crédito), historial, comprobante y `cancelar`, que devuelve el crédito acreditado | RF-05, RF-37, RF-73 | — |
| `api/candy-api.ts`, `cupon-api.ts` ✅, `fidelizacion-api.ts` ✅ | Candy, cupones, crédito, puntos y recompensas | RF-40 a RF-50, RF-64 a RF-66, RF-75 | El cliente no lee `cupones`: los sugeridos y la validación van por RPC. El saldo de crédito sale de la vista `saldos_credito`; sin movimientos no hay fila y el saldo es 0 |
| `api/notificacion-api.ts` ✅, `resena-api.ts` ✅, `alerta-api.ts` ✅ | Bandeja, reseñas y alertas | RF-61 a RF-63, RF-69, RF-87 a RF-89 | — |
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
| `rules/puntos.ts` ✅ | Puntos ganados, disponibles y pendientes, y el texto con punto de miles | RN-12, RN-19 | — |
| `rules/asignacion.ts` ✅ | Vista previa de la asignación de salas | RF-24, RN-01, RN-03 | Se prueba con Vitest |
| `rules/contiguidad.ts` ✅ | `motivoNoContiguas` | RN-25 | La base del validador |
| `rules/tarjeta.ts` ✅ | Luhn, vencimiento y CVV | S-08 | Avisa antes de enviar el pago |
| `rules/resena.ts` ✅ | Texto del promedio, cantidad de reseñas y estrellas llenas | RF-61, RF-62 | La ficha, la tarjeta y `Estrellas` muestran el promedio igual |
| `rules/mascara.ts` ✅ | Separadores de fecha, hora, tarjeta y vencimiento mientras se escribe, y dónde queda el cursor | RNF-19 | La lógica de `appMascara`, probada con Vitest (10/10) |
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
| `directives/mascara-entrada.ts` ✅ | De atributo (`input[appMascara]`): pone los separadores mientras se escribe y le avisa el texto al formulario. Va en fechas, horas, tarjeta y vencimiento | RNF-19 | Mejora de experiencia del 10/10 (`d788b9b`); la lógica está en `rules/mascara.ts` |
| `validators/butacas-contiguas.ts` ✅ | Regla de Signal Forms sobre `validate()` | RNF-28, RF-90 | — |
| `components/` ✅ (`tarjeta-pelicula`, `mapa-butacas`, `contador-retencion`, `selector-dias`, `selector-fecha`, `selector-hora`, `selector-horario`, `indicador-pasos`, `resumen-compra`, `vista-comprobante`, `formulario-perfil`, `bandeja-notificaciones`, `grafico-barras`, `menu-principal`, `ver-clave`, `boton-alerta`, `estrellas`) | Tarjeta de película, mapa, selectores, contador, indicador de pasos, resumen, comprobante, formulario de perfil, bandeja, barra de navegación, botón para ver la clave, botón de alerta y estrellas | RNF-03 | Se reutilizan en varias pantallas o áreas |
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
| `pelicula-detalle/` ✅ | Ficha y funciones de un día por vez con los chips de `SelectorFecha`, con precios por formato. Recibe la película del resolver; Comprar con `*appRestrictEdad` y el motivo con `@if`; `BotonAlerta` si es un próximo estreno. Calificación en los detalles y sección Reseñas | RF-15, RF-35, RF-62, RF-69 |
| `login/` ✅ | Email y clave, GitHub y Google, como el `Login` de clase. Vuelve a la ruta del `volver` | RF-02 |
| `registro/` ✅ | Email y clave; después lleva a completar el perfil (sección 2.21) | RF-01 |
| `recuperar-clave/`, `restablecer-clave/` ✅ | Pide el mail sin revelar si el email tiene cuenta, y elige la clave nueva con la sesión del enlace | RF-02 |
| `privacidad/` ✅ | Qué datos guarda Cineverso según la base, para qué se usan, con quién se comparten, los derechos de la Ley 25.326 y un mail de contacto. Existe porque Google pide esa URL para pasar el login a producción | RNF-22 |
| `comprobantes/` ✅ | Comprobantes guardados en el dispositivo, sin conexión | RNF-15, S-04 |
| `compra/checkout/` ✅ | Contenedor del flujo con `@switch` sobre el paso, con `IndicadorPasos` y el `ContadorRetencion` arriba. Implementa `ConCambiosPendientes` y en `ngOnDestroy` llama a `reiniciar()` de los dos stores | RF-32 |
| `compra/paso-funcion/` ✅ | Chips de días y horarios | RF-34 |
| `compra/paso-butacas/` ✅ | Mapa, contador y `butacasContiguas` | RF-26 a RF-31, RF-90 |
| `compra/paso-candy/` ✅ | Combos destacados arriba y productos por categoría, con − y +. Un combo por entrada como máximo; sin nada elegido, el botón dice "Seguir sin candy" | RF-47, RF-49 |
| `compra/paso-pago/` ✅ | Cupón (sugerido o tipeado), crédito (todo o una parte), declaración de edad y tarjeta. Si no queda nada por pagar, reemplaza la tarjeta por **Confirmar compra**. Canje de puntos con contadores | RF-37, RF-43 a RF-45, RF-76 |
| `compra/paso-comprobante/` ✅ | QR, código, PDF y guardado local | RF-51 a RF-54 |

### 4.6 `cliente/`, `empleado/` y `admin/`

Cada pantalla se carga con `loadComponent` y tiene el guard de su rol en `canMatch`. No hay `*.routes.ts` por área ni componente de layout: el menú de cada rol lo arma `MenuPrincipal`, que va en `app.html`.

| Componente | Qué hace | Req. |
|---|---|---|
| `cliente/perfil/` ✅ | Datos (fecha de nacimiento de solo lectura) y edad con el pipe. Cupones disponibles (F9) y crédito con sus movimientos (F10). Puntos disponibles y pendientes, movimientos y recompensas (F13) | RF-04, RF-66, RF-75 |
| `cliente/completar-perfil/` ✅ | Datos del perfil para quien se registró con email o entró por OAuth; vuelve a la ruta del `volver` | RF-86 |
| `cliente/mis-compras/`, `detalle-compra/` ✅ | Historial con el estado en un chip (`@switch`), QR, PDF y cancelación. El botón aparece solo si la compra está vigente, y si no se puede cancelar queda deshabilitado con el motivo | RF-05, RF-73 |
| `cliente/mis-peliculas/` ✅ | Películas vistas con la reseña en línea (estrellas y comentario de hasta 280) | RF-61 a RF-63, RF-72 |
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
| `admin/cupones-admin/` ✅, `recompensas-admin/` ✅ | Promociones. `BIENVENIDA` aparece aparte, con un enlace a Precios | RF-40, RF-42, RF-65 |
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

---

## 13. Árbol del proyecto, relaciones y trazabilidad

La sección 4 explica archivo por archivo en tablas. Esta sección da el mapa entero: el árbol con todas las carpetas y archivos, qué función cumple cada carpeta, cómo se comunican las piezas y cómo se llega desde un requerimiento del documento hasta el archivo que lo cumple. Está armada sobre el código del 10/10 (después de `d788b9b`).

### 13.1 Los identificadores del documento

El documento de requerimientos le pone un identificador a cada ítem para poder señalarlo en una corrección ("RN-08", "S-04"). Hay cuatro prefijos, más las decisiones que agrega el SPEC:

| Prefijo | Qué es | Cuántos | Ejemplo | Cómo se explica |
|---|---|---|---|---|
| **RF** | Requerimiento funcional: **qué** hace el sistema, lo que ve o hace un usuario | 90 (RF-01 a RF-90) | RF-29: retención con contador | "Es una función que el sistema tiene que ofrecer" |
| **RN** | Regla de negocio: una condición que **tiene que cumplirse siempre**, la pida quien la pida | 25 (RN-01 a RN-25) | RN-04: una butaca se vende una sola vez por función | "Si se rompe, el cine pierde plata o vende mal. Por eso la garantiza la base" |
| **RNF** | Requerimiento no funcional: **cómo** está hecho (tecnología, calidad, seguridad, accesibilidad) | 28 (RNF-01 a RNF-28) | RNF-06: carga diferida y guards por rol | "No es una pantalla: es una propiedad de todo el sistema" |
| **S** | Supuesto: la decisión que se tomó ante algo ambiguo o contradictorio en los mails del cliente | 24 (S-01 a S-24) | S-01: J y K con 14 lugares, 532 butacas | "El cliente no lo dejó claro; elegí esto y lo justifiqué" |
| **D** | Decisión del SPEC: lo que se resolvió durante la implementación (no está en el documento) | D-01 a D-26 | D-25: push con la app cerrada | Están en la sección 13.1 del SPEC |

La diferencia que más preguntan, **RF contra RN**: RF-73 dice que el cliente puede cancelar (una función); RN-18 dice bajo qué condiciones (hasta 2 h antes, nada usado, la compra completa). El RF se cumple con una pantalla; la RN se cumple en `cancelar_compra` aunque alguien llame a la API sin pasar por la pantalla.

### 13.2 La matriz de trazabilidad

**Qué es:** una tabla que une cada requerimiento con el lugar donde se cumple, para poder demostrar que nada quedó sin hacer y que nada se hizo sin motivo. El documento trae una **matriz de trazabilidad técnica** (al final de los requerimientos no funcionales, antes de la sección 7 de supuestos) que une cada concepto técnico de la materia con los requerimientos donde se aplica.

**Cómo se encadena en este proyecto:**

```text
Documento de requerimientos ──► SPEC.md, sección 12 ──► código (este árbol) ──► pruebas y README
  RF, RN, RNF y S                 cada ID con su sección     archivo, función      rules/*.spec.ts
  matriz técnica                  de diseño y su fase        de la base o           README: arquitectura
                                  (F0 a F16)                 componente             y decisiones
```

- **Hacia adelante** (del requerimiento al código): "¿dónde está RN-25?" → SPEC 12.2 → `butacas_contiguas` en la base y `validators/butacas-contiguas.ts` en Angular → `contiguidad.spec.ts`.
- **Hacia atrás** (del código al requerimiento): "¿por qué existe `rules/cancelacion.ts`?" → RF-73 y RN-18, tabla 4.4 o las tablas de abajo.

**La matriz técnica del documento, con los archivos reales:**

| Concepto | Dónde lo pide el documento | Req. | Dónde está en el código |
|---|---|---|---|
| Componentes standalone | Toda la aplicación, con arranque por `bootstrapApplication` | RNF-01 | `src/main.ts` y `app.config.ts`. Ningún `NgModule`: cada componente declara sus `imports` |
| Signals (`signal`, `computed`, `effect`) | Stores de cartelera, compra, butacas y sesión; resumen y precio calculados | RNF-02, RF-28, RF-36 | `shared/stores/*.ts`. `CompraStore.resumen` es un `computed`; `SesionStore` y `NotificacionesStore` usan `effect` para reaccionar al usuario que entra |
| Signal inputs y outputs | Mapa de butacas, tarjeta de película y selector de fecha | RNF-03, RF-26, RF-34 | `MapaButacas` (`input.required` de butacas y `output` `alternada`), `TarjetaPelicula` (`input` de película y puesto), `SelectorFecha`; con `model()`, `SelectorGeneros`, `SelectorFormatos`, `SelectorProductos`, `Estrellas` y `VerClave` |
| Control de flujo (`@if`, `@for`, `@switch`) | Estados de butacas, listados y resultado de validación | RNF-04, RF-27, RF-58 | Todas las `.html`. Los `@switch` más claros: el paso en `checkout.html`, el resultado en `resultado-validacion.html` y el estado de la compra en `mis-compras.html`. El menú por rol sale de `computed` sobre `SesionStore.rol()` y se dibuja con `@for` |
| Signal Forms | Registro, ABM del admin, programación de funciones y checkout | RNF-05, RF-01, RF-21, RF-22 | `form()` en 23 archivos: login, registro, las dos de clave, 14 pantallas del admin, `PasoButacas`, `PasoPago`, `MisPeliculas`, `Validacion` y el componente `FormularioPerfil` |
| Lazy loading | Rutas `/admin` y `/empleado` cargadas a demanda | RNF-06, RF-55, RF-77 | `app.routes.ts` con `loadComponent`; `ngsw-config.json` deja esas pantallas en `installMode: lazy` |
| Guards funcionales | Acceso por rol a `/admin`, `/empleado` y la cuenta del cliente | RNF-06, RF-06 | `shared/guard/`: tres `CanMatchFn` por rol, `invitadoGuard`, `puedeComprarGuard`, `pendingChangesGuard` y `peliculaResolver` |
| Pipes propios | `duracion`, `precioArs` y `edad` | RNF-26, RF-15, RF-36, RF-04 | `shared/pipes/`, con la lógica en `rules/formato.ts` y `rules/edad.ts` |
| Directivas propias | `appHighlightButaca` y `appRestrictEdad` | RNF-27, RF-30, RF-35 | `shared/directives/` (más `appMascara`, que suma los separadores al escribir fechas, horas y tarjetas) |
| Validadores propios | `butacasContiguas` | RNF-28, RF-90, RN-25 | `shared/validators/butacas-contiguas.ts`, con el motivo en `rules/contiguidad.ts` |
| Servicios e inyección de dependencias | Acceso a datos separado y pago simulado intercambiable | RNF-07, S-08 | `shared/api/` (21 clases `@Injectable`), `shared/services/pago-service.ts` → RPC `confirmar_compra` → `simular_pago` |
| Supabase Auth y OAuth | Email y contraseña, Google, GitHub y sesiones anónimas | RNF-09, RF-01, RF-02, RF-86 | `api/auth-api.ts`, `stores/sesion-store.ts`, `publico/login/`, `cliente/completar-perfil/`; la anónima la abre `ButacasStore` al tocar la primera butaca |
| RLS | Todas las tablas; datos sensibles visibles solo para su titular | RNF-10, S-16 | `tp1-sistema-cine.sql` (RLS en sus 25 tablas, 48 políticas), `01_base.sql` y `03_push.sql`; `datos_sensibles` solo la lee su dueño, ni el admin |
| Realtime | Mapa de butacas y bandeja de notificaciones | RNF-12, RF-28 | `OcupacionApi` + `ButacasStore` y `NotificacionApi` + `NotificacionesStore`; en la base, `ocupaciones` y `notificaciones` en la publicación `supabase_realtime` |
| Storage | Pósters e imágenes de productos | RNF-13, RF-08, RF-46 | `api/imagen-api.ts` y los buckets `posters` y `productos` (solo el admin escribe) |
| PWA y notificaciones push | Instalación, caché, alertas, recordatorios y aviso de validación | RNF-14 a RNF-16, RF-69, RF-87 a RF-89 | `manifest.webmanifest`, `ngsw-config.json`, `services/avisos-sistema.ts`, `api/push-api.ts`, `03_push.sql` y `supabase/functions/enviar-push/` |
| Pruebas con Vitest | Reglas de precio, cupones, edad, puntos, asignación y contigüidad | RNF-23 | 21 archivos `.spec.ts`: 18 en `rules/` y 3 en `utils/` |

### 13.3 El árbol completo

Convención para no repetir: **cada carpeta de componente tiene exactamente tres archivos con su mismo nombre**, que el árbol marca como `[ts·html·css]`:

- `nombre.ts`: la clase. Inyecta stores, Apis y servicios con `inject()`, declara los signals y los `computed`, arma el formulario y maneja los eventos.
- `nombre.html`: la plantilla, con `@if`, `@for` y `@switch`.
- `nombre.css`: estilos encapsulados de ese componente; los tokens de color vienen de `styles.css`.

`(local)` marca lo que está en `.git/info/exclude` y no se sube; `(generado)` lo que crean `npm` o el CLI.

```text
tp1-sistema-cine/
├── README.md ..................... lo que lee el docente: arquitectura, decisiones, cómo correr, base, usuarios y tarjetas de prueba (RNF-25)
├── SPEC.md ....................... (local) diseño completo y plan por fases; su sección 12 es la trazabilidad
├── DEFENSA.md .................... (local) esta guía
├── package.json .................. dependencias (Angular 22, supabase-js, zxing, jspdf, qrcode, write-excel-file) y scripts start, build y test
├── package-lock.json ............. versiones exactas; Vercel instala lo mismo que en la máquina local
├── angular.json .................. build, presupuestos, fileReplacements de producción, namedChunks y serviceWorker (RNF-24, RNF-14)
├── tsconfig.json ................. strict y strictTemplates (RNF-08)
├── tsconfig.app.json ............. qué compila la app: todo src/ menos los .spec.ts
├── tsconfig.spec.json ............ qué compilan las pruebas: solo los .spec.ts, con los tipos de Vitest (RNF-23)
├── ngsw-config.json .............. qué guarda el service worker: inicial, diferido y recursos; datos cartelera e imagenes (RNF-15)
├── .editorconfig · .prettierrc ... formato del código (sangría, comillas, ancho de línea), igual en todos los archivos
├── .gitignore .................... lo que nunca se sube: node_modules, dist, la caché de .angular
├── .vscode/ ...................... extensions.json, launch.json y tasks.json: los genera el CLI para correr y depurar en VS Code
├── node_modules/ · dist/ · .angular/   (generado) dependencias instaladas, salida del build y caché del CLI
│
├── public/ ....................... se copia tal cual a la raíz del sitio publicado
│   ├── favicon.ico
│   ├── manifest.webmanifest ...... nombre, idioma, colores e íconos de la app instalada (RNF-14)
│   └── icons/ .................... icon-72x72.png a icon-512x512.png (8 tamaños), any y maskable
│
├── supabase/ ..................... (local) lo que vive en Supabase; se pega a mano en el panel, sin CLI
│   ├── sql/ ...................... se aplican en este orden en el SQL Editor; un script aplicado no se edita
│   │   ├── 01_base.sql ........... configuracion (fila única) y precios_formato, con RLS de solo lectura (RF-39)
│   │   ├── tp1-sistema-cine.sql .. el resto de la base: 25 tablas, 42 funciones, 15 triggers, 48 políticas,
│   │   │                           5 vistas, buckets, Realtime y 3 jobs de pg_cron (RNF-10, RNF-11, todas las RN)
│   │   ├── 02_permisos_anon.sql .. quita a anon las 19 RPC que necesitan sesión (RNF-10)
│   │   ├── 03_push.sql ........... pg_net, suscripciones_push, sus 2 RPC y los triggers del push (RF-87 a RF-89, RNF-16)
│   │   └── 04_quita_demo_avisos.sql  borra la demo de avisos y vuelve alertas-venta a 10 minutos (alcance)
│   └── functions/
│       └── enviar-push/
│           └── index.ts .......... Edge Function: firma con VAPID y manda el push a cada dispositivo del usuario (D-25)
│
└── src/
    ├── index.html ................ lang="es", título, manifest y theme-color (RNF-14, RNF-20)
    ├── main.ts ................... bootstrapApplication(App, appConfig) y locale es-AR (RNF-01, RNF-21)
    ├── styles.css ................ tokens de la identidad, foco visible, .solo-lectores y clases comunes (.panel, .boton, .chip, .campo) (RNF-17, RNF-20)
    ├── environments/
    │   ├── environment.ts ........ desarrollo: URL de Supabase, clave publicable y clave pública VAPID (RNF-24)
    │   └── environment.prod.ts ... producción; lo pone fileReplacements en el build
    │
    └── app/
        ├── app.ts · app.html · app.css   raíz: router-outlet, MenuPrincipal, bandeja, aviso de perfil incompleto y de versión nueva (RF-77, RNF-15)
        ├── app.config.ts ......... providers globales: router con inputs desde la ruta, LOCALE_ID, zona -0300 y service worker
        ├── app.routes.ts ......... las 37 rutas: loadComponent, guards en canMatch/canActivate/canDeactivate y resolver (RNF-06)
        │
        ├── shared/ ............... todo lo que se reutiliza; ninguna pantalla vive acá
        │   ├── services/
        │   │   ├── supabase.client.ts ...... el único cliente de Supabase de la app; solo lo importan las Apis
        │   │   ├── pago-service.ts ......... la pasarela simulada en un solo lugar: llama a confirmar_compra con o sin tarjeta (S-08, RNF-07)
        │   │   ├── comprobantes-locales.ts . los últimos 20 comprobantes en localStorage, para verlos sin conexión (RNF-15, S-04)
        │   │   └── avisos-sistema.ts ....... permiso de avisos, aviso con la app abierta y suscripción al push con SwPush (RNF-16)
        │   │
        │   ├── api/ ............... una clase por dominio; hace la llamada a Supabase y devuelve datos, sin guardar estado
        │   │   ├── auth-api.ts ............. registro, login, OAuth, sesión anónima, recuperación y salida (RF-01, RF-02, RNF-09)
        │   │   ├── perfil-api.ts ........... perfil y tipo de sangre con consentimiento (RF-04, RF-86, S-16)
        │   │   ├── cuenta-interna-api.ts ... segundo cliente sin sesión + RPC asignar_rol (RF-06)
        │   │   ├── configuracion-api.ts .... parámetros del negocio y precios por formato (RF-39)
        │   │   ├── pelicula-api.ts ......... cartelera, top 3, ficha y ABM de películas (RF-08 a RF-15)
        │   │   ├── genero-api.ts ........... ABM de géneros (RF-09)
        │   │   ├── imagen-api.ts ........... sube pósters e imágenes del candy a Storage (RNF-13)
        │   │   ├── sala-api.ts ............. salas y butacas (RF-16 a RF-20)
        │   │   ├── funcion-api.ts .......... funciones, programar_funciones y modificar_funcion (RF-21 a RF-25)
        │   │   ├── ocupacion-api.ts ........ retener y liberar butacas, y el canal Realtime del mapa (RF-26 a RF-29)
        │   │   ├── compra-api.ts ........... confirmar, historial, comprobante y cancelar (RF-05, RF-37, RF-73)
        │   │   ├── candy-api.ts ............ categorías, productos y combos (RF-46 a RF-50)
        │   │   ├── cupon-api.ts ............ ABM, sugeridos y validación por RPC (RF-40 a RF-43)
        │   │   ├── fidelizacion-api.ts ..... crédito, puntos y recompensas (RF-44, RF-64 a RF-66, RF-75)
        │   │   ├── validacion-api.ts ....... RPC validar_qr (RF-55 a RF-59)
        │   │   ├── resena-api.ts ........... reseñas y Mis películas (RF-61 a RF-63, RF-72)
        │   │   ├── alerta-api.ts ........... alertas de Próximamente (RF-69)
        │   │   ├── notificacion-api.ts ..... bandeja y su canal Realtime (RF-87 a RF-89)
        │   │   ├── push-api.ts ............. guardar y borrar la suscripción push del dispositivo (RNF-16)
        │   │   ├── reporte-api.ts .......... las tres RPC de reportes (RF-79 a RF-81)
        │   │   └── auditoria-api.ts ........ el log, con filtros (RF-83, RF-85)
        │   │
        │   ├── stores/ ............ estado compartido como signals, en root; lo leen varias pantallas (RNF-02)
        │   │   ├── sesion-store.ts ......... sesión, perfil, rol, edad y listo() para los guards
        │   │   ├── cartelera-store.ts ...... películas, top y filtros con computed (RF-13, RF-14)
        │   │   ├── butacas-store.ts ........ mapa, selección, contador y Realtime; reiniciar() (RF-26 a RF-31)
        │   │   ├── compra-store.ts ......... paso, candy, cupón, crédito, canjes y resumen; reiniciar() (RF-32, RF-36)
        │   │   ├── notificaciones-store.ts . bandeja con Realtime y noLeidas (RF-89, RNF-12)
        │   │   └── alertas-store.ts ........ películas con alerta del cliente, una consulta por sesión (RF-69)
        │   │
        │   ├── rules/ ............. funciones puras: sin Angular ni red; repiten las reglas de la base para anticipar (RNF-23)
        │   │   ├── fechas.ts · .spec ....... hora de Buenos Aires, dd/mm/aaaa, chips de días, desfase del reloj (RNF-21)
        │   │   ├── formato.ts · .spec ...... duración, precio, cuenta regresiva y código ABCD-EFGH-JK (RNF-26)
        │   │   ├── edad.ts · .spec ......... edad, edad mínima, motivo y leyenda (RF-07, RN-06)
        │   │   ├── perfil.ts · .spec ....... perfil completo, borrador y fecha de nacimiento (RF-01, RF-86)
        │   │   ├── pelicula.ts · .spec ..... borrador, filtro de la cartelera y apertura de la preventa (RF-08, RF-14, RF-70)
        │   │   ├── sala.ts · .spec ......... etiqueta de sala y butacas por bloque (RF-16, RF-20)
        │   │   ├── asignacion.ts · .spec ... vista previa de la sala de cada fecha (RF-24, RN-01, RN-03)
        │   │   ├── contiguidad.ts · .spec .. motivo de butacas no contiguas (RN-25)
        │   │   ├── precio.ts · .spec ....... entrada, preventa, combos y canjes (RN-07, RN-08, RN-13, RN-15)
        │   │   ├── candy.ts · .spec ........ agrupar, detalle del combo y cantidades (RF-46 a RF-49)
        │   │   ├── cupones.ts · .spec ...... estado, motivo de rechazo y mejor cupón (RN-09 a RN-11)
        │   │   ├── tarjeta.ts · .spec ...... Luhn, vencimiento y CVV (S-08)
        │   │   ├── cancelacion.ts · .spec .. plazo y motivo de no cancelable (RF-73, RN-18)
        │   │   ├── puntos.ts · .spec ....... puntos ganados, disponibles y pendientes (RF-64, RN-12, RN-19)
        │   │   ├── resena.ts · .spec ....... promedio, cantidad y estrellas (RF-62)
        │   │   ├── reportes.ts · .spec ..... rango, atajos y totales (RF-79 a RF-82)
        │   │   ├── auditoria.ts · .spec .... campos que cambiaron y quién actuó (RF-83, RF-85)
        │   │   └── mascara.ts · .spec ...... separadores de fecha, hora, tarjeta y vencimiento mientras se escribe (RNF-19)
        │   │
        │   ├── models/ ............ interfaces y tipos por entidad, con los nombres de columna de la base (RNF-08)
        │   │   ├── auth · perfil · menu ............................. usuarios, roles y grupos de la barra
        │   │   ├── pelicula · sala · butaca · funcion · configuracion  catálogo, salas y programación
        │   │   ├── compra · pago · candy · cupon · fidelizacion ...... venta, promociones, crédito y puntos
        │   │   ├── validacion · notificacion · resena ................ QR, avisos y comunidad
        │   │   └── reporte · auditoria ............................... panel del admin
        │   │                            (18 archivos *.model.ts)
        │   ├── guard/ ............. quién entra a cada ruta (RNF-06)
        │   │   ├── cliente-guard.ts · empleado-guard.ts · admin-guard.ts   CanMatchFn por rol; sin sesión, al login con volver
        │   │   ├── invitado-guard.ts ....... quien ya entró no vuelve al login (RF-02)
        │   │   ├── puede-comprar-guard.ts .. visitante o cliente con perfil completo (RF-86)
        │   │   ├── pending-changes-guard.ts  pregunta antes de salir con cambios o butacas retenidas
        │   │   └── pelicula-resolver.ts .... la ficha nace con su película (RF-15)
        │   ├── pipes/ ............. duracion-pipe.ts · precio-ars-pipe.ts · edad-pipe.ts (RNF-26)
        │   ├── directives/ ........ highlight-butaca.ts · restrict-edad.ts (RNF-27) · mascara-entrada.ts (RNF-19)
        │   ├── validators/ ........ butacas-contiguas.ts (RNF-28, RF-90)
        │   ├── utils/ ............. ayudas que no son reglas del negocio
        │   │   ├── errores.ts · .spec ...... traduce los errores de Supabase y de la base a mensajes
        │   │   ├── rutas.ts · .spec ........ rutaSegura: el volver solo puede ser una ruta interna
        │   │   ├── archivos.ts · .spec ..... nombre seguro y validación de imágenes (RNF-13)
        │   │   ├── qr.ts ................... genera el QR sin red (RF-53)
        │   │   ├── comprobante-pdf.ts ...... PDF del comprobante (RF-51, RF-52)
        │   │   └── reporte-pdf.ts · excel.ts  exportación del reporte (RF-82)
        │   └── components/ ........ piezas visuales reutilizables, todas [ts·html·css] (RNF-03)
        │       ├── menu-principal/ ......... barra por rol, con grupos y panel lateral en el celular (RF-77, RNF-19)
        │       ├── bandeja-notificaciones/ . campana con los avisos (RF-89)
        │       ├── tarjeta-pelicula/ ....... póster, datos y promedio de una película
        │       ├── boton-alerta/ ........... "Avisame cuando salga a la venta" (RF-69)
        │       ├── estrellas/ .............. calificación de 1 a 5, editable o de solo lectura (RF-61)
        │       ├── mapa-butacas/ ........... el mapa por bloques y pasillos (RF-26, RF-30)
        │       ├── contador-retencion/ ..... cuenta regresiva de los 10 minutos (RF-29)
        │       ├── indicador-pasos/ ........ en qué paso de la compra está (RF-32)
        │       ├── resumen-compra/ ......... desglose de la compra (RF-36)
        │       ├── vista-comprobante/ ...... QR, código, detalle y PDF (RF-51 a RF-54)
        │       ├── selector-fecha/ ......... chips de días, sin calendario con scroll (RF-34, RF-78)
        │       ├── selector-horario/ ....... chips de horarios (RF-34)
        │       ├── selector-dias/ .......... días de la semana para programar (RF-22)
        │       ├── selector-hora/ .......... hora HH:MM (RF-22)
        │       ├── formulario-perfil/ ...... datos personales, compartido por Perfil y Completar perfil (RF-04, RF-86)
        │       ├── grafico-barras/ ......... barras con su tabla para lectores de pantalla (RF-80, RF-81)
        │       └── ver-clave/ .............. botón para mostrar u ocultar la clave
        │
        ├── publico/ .............. pantallas sin guard de rol: visitante, anónimo o cliente
        │   ├── inicio/ [ts·html·css] ............ top 3 y cartelera con buscador y filtros (RF-13, RF-14)
        │   ├── proximamente/ [ts·html·css] ...... próximos estrenos con BotonAlerta (RF-68, RF-69)
        │   ├── pelicula-detalle/ [ts·html·css] .. ficha, funciones de un día por vez, precios, reseñas y Comprar (RF-15, RF-35, RF-62)
        │   ├── login/ [ts·html·css] ............. email y clave, GitHub, Google y acceso para la corrección (RF-02)
        │   ├── registro/ [ts·html·css] .......... alta con email y clave (RF-01)
        │   ├── recuperar-clave/ [ts·html·css] ... pide el mail de recuperación (RF-02)
        │   ├── restablecer-clave/ [ts·html·css] . elige la clave nueva con la sesión del enlace (RF-02)
        │   ├── privacidad/ [ts·html·css] ........ política de datos, Ley 25.326 (RNF-22)
        │   ├── comprobantes/ [ts·html·css] ...... comprobantes guardados en el dispositivo, sin conexión (RNF-15)
        │   └── compra/ ........................... el checkout: un contenedor y un componente por paso
        │       ├── checkout/ [ts·html·css] ...... @switch sobre el paso, IndicadorPasos y ContadorRetencion (RF-32)
        │       ├── paso-funcion/ [ts·html·css] .. día y horario; Comprar con *appRestrictEdad (RF-34, RF-35)
        │       ├── paso-butacas/ [ts·html·css] .. mapa en tiempo real y butacasContiguas (RF-26 a RF-31, RF-90)
        │       ├── paso-candy/ [ts·html·css] .... combos destacados y productos por categoría (RF-47, RF-49)
        │       ├── paso-pago/ [ts·html·css] ..... cupón, crédito, canjes, edad declarada y tarjeta (RF-37, RF-43 a RF-45)
        │       └── paso-comprobante/ [ts·html·css]  QR, código, PDF y guardado local (RF-51 a RF-54)
        │
        ├── cliente/ .............. con clienteGuard en canMatch
        │   ├── perfil/ [ts·html·css] ............ datos, edad, cupones, crédito y puntos (RF-04, RF-66, RF-75)
        │   ├── completar-perfil/ [ts·html·css] .. datos que faltan después del registro o de OAuth (RF-86)
        │   ├── mis-compras/ [ts·html·css] ....... historial con el estado en un chip (RF-05)
        │   ├── detalle-compra/ [ts·html·css] .... comprobante, PDF y cancelación (RF-05, RF-73)
        │   ├── mis-peliculas/ [ts·html·css] ..... películas vistas y su reseña (RF-61 a RF-63, RF-72)
        │   └── notificaciones/ [ts·html·css] .... bandeja completa y Activar avisos (RF-87 a RF-89)
        │
        ├── empleado/ ............. con empleadoGuard (empleado y admin)
        │   ├── validacion/ [ts·html·css] ........ modo acceso o candy, escáner o código manual (RF-55 a RF-57)
        │   ├── escaner-qr/ [ts·html·css] ........ cámara con @zxing/browser; emite el código leído (RF-55)
        │   └── resultado-validacion/ [ts·html·css]  resultado grande con @switch (RF-58, RF-59)
        │
        └── admin/ ................ con adminGuard
            ├── peliculas-admin/ [ts·html·css] ... listado del catálogo (RF-08, RF-11)
            ├── pelicula-form/ [ts·html·css] ..... alta y edición, póster y preventa (RF-08 a RF-11, RF-70)
            ├── selector-generos/ [ts·html·css] .. control con model() para el formulario de película (RF-09)
            ├── generos-admin/ [ts·html·css] ..... ABM de géneros (RF-09)
            ├── salas-admin/ [ts·html·css] ....... ABM de salas (RF-16)
            ├── selector-formatos/ [ts·html·css] . control con model() para los formatos de la sala (S-03)
            ├── sala-butacas/ [ts·html·css] ...... tipo de butaca y fuera de servicio (RF-20)
            ├── funciones-admin/ [ts·html·css] ... listado de funciones (RF-21)
            ├── funcion-form/ [ts·html·css] ...... editar horario, formato o idioma (RF-21, RF-25)
            ├── programar-funciones/ [ts·html·css]  programación recurrente con vista previa (RF-22 a RF-24, RF-78)
            ├── precios-admin/ [ts·html·css] ..... precio por formato, recargo VIP y cupón de primera compra (RF-39, RF-41)
            ├── categorias-admin/ [ts·html·css] .. categorías del candy (RF-46)
            ├── productos-admin/ [ts·html·css] ... productos con imagen (RF-46)
            ├── combos-admin/ [ts·html·css] ...... combos y destacados (RF-48, RF-49)
            ├── selector-productos/ [ts·html·css]  control con model() para los productos del combo (RF-48)
            ├── cupones-admin/ [ts·html·css] ..... cupones generales y por edad (RF-40, RF-42)
            ├── recompensas-admin/ [ts·html·css] . catálogo de recompensas en puntos (RF-65)
            ├── cuentas-admin/ [ts·html·css] ..... alta de empleados y admins (RF-06)
            ├── reportes/ [ts·html·css] .......... facturación, más vistas, candy, PDF y Excel (RF-79 a RF-82)
            └── auditoria/ [ts·html·css] ......... log con filtros (RF-83 a RF-85)
```

En números: 62 componentes (9 públicos, 6 del checkout, 6 de cliente, 3 de empleado, 20 de admin, 17 compartidos y la raíz), 21 Apis, 6 stores, 4 servicios, 18 reglas puras, 18 modelos, 6 guards y 1 resolver, 3 pipes, 3 directivas, 1 validador y 37 rutas (contando la redirección de `/admin` y el comodín `**`).

### 13.4 Qué función cumple cada carpeta

| Carpeta | Capa | Qué tiene adentro | Quién la usa | Qué usa ella | Regla que no se rompe |
|---|---|---|---|---|---|
| `publico/`, `cliente/`, `empleado/`, `admin/` | Presentación (pantallas) | Un componente por ruta, agrupados por quién los ve | El router (`app.routes.ts`) | `components/`, `pipes/`, `directives/`, `stores/` y `api/` | No importan `supabase.client.ts` |
| `shared/components/` | Presentación (piezas) | Componentes sin ruta, con `input`, `output` y `model` | Las pantallas | `pipes/`, `directives/`, `rules/`; `MenuPrincipal`, `BotonAlerta` y `BandejaNotificaciones` leen un store | Reciben datos y avisan eventos; casi ninguno sabe de dónde vienen los datos |
| `shared/stores/` | Estado | Signals y `computed` compartidos entre pantallas | Pantallas, guards, directivas y componentes | `api/`, `rules/`, `services/` | No saben de HTML |
| `shared/api/` | Acceso a datos | Una clase por dominio con las llamadas a tablas, vistas, RPC, Storage y Realtime | Stores y pantallas de ABM | `services/supabase.client.ts` | No guardan estado: llaman, lanzan el error y devuelven datos |
| `shared/services/` | Servicios transversales | El cliente de Supabase, el pago, los comprobantes locales y los avisos | Apis, stores y la raíz | El navegador (`localStorage`, `Notification`, `SwPush`) | Cada uno resuelve una sola cosa del sistema |
| `shared/rules/` | Lógica pura | Funciones que reciben datos y devuelven datos | Stores, pipes, validador, directivas y componentes | Solo `models/` y otras reglas | No importan Angular; por eso se prueban con Vitest |
| `shared/models/` | Tipos | Interfaces, uniones de literales, etiquetas y borradores de formulario | Todos | Nada | Los nombres son los de las columnas de la base |
| `shared/guard/` | Navegación | Guards funcionales y el resolver | `app.routes.ts` | `SesionStore` y `PeliculaApi` | Esperan `listo()` antes de decidir por rol |
| `shared/pipes/`, `directives/`, `validators/` | Presentación (lo que pidió la cátedra) | Los tres pipes, las tres directivas y el validador | Plantillas y formularios | `rules/` | La lógica vive en `rules/`, ellos solo la conectan con Angular |
| `shared/utils/` | Ayudas | Errores, rutas, archivos, QR, PDF y Excel | Pantallas y stores | Librerías externas con `import()` | No son reglas del negocio |
| `src/environments/` | Configuración | URL y claves públicas | `supabase.client.ts` y `AvisosSistema` | — | Nunca una clave secreta |
| `public/` | Recursos estáticos | Manifest e íconos | El navegador al instalar | — | — |
| `supabase/sql/` | Base de datos | Tablas, RLS, RPC, triggers, vistas y jobs | Las Apis, a través de PostgREST | — | Lo crítico se garantiza acá (RNF-11) |
| `supabase/functions/` | Servidor | `enviar-push` | El trigger `notificaciones_push` por pg_net | `web-push` y los secretos VAPID | Solo envía; no decide a quién (eso lo hace la base) |

### 13.5 Cómo se comunican

**El camino de una acción, de la pantalla a la base y de vuelta:**

```text
 URL ──► app.routes.ts ──► guard (canMatch / canActivate) ──► resolver ──► Pantalla
                                   │ lee SesionStore.listo()                 │
                                                                             │ plantilla: components/, pipes/, directives/
                                                                             │ inject(): stores/ o api/
                                                                             ▼
          rules/ ◄──usa── stores/ ──llama──► api/ ──► services/supabase.client.ts
       (funciones puras)                                      │
                                                              ▼
                                        ┌──────────────── Supabase ────────────────┐
                                        │ Auth ........ sesión, OAuth y anónima     │
                                        │ PostgREST ... tablas y vistas con RLS     │
                                        │ RPC ......... confirmar_compra, validar_qr│
                                        │ Realtime .... ocupaciones, notificaciones │
                                        │ Storage ..... posters, productos          │
                                        └───────────────────────────────────────────┘
                                                              │
               signal actualizado ◄── store ◄── respuesta o evento Realtime
                         │
                         ▼
               la plantilla se vuelve a dibujar sola
```

**Lo que pasa dentro de la base sin que nadie lo llame:**

```text
INSERT/UPDATE/DELETE ──► triggers ──► auditoria (auditar_*), perfil nuevo (auth_usuario_creado),
                                      fin de la función (preparar_funcion), butacas de la sala nueva,
                                      autor de la reseña, fecha de nacimiento fija
pg_cron ─┬─ cada minuto ........ limpiar-retenciones: borra las butacas retenidas vencidas
         ├─ cada 5 minutos ..... recordatorios: procesar_recordatorios (24 h y 2 h)
         └─ cada 10 minutos .... alertas-venta: procesar_alertas
INSERT en notificaciones ──► Realtime ──► NotificacionesStore ──► bandeja + aviso del sistema (app abierta)
                         └─► trigger notificaciones_push ──► pg_net ──► Edge Function enviar-push
                                 ──► servicio de push (FCM, Mozilla o Apple) ──► service worker ──► aviso (app cerrada)
```

**Qué usa cada pantalla** (sale de los `import` del código):

| Pantalla | Componentes, directivas y pipes compartidos | Stores y servicios | Apis | Reglas |
|---|---|---|---|---|
| `app` (raíz) | `MenuPrincipal`, `BandejaNotificaciones` | `SesionStore`, `AvisosSistema` | `AuthApi` | — |
| `inicio` | `TarjetaPelicula` | `CarteleraStore` | — | — |
| `proximamente` | `TarjetaPelicula`, `BotonAlerta` | `CarteleraStore` | — | `fechas` |
| `pelicula-detalle` | `SelectorFecha`, `BotonAlerta`, `Estrellas`, `*appRestrictEdad`, `duracion`, `precioArs` | `SesionStore` | `PeliculaApi`, `FuncionApi`, `ConfiguracionApi`, `ResenaApi` | `edad`, `fechas`, `resena` |
| `login`, `registro`, `restablecer-clave` | `VerClave` | `SesionStore` | `AuthApi` | — (`login` usa `rutaSegura`) |
| `checkout` | `IndicadorPasos`, `ContadorRetencion` y los cinco pasos | `ButacasStore`, `CompraStore` | — | `fechas`, `sala` |
| `paso-funcion` | `SelectorFecha`, `SelectorHorario`, `*appRestrictEdad` | `CompraStore`, `SesionStore` | — | `edad`, `fechas` |
| `paso-butacas` | `MapaButacas`, `butacasContiguas`, `precioArs` | `ButacasStore`, `CompraStore` | — | — |
| `paso-candy` | `ResumenCompra`, `precioArs` | `ButacasStore`, `CompraStore` | — | `candy` |
| `paso-pago` | `ResumenCompra`, `VerClave`, `appMascara`, `precioArs` | `ButacasStore`, `CompraStore`, `SesionStore` | — | `edad`, `formato`, `puntos`, `tarjeta` |
| `paso-comprobante` | `VistaComprobante` | `CompraStore`, `SesionStore` | — | `formato` |
| `comprobantes` | `VistaComprobante` | `ComprobantesLocales` | — | `fechas` |
| `perfil` | `FormularioPerfil`, `edad`, `precioArs` | `SesionStore` | `PerfilApi`, `CuponApi`, `FidelizacionApi` | `perfil`, `puntos`, `fechas` |
| `detalle-compra` | `VistaComprobante` | `ComprobantesLocales` | `CompraApi`, `ConfiguracionApi` | `cancelacion`, `formato` |
| `mis-peliculas` | `Estrellas` | `SesionStore` | `ResenaApi` | `fechas` |
| `notificaciones` | — | `NotificacionesStore`, `AvisosSistema` | — | `fechas` |
| `validacion` | `EscanerQr`, `ResultadoValidacion` | — | `ValidacionApi` | `formato` |
| `programar-funciones` | `SelectorDias`, `SelectorFecha`, `SelectorHora` | — | `FuncionApi`, `PeliculaApi`, `SalaApi`, `ConfiguracionApi` | `asignacion`, `fechas`, `sala` |
| `pelicula-form` | `SelectorGeneros`, `appMascara` | — | `PeliculaApi`, `GeneroApi`, `ImagenApi` | `pelicula`, `fechas` |
| `reportes` | `GraficoBarras`, `appMascara`, `precioArs` | — | `ReporteApi` | `reportes`, `fechas` |
| Resto del admin | Su selector cuando lo tiene (`SelectorFormatos`, `SelectorProductos`) | — | La Api de su dominio | La regla de su dominio |

**Por qué el admin llama directo a la Api y el checkout pasa por un store:** el estado del checkout lo comparten seis componentes (contenedor y cinco pasos) y el mapa recibe cambios por Realtime, así que vive en `CompraStore` y `ButacasStore`. Un ABM del admin es una sola pantalla que carga, edita y guarda; un store no agregaría nada (igual que `TaskApi` en clase).

**Cómo se hablan los componentes entre sí:**

| Mecanismo | Ejemplo |
|---|---|
| `input` (padre → hijo) | `MapaButacas` recibe `butacas`; `TarjetaPelicula`, la `pelicula` y su `puesto` en el top; `VistaComprobante`, el `comprobante` |
| `output` (hijo → padre) | `MapaButacas` emite `alternada` con la butaca tocada; `EscanerQr` emite el código leído a `Validacion`; `MenuPrincipal` emite `salir` a la raíz |
| `model()` (ida y vuelta) | `SelectorGeneros`, `SelectorFormatos`, `SelectorProductos`, `Estrellas` y `VerClave` funcionan como un campo de formulario |
| Store compartido | Los cinco pasos del checkout no reciben `input`: leen y escriben `CompraStore` y `ButacasStore`; `BotonAlerta` de Próximamente y de la ficha comparten `AlertasStore` |
| Router | Parámetros (`:peliculaId`) que llegan como `input` por `withComponentInputBinding`, `volver` en la URL después del login, y el resolver de la ficha |
| La base | Dos pantallas en dos dispositivos se enteran de lo mismo por Realtime (el mapa, la bandeja) |

### 13.6 Trazabilidad: de cada requerimiento al archivo

#### Requerimientos funcionales, por módulo

| Módulo | RF | Pantallas | `shared/` | Base |
|---|---|---|---|---|
| Cuentas y acceso | RF-01, RF-02, RF-04, RF-06, RF-07, RF-86 | `login`, `registro`, `recuperar-clave`, `restablecer-clave`, `perfil`, `completar-perfil`, `cuentas-admin` | `auth-api`, `perfil-api`, `cuenta-interna-api`, `sesion-store`, `formulario-perfil`, `ver-clave`, pipe `edad`, `rules/edad`, `rules/perfil`, `utils/rutas`, los guards | `perfiles`, `datos_sensibles`, `crear_perfil`, `proteger_fecha_nacimiento`, `asignar_rol` |
| Catálogo y cartelera | RF-08 a RF-15, RF-68 | `inicio`, `proximamente`, `pelicula-detalle`, `peliculas-admin`, `pelicula-form`, `generos-admin`, `selector-generos` | `pelicula-api`, `genero-api`, `imagen-api`, `cartelera-store`, `tarjeta-pelicula`, pipe `duracion`, `rules/pelicula`, `peliculaResolver` | `peliculas`, `generos`, `peliculas_generos`, vista `peliculas_cartelera`, `top_peliculas`, bucket `posters` |
| Salas, precios y funciones | RF-12, RF-16 a RF-25, RF-39, RF-78 | `salas-admin`, `sala-butacas`, `selector-formatos`, `funciones-admin`, `funcion-form`, `programar-funciones`, `precios-admin` | `sala-api`, `funcion-api`, `configuracion-api`, `selector-dias`, `selector-fecha`, `selector-hora`, `rules/asignacion`, `rules/sala`, `rules/fechas` | `plantilla_butacas`, `salas`, `butacas`, `funciones` con exclusión, `crear_butacas_sala`, `preparar_funcion`, `proteger_funcion_vendida`, `asignar_sala`, `programar_funciones`, `modificar_funcion` |
| Mapa de butacas | RF-26 a RF-31, RF-38, RF-90 | `paso-butacas` | `ocupacion-api`, `butacas-store`, `mapa-butacas`, `contador-retencion`, `appHighlightButaca`, `butacasContiguas`, `rules/contiguidad`, `rules/precio` | `ocupaciones` (PK), `retener_butacas`, `liberar_butacas`, Realtime, job `limpiar-retenciones` |
| Compra, pago y comprobante | RF-05, RF-32 a RF-37, RF-51 a RF-54 | `checkout`, `paso-funcion`, `paso-pago`, `paso-comprobante`, `comprobantes`, `mis-compras`, `detalle-compra` | `compra-store`, `compra-api`, `pago-service`, `comprobantes-locales`, `indicador-pasos`, `resumen-compra`, `vista-comprobante`, `selector-fecha`, `selector-horario`, `*appRestrictEdad`, `appMascara`, `rules/tarjeta`, `rules/precio`, `utils/qr`, `utils/comprobante-pdf` | `confirmar_compra`, `simular_pago`, `validar_venta`, `generar_codigo`, `comprobante`, vista `mis_compras` |
| Candy | RF-46 a RF-50 | `paso-candy`, `categorias-admin`, `productos-admin`, `combos-admin`, `selector-productos` | `candy-api`, `rules/candy` | `categorias`, `productos`, `combos`, `combos_productos`, `compras_items`, bucket `productos` |
| Cupones | RF-03, RF-40 a RF-43 | `paso-pago`, `cupones-admin`, `precios-admin`, `perfil` | `cupon-api`, `rules/cupones` | `cupones` (`BIENVENIDA`), `motivo_cupon`, `cupones_sugeridos`, `validar_cupon` |
| Cancelación y crédito | RF-44, RF-73 a RF-76 | `detalle-compra`, `paso-pago`, `perfil` | `compra-api` (`cancelar`), `fidelizacion-api`, `rules/cancelacion` | `cancelar_compra`, `creditos_movimientos`, vista `saldos_credito` |
| Puntos y recompensas | RF-45, RF-64 a RF-67 | `perfil`, `paso-pago`, `recompensas-admin` | `fidelizacion-api`, `compra-store` (canjes), `rules/puntos`, `rules/precio` | `puntos_movimientos`, `recompensas`, vista `saldos_puntos` |
| Validación de QR | RF-55 a RF-60 | `validacion`, `escaner-qr`, `resultado-validacion` | `validacion-api`, `rules/formato` | `validar_qr`, `auditoria` |
| Reseñas y Mis películas | RF-15, RF-61 a RF-63, RF-72 | `mis-peliculas`, `pelicula-detalle` | `resena-api`, `estrellas`, `tarjeta-pelicula`, `rules/resena` | `resenas`, `vio_pelicula`, vista `mis_peliculas`, `completar_autor_resena` |
| Preventa y alertas | RF-68 a RF-71 | `proximamente`, `pelicula-detalle`, `pelicula-form` | `alerta-api`, `alertas-store`, `boton-alerta`, `rules/precio` (`enPreventa`), `rules/pelicula` (`aperturaPreventa`) | `alertas`, `en_preventa`, `precio_entrada`, `procesar_alertas` (job `alertas-venta`) |
| Avisos | RF-87 a RF-89 | `notificaciones`, la raíz | `notificacion-api`, `notificaciones-store`, `bandeja-notificaciones`, `avisos-sistema`, `push-api` | `notificaciones`, `procesar_recordatorios` (job `recordatorios`), `validar_qr`, `suscripciones_push`, triggers `notificaciones_push` y `notificaciones_validacion`, `enviar-push` |
| Panel, reportes y auditoría | RF-77, RF-79 a RF-85 | `menu-principal` (con `menu.model.ts`), `reportes`, `auditoria` | `reporte-api`, `auditoria-api`, `grafico-barras`, `rules/reportes`, `rules/auditoria`, `utils/reporte-pdf`, `utils/excel` | `reporte_facturacion`, `reporte_mas_vistas`, `reporte_productos`, `auditoria`, `auditar_cambio` y los nueve triggers `auditar_*` |

#### Reglas de negocio: dónde se garantizan y dónde se anticipan

La columna "Base" es la que manda; "Angular" solo avisa antes para que la persona no llegue a un error (sección 2.1).

| RN | Regla | Base (garantiza) | Angular (anticipa) | Prueba |
|---|---|---|---|---|
| RN-01 | 30 min de limpieza | `preparar_funcion`, `asignar_sala` | `rules/asignacion.ts` (`ocupacionHasta`) | `asignacion.spec.ts` |
| RN-02 | Sin funciones solapadas | Restricción de exclusión `funciones_sin_solapamiento` | — | — |
| RN-03 | Sala de menor número | `asignar_sala` | `asignarSalas` en la vista previa de `programar-funciones` | `asignacion.spec.ts` |
| RN-04 | Butaca única por función | Clave primaria `(funcion_id, butaca_id)` de `ocupaciones` | Estados del mapa en `ButacasStore` | — |
| RN-05 | Retención de 10 min | `retener_butacas` y el job `limpiar-retenciones` | `ContadorRetencion` | `formato.spec.ts` (cuenta regresiva) |
| RN-06 | Edad según clasificación | `validar_venta`, `confirmar_compra`, `proteger_fecha_nacimiento` | `*appRestrictEdad`, `rules/edad.ts`, declaración del anónimo en `paso-pago` | `edad.spec.ts` |
| RN-07 | Precio de la entrada | `precio_entrada` | `precioEntrada` en `rules/precio.ts` | `precio.spec.ts` |
| RN-08 | Ventana de preventa | `en_preventa` | `enPreventa` | `precio.spec.ts` |
| RN-09, RN-10 | Cupón de primera compra y por edad | `motivo_cupon` | `motivoRechazoCupon` en `rules/cupones.ts` | `cupones.spec.ts` |
| RN-11 | Un cupón; sugerir el mejor | `confirmar_compra`, `cupones_sugeridos` | `mejorCupon` | `cupones.spec.ts` |
| RN-12 | Orden del cálculo y puntos | `confirmar_compra` | `CompraStore.resumen`, `puntosGanados` | `precio.spec.ts`, `puntos.spec.ts` |
| RN-13 | Canje sin sumar puntos | `confirmar_compra` | Canjes en `CompraStore`, `entradasConCombos` | `precio.spec.ts` |
| RN-14 | Candy solo con entrada | `confirmar_compra` | El paso de candy va después del de butacas | — |
| RN-15 | Combo con una 2D estándar | `confirmar_compra` | `diferenciaCombo` | `precio.spec.ts` |
| RN-16, RN-17 | QR con dos partes y ventanas de validación | `validar_qr` | `ResultadoValidacion` | — |
| RN-18 | Condiciones de la cancelación | `cancelar_compra` | `motivoNoCancelable` en `rules/cancelacion.ts` | `cancelacion.spec.ts` |
| RN-19 | Puntos pendientes hasta la función | `puntos_movimientos.disponible_desde` | `saldoPuntos` | `puntos.spec.ts` |
| RN-20 | Top 3 | `top_peliculas` | `CarteleraStore` | — |
| RN-21, RN-22 | Reseña solo de quien la vio; Mis películas | RLS de `resenas` con `vio_pelicula`, vista `mis_peliculas` | El botón de reseñar vive solo en `mis-peliculas` | — |
| RN-23 | Facturación y más vistas | `reporte_facturacion`, `reporte_mas_vistas` | Totales en `rules/reportes.ts` | `reportes.spec.ts` |
| RN-24 | Auditoría desde la base | `auditar_cambio`, los triggers `auditar_*` y las RPC | `rules/auditoria.ts` solo la muestra | `auditoria.spec.ts` |
| RN-25 | Butacas contiguas | `butacas_contiguas` dentro de `confirmar_compra` | `butacasContiguas` y `rules/contiguidad.ts` | `contiguidad.spec.ts` |

#### Requerimientos no funcionales

| RNF | Qué pide | Dónde se cumple |
|---|---|---|
| RNF-01 | Angular 22 standalone | `main.ts`, `app.config.ts`; ningún `NgModule` |
| RNF-02 | Signals en stores | `shared/stores/` |
| RNF-03 | Componentes reutilizables | `shared/components/` y los tres `selector-*` del admin |
| RNF-04 | `@if`, `@for` y `@switch` | Todas las `.html`; no hay `*ngIf` ni `*ngFor` |
| RNF-05 | Signal Forms | Los 23 archivos con `form()` (sección 13.2) |
| RNF-06 | Carga diferida y guards por rol | `app.routes.ts`, `shared/guard/`, `ngsw-config.json` |
| RNF-07 | Capas separadas | `api/` → `stores/` → pantallas, con `rules/` aparte (sección 3) |
| RNF-08 | Tipado estricto | `tsconfig.json` y `shared/models/` |
| RNF-09 | Auth con email, Google, GitHub y anónima | `auth-api.ts`, `sesion-store.ts`, `butacas-store.ts` |
| RNF-10 | RLS y nada de `service_role` en el front | RLS en las 28 tablas, `02_permisos_anon.sql`, `environment*.ts` con la clave publicable, `cuenta-interna-api.ts` |
| RNF-11 | Reglas críticas en la base | RPC y triggers de `tp1-sistema-cine.sql` (tabla de RN de arriba) |
| RNF-12 | Realtime | `ocupacion-api` + `butacas-store`, `notificacion-api` + `notificaciones-store` |
| RNF-13 | Storage, solo el admin escribe | `imagen-api.ts`, `utils/archivos.ts`, políticas de los buckets con `es_admin()` |
| RNF-14 | PWA instalable | `manifest.webmanifest`, `icons/`, `index.html`, `serviceWorker` en `angular.json` |
| RNF-15 | Caché, offline y aviso de versión | `ngsw-config.json`, `comprobantes-locales.ts`, `publico/comprobantes/`, `SwUpdate` en `app.ts` |
| RNF-16 | Push | `avisos-sistema.ts`, `push-api.ts`, `03_push.sql`, `enviar-push` |
| RNF-17 | Diseño visual propio | Tokens de `styles.css` y el `.css` de cada componente |
| RNF-18 | Responsive | `@media` en los `.css`; el panel lateral de `MenuPrincipal` |
| RNF-19 | Navegación fácil, fechas sin scroll | `MenuPrincipal`, `IndicadorPasos`, los selectores de chips, `appMascara` |
| RNF-20 | Accesibilidad AA | Foco visible y `.solo-lectores` en `styles.css`, `aria-*` de `appHighlightButaca`, la tabla oculta de `GraficoBarras`, contraste medido (sección 2.20) |
| RNF-21 | Español, pesos y hora de Buenos Aires | `main.ts`, `app.config.ts`, `rules/fechas.ts`, pipe `precioArs` |
| RNF-22 | Ley 25.326; tipo de sangre aparte | `privacidad/`, `datos_sensibles`, consentimiento en `FormularioPerfil` |
| RNF-23 | Vitest | Los 21 `.spec.ts`, `tsconfig.spec.json` |
| RNF-24 | Vercel y entornos | `environments/`, `fileReplacements` en `angular.json` |
| RNF-25 | GitHub y README | `README.md` |
| RNF-26 | Pipes `duracion`, `precioArs`, `edad` | `shared/pipes/` |
| RNF-27 | Directivas `appHighlightButaca`, `appRestrictEdad` | `shared/directives/` |
| RNF-28 | Validador `butacasContiguas` | `shared/validators/` |

#### Supuestos

| S | Supuesto | Dónde se aplica |
|---|---|---|
| S-01 | J y K con 14 lugares; 532 butacas | `plantilla_butacas` y `crear_butacas_sala`; se ve en `sala-butacas` y en el mapa |
| S-02 | Token aleatorio y validación por sector | `generar_codigo`, `validar_qr`, `ResultadoValidacion` |
| S-03 | Formatos por sala | `salas.formatos`, `selector-formatos`, `asignar_sala` |
| S-04 | El anónimo solo paga y declara la edad | Sesión anónima en `ButacasStore`, declaración en `paso-pago`, `comprobantes-locales` |
| S-05 | Leyenda con la clasificación | `leyendaClasificacion` en `rules/edad.ts`, `vista-comprobante`, `comprobante-pdf` |
| S-06 | Cupón con edad mínima | `cupones.edad_minima`, `cupones-admin`, `motivo_cupon` |
| S-07 | Puntos tras el cupón, pendientes y anulables | `confirmar_compra`, `cancelar_compra`, `rules/puntos.ts` |
| S-08 | Pasarela simulada e intercambiable | `simular_pago` en la base, `pago-service.ts` en Angular |
| S-09 | Preventa por fecha de compra | `en_preventa`, `enPreventa` |
| S-10 | Alertas por bandeja y push, una vez | `procesar_alertas`, `notificaciones`, `enviar-push` |
| S-11 | Candy solo con entrada | `confirmar_compra`, orden del checkout |
| S-12 | Combo con 2D estándar más la diferencia | `confirmar_compra`, `diferenciaCombo` |
| S-13 | Cuándo una película cuenta como vista | `vio_pelicula` |
| S-14 | Una reseña por película, solo de quien la vio | `unique` y RLS de `resenas` |
| S-15 | Top 3 de 30 días | `top_peliculas` |
| S-16 | Tipo de sangre opcional y aparte | `datos_sensibles`, `perfil-api.ts` |
| S-17 | Retención de 10 min con reemplazo y pg_cron | `retener_butacas` (`on conflict … where`), job `limpiar-retenciones` |
| S-18 | Horarios de validación | `validar_qr` |
| S-19 | Asignación por menor número | `asignar_sala`, `asignarSalas` |
| S-20 | Cancela el registrado, la compra entera, sin devolver el cupón | `cancelar_compra`, `motivoNoCancelable` |
| S-21 | Máximo de 10 butacas configurable | `configuracion.max_butacas_compra`, `retener_butacas` |
| S-22 | Precio base más recargo VIP fijo | `precio_entrada`, `precios-admin` |
| S-23 | Un cupón, combinable con crédito | `confirmar_compra`, `paso-pago` |
| S-24 | Facturación por compra y más vistas por función | `reporte_facturacion`, `reporte_mas_vistas` |

### 13.7 Preguntas sobre la estructura

1. **¿Por qué la carpeta se llama `shared/` y el resto por rol?**
   Lo compartido va en `shared/`, como en clase. Las pantallas se agrupan por quién las ve, que es también el guard que llevan: así la carpeta dice quién puede entrar.

2. **¿Por qué no hay un `*.routes.ts` por área ni un layout por rol?**
   Con 37 rutas, un solo `app.routes.ts` se lee entero de un vistazo, como en clase. El menú de cada rol lo arma `MenuPrincipal` con `computed` sobre el rol de `SesionStore`, así que no hace falta un componente de layout.

3. **¿Qué diferencia hay entre `rules/` y `utils/`?**
   `rules/` tiene las reglas del negocio (precio, edad, cupones) y repite lo que hace la base. `utils/` tiene ayudas técnicas (traducir errores, armar un PDF). Las dos son funciones, pero solo las de `rules/` son lógica del cine.

4. **¿Por qué hay reglas repetidas en `rules/` y en la base?**
   La base es la que garantiza; `rules/` anticipa para que la persona vea el motivo antes de tocar Pagar. Si alguien se saltea la pantalla, la base igual lo rechaza.

5. **¿Cómo demostrás que cubriste todos los requerimientos?**
   Con la matriz: el SPEC (sección 12) une cada RF, RN, RNF y S con su diseño y su fase, y esta sección los une con los archivos. Para las RN, además, está la prueba de la regla pura.

6. **¿Qué pasa si se cambia la pasarela de pago?**
   Se cambia `simular_pago` en la base y, si hiciera falta, `pago-service.ts`. Ninguna pantalla sabe cómo se cobra (S-08).

7. **¿Por qué `supabase/` no está en el repo?**
   Porque los scripts se aplican a mano en el SQL Editor y uno ya aplicado no se vuelve a correr. El README describe lo que tiene la base; los archivos quedan como respaldo local.

Frase para el oral: "Cada carpeta es una capa con una sola responsabilidad: las pantallas muestran, los stores guardan estado, las Apis hablan con Supabase, las reglas calculan y la base garantiza. Y cada archivo existe por un requerimiento que puedo señalar."

---

## 14. Guía para leer el código a ojo

La idea es no leer carpeta por carpeta, sino **de afuera hacia adentro y siguiendo flujos**: primero cómo arranca la app y qué rutas hay, después las piezas que usa todo el resto, y recién ahí un flujo completo de punta a punta (de la pantalla a la base). Los ABM del admin se parecen mucho entre sí, así que alcanza con leer uno bien.

### 14.1 Cómo leer cada archivo

- **Abrí el `.ts` y el `.html` de un componente lado a lado** (en VS Code, clic derecho en la pestaña → *Split Right*). El `.css` dejalo para el final o salteálo.
- **Leé primero los comentarios de una línea**: cada función y cada paso tiene uno que dice qué hace. Con eso solo ya entendés el archivo; el código lo confirma.
- **Movete con el editor, no con el explorador:** `Ctrl+P` abre un archivo por nombre, `F12` va a la definición de lo que tenés bajo el cursor y `Shift+F12` muestra quién lo usa. Así seguís un flujo sin perderte.
- **En cada archivo contestate tres preguntas:** ¿qué recibe? ¿qué devuelve o muestra? ¿a quién llama?
- **Tené la app abierta** (`npm start` o Vercel) y tocá en pantalla lo que estás leyendo.

### 14.2 El recorrido, por etapas

#### Etapa 1 · Cómo arranca y qué pantallas hay

| Orden | Archivo | Qué mirar |
|---|---|---|
| 1 | `src/main.ts` | `bootstrapApplication`: acá empieza todo. Sin `NgModule` |
| 2 | `src/app/app.config.ts` | Los providers globales: router, `LOCALE_ID` es-AR, zona `-0300` y el service worker |
| 3 | `src/app/app.routes.ts` | **El mapa de la app.** Cada ruta con su pantalla, `loadComponent` (carga diferida) y el guard que la protege. Leelo entero: después vas a saber dónde está cada cosa |
| 4 | `src/app/app.ts` y `app.html` | El esqueleto que se ve siempre: menú, bandeja, `<router-outlet />` y los avisos de perfil incompleto y de versión nueva |
| 5 | `shared/models/menu.model.ts` y `shared/components/menu-principal/` | Cómo cambia el menú según el rol: `computed` sobre `SesionStore.rol()` |

Al terminar tenés que poder decir: "la app tiene cuatro áreas (público, cliente, empleado y admin), cada pantalla baja cuando se la pide y un guard decide quién entra".

#### Etapa 2 · Los cimientos de `shared/`

| Orden | Archivo | Qué mirar |
|---|---|---|
| 6 | `src/environments/environment.ts` | Solo la URL y claves públicas |
| 7 | `shared/services/supabase.client.ts` | Cuatro líneas: el único cliente de Supabase de toda la app |
| 8 | `shared/models/pelicula.model.ts`, `funcion.model.ts`, `butaca.model.ts` | Cómo se tipan las entidades: interfaces con los nombres de columna y uniones de literales (`'ATP' \| '+13' \| '+18'`) |
| 9 | `shared/api/genero-api.ts` y después `pelicula-api.ts` | El patrón de toda Api: llama a Supabase, si hay error lo lanza, si no devuelve datos. No guarda nada |
| 10 | `shared/rules/formato.ts` + `formato.spec.ts` | Una regla pura y su prueba. Fijate que no importa nada de Angular |
| 11 | `shared/pipes/duracion-pipe.ts` | El pipe solo llama a `formatearDuracion` de la regla: la lógica está en `rules/` |

Con esto ya viste las cuatro capas de la sección 3: modelo, Api, regla y la pieza de Angular que la usa.

#### Etapa 3 · Sesión y guards

| Orden | Archivo | Qué mirar |
|---|---|---|
| 12 | `shared/api/auth-api.ts` | Registro, login, OAuth, sesión anónima y salida: todo `supabase.auth` está acá |
| 13 | `shared/stores/sesion-store.ts` | **El primer store.** Signals privados (`_sesion`, `_perfil`), sus versiones de solo lectura, los `computed` (`rol`, `edad`, `perfilCompleto`) y `listo()` |
| 14 | `shared/guard/cliente-guard.ts` y `admin-guard.ts` | 13 líneas cada uno: esperan `listo()`, miran el rol y dejan pasar o redirigen |
| 15 | `invitado-guard.ts`, `puede-comprar-guard.ts`, `pending-changes-guard.ts` | Los otros tres casos: quien ya entró, quien puede comprar y salir con cambios sin guardar |
| 16 | `publico/login/` (`.ts` + `.html`) | Un formulario simple con Signal Forms: `form()`, los validadores y el `submit` que llama a la Api |

#### Etapa 4 · Un flujo simple de punta a punta: la cartelera

| Orden | Archivo | Qué mirar |
|---|---|---|
| 17 | `shared/stores/cartelera-store.ts` | `cargar()` trae una vez y los filtros son `computed` (`filtradas`, `top`): se recalculan solos al cambiar un filtro |
| 18 | `shared/rules/pelicula.ts` | El filtro de la cartelera (sin tildes ni mayúsculas) |
| 19 | `publico/inicio/` | La pantalla solo lee el store y dibuja con `@for` |
| 20 | `shared/components/tarjeta-pelicula/` | Un componente reutilizable con `input()` |
| 21 | `shared/guard/pelicula-resolver.ts` → `publico/pelicula-detalle/` | La ficha nace con su película; el botón Comprar con `*appRestrictEdad` (`shared/directives/restrict-edad.ts`) |

Es el mismo patrón que en clase: store con signals, `computed` para filtrar y una pantalla que solo muestra.

#### Etapa 5 · El corazón: la compra

Es la parte más importante y la más preguntada. Seguila **en el orden en que la vive el usuario**:

| Orden | Archivo | Qué mirar |
|---|---|---|
| 22 | `publico/compra/checkout/` | El contenedor: un `@switch` sobre el paso del store. `ngOnDestroy` llama a `reiniciar()` de los dos stores |
| 23 | `publico/compra/paso-funcion/` | Elegir día y horario con `SelectorFecha` y `SelectorHorario` |
| 24 | `shared/api/ocupacion-api.ts` | `retener`, `liberar` y `suscribir` (el canal Realtime del mapa) |
| 25 | `shared/stores/butacas-store.ts` | **El archivo más interesante.** En `cargar()`: primero escucha, después lee (los "paso 1, 2, 3" comentados). En `alternar()`: marca la butaca enseguida, la retiene en la base y vuelve atrás si falla. `aplicarCambio()` recibe lo que llega por Realtime |
| 26 | `shared/components/mapa-butacas/` + `shared/directives/highlight-butaca.ts` | El mapa recibe `butacas` y emite `alternada`; la directiva pone clases y `aria-*` a cada butaca |
| 27 | `publico/compra/paso-butacas/` + `shared/validators/butacas-contiguas.ts` + `shared/rules/contiguidad.ts` | El validador propio dentro de Signal Forms, con la lógica en la regla pura |
| 28 | `publico/compra/paso-candy/` + `shared/rules/candy.ts` | Combos y productos con − y + |
| 29 | `shared/rules/precio.ts` + `precio.spec.ts` | Cómo se calcula cada entrada, la preventa y la diferencia del combo |
| 30 | `shared/stores/compra-store.ts` | **El store más grande (505 líneas).** No lo leas de corrido: mirá los `computed` (`entradas`, `resumen`, `subtotal`, `cuponSugerido`, `creditoDisponible`) y después `confirmar()` |
| 31 | `shared/services/pago-service.ts` | Llama a la RPC con la tarjeta o sin ella; la pasarela está en un solo lugar |
| 32 | `publico/compra/paso-pago/` | Cupón, crédito, canjes, declaración de edad y tarjeta |
| 33 | `publico/compra/paso-comprobante/` + `shared/components/vista-comprobante/` + `shared/utils/qr.ts` | QR, código y PDF; las librerías con `import()` |

Cerrá la etapa leyendo **lo que pasa del otro lado**, en `supabase/sql/tp1-sistema-cine.sql`:

- `retener_butacas` (línea 880): el `on conflict … do update … where` que reemplaza retenciones vencidas.
- `confirmar_compra` (línea 941): la transacción entera, de punta a punta. Es larga; leé los comentarios de cada paso.

Si entendés esta etapa, entendés las tres ideas fuertes del TP: Realtime, reglas anticipadas en Angular y garantizadas en la base, y stores compartidos entre pantallas.

#### Etapa 6 · La base de datos

`supabase/sql/tp1-sistema-cine.sql` está ordenado por bloques. Leelo en este orden, mirando los comentarios de una línea de cada objeto:

| Líneas (aprox.) | Bloque | Qué mirar |
|---|---|---|
| 1 a 325 | Tablas | Las columnas, los `check` y las claves. En `funciones` (línea 124), la restricción de exclusión; en `ocupaciones` (línea 233), la clave primaria `(funcion_id, butaca_id)` |
| 328 a 565 | Funciones auxiliares | `rol_actual`, `es_admin`, `edad`, `precio_entrada`, `en_preventa`, `validar_venta`, `motivo_cupon`: las reglas que después usan las RPC |
| 566 a 736 | Triggers | Perfil nuevo, fecha de nacimiento fija, auditoría, butacas de la sala nueva, fin de la función |
| 737 a 1550 | RPC | Programación, butacas, compra, validación, cancelación, cupones, comprobante y reportes |
| 1551 a 1623 | Tareas de pg_cron | `procesar_recordatorios` y `procesar_alertas` |
| 1624 a 1702 | Vistas | `peliculas_cartelera`, `saldos_puntos`, `saldos_credito`, `mis_compras` y `mis_peliculas`, todas `security_invoker` |
| 1703 al final | Seguridad y servicios | RLS en cada tabla, las políticas (desde la línea 1731), los grants, Storage, Realtime (línea 1942) y los tres jobs (línea 1945) |

Después los scripts chicos: `01_base.sql`, `02_permisos_anon.sql`, `03_push.sql` y `04_quita_demo_avisos.sql`.

#### Etapa 7 · El empleado: validar el QR

| Orden | Archivo | Qué mirar |
|---|---|---|
| 34 | `empleado/validacion/` | Modo acceso o candy, y el formulario del código manual |
| 35 | `empleado/escaner-qr/` | Emite el código leído con un `output` |
| 36 | `empleado/resultado-validacion/` | Un `@switch` por cada resultado posible |
| 37 | `validar_qr` en el SQL (línea 1206) | El QR con dos partes y las ventanas de horario |

#### Etapa 8 · El cliente

| Orden | Archivo | Qué mirar |
|---|---|---|
| 38 | `cliente/perfil/` + `shared/components/formulario-perfil/` | Un formulario compartido entre Perfil y Completar perfil; cupones, crédito y puntos |
| 39 | `cliente/mis-compras/` y `cliente/detalle-compra/` | El estado en un chip con `@switch` y la cancelación |
| 40 | `shared/rules/cancelacion.ts` → `cancelar_compra` en el SQL (línea 1300) | La misma regla dos veces: en Angular para avisar y en la base para garantizar |
| 41 | `cliente/mis-peliculas/` + `shared/components/estrellas/` | Un control propio con `model()` usado como campo de formulario |

#### Etapa 9 · El admin (leé uno bien y hojeá el resto)

| Orden | Archivo | Qué mirar |
|---|---|---|
| 42 | `admin/generos-admin/` | **El ABM más simple:** listar, crear, editar y borrar con su Api. Todos los demás siguen este molde |
| 43 | `admin/pelicula-form/` + `admin/selector-generos/` + `shared/api/imagen-api.ts` | Un formulario grande con Signal Forms, un control propio con `model()`, la subida del póster y la preventa |
| 44 | `admin/programar-funciones/` + `shared/rules/asignacion.ts` | La vista previa de la asignación de salas (y `asignar_sala` en la línea 764 del SQL) |
| 45 | `admin/reportes/` + `shared/components/grafico-barras/` | Las tres RPC de reporte, el gráfico y la exportación |
| 46 | `admin/auditoria/` + `shared/rules/auditoria.ts` | El log con filtros y "Ver datos" |

El resto del admin (salas, precios, candy, cupones, recompensas, cuentas) repite el molde de `generos-admin`: hojealo buscando solo lo distinto.

#### Etapa 10 · Avisos y PWA

| Orden | Archivo | Qué mirar |
|---|---|---|
| 47 | `shared/stores/notificaciones-store.ts` | La bandeja: escucha antes de leer, igual que el mapa |
| 48 | `shared/services/avisos-sistema.ts` | Permiso, aviso con la app abierta y suscripción al push con `SwPush` |
| 49 | `shared/api/push-api.ts` → `supabase/sql/03_push.sql` | Cómo se guarda el dispositivo y el trigger que dispara el push |
| 50 | `supabase/functions/enviar-push/index.ts` | 61 líneas: recibe la notificación y la manda a cada dispositivo |
| 51 | `ngsw-config.json` y `public/manifest.webmanifest` | Qué se cachea y cómo se instala |

### 14.3 Lo que podés dejar para el final o saltear

- Los `.css`: son estilos, no lógica. Si querés ver la identidad, alcanza con los tokens del principio de `src/styles.css`.
- `shared/utils/comprobante-pdf.ts`, `reporte-pdf.ts` y `excel.ts`: dibujan el PDF y el Excel línea por línea.
- `shared/rules/mascara.ts` y `directives/mascara-entrada.ts`: solo ponen las barras y los dos puntos mientras se escribe.
- `angular.json`, `tsconfig*.json`, `.vscode/`, `.editorconfig` y `.prettierrc`: configuración.
- Los ABM del admin después del primero.

### 14.4 Si tenés solo una hora

Estos diez archivos, en este orden, te dan el TP entero:

1. `app.routes.ts`: las pantallas y quién entra.
2. `shared/stores/sesion-store.ts` y `shared/guard/cliente-guard.ts`: cómo se sabe quién es el usuario y cómo se protege una ruta.
3. `shared/api/ocupacion-api.ts`: una Api con Realtime.
4. `shared/stores/butacas-store.ts`: el mapa en tiempo real.
5. `shared/validators/butacas-contiguas.ts` y `shared/rules/contiguidad.ts`: el validador que pidió la cátedra.
6. `shared/rules/precio.ts` con su `.spec.ts`: una regla pura y su prueba.
7. `shared/stores/compra-store.ts`, solo los `computed` y `confirmar()`.
8. `publico/compra/checkout/`: el `@switch` del flujo.
9. `confirmar_compra` en `tp1-sistema-cine.sql`: lo que garantiza la base.
10. `admin/generos-admin/`: el molde de todo ABM.

### 14.5 Cómo saber que ya lo entendiste

Probá contestar sin mirar:

- Si toco una butaca, ¿qué archivos se ejecutan, en qué orden, y cómo se entera otro usuario que la está mirando?
- ¿Dónde se calcula el precio de una entrada y por qué está en dos lugares?
- ¿Qué pasa si un cliente de 16 años abre la URL de compra de una película +18 a mano?
- ¿Por qué el checkout usa stores y `generos-admin` llama directo a su Api?
- ¿Qué archivo tendrías que cambiar para que la retención dure 15 minutos en vez de 10? (Ninguno: es `configuracion.minutos_retencion` en la base.)

Si las cinco te salen, ya podés explicar cualquier otra parte siguiendo el mismo patrón.

---

## 15. Decisiones por tema: lo de la cátedra y lo de afuera

La sección 2 explica las decisiones grandes una por una, y la 12 dice de qué lámina sale cada técnica. Esta sección junta las dos cosas y las ordena **por tema**, en dos partes:

- **Parte A:** temas que se vieron en clase o que están en el contenido de la cátedra. Acá la defensa es "lo hice como en clase", y lo interesante es dónde se aplicó.
- **Parte B:** temas y herramientas que **no** estaban en el contenido de la cátedra. Acá la defensa es "por qué hizo falta": cada uno lo pide un requerimiento, lo pidió la cátedra después o resuelve algo que lo de clase no podía resolver.

El criterio que ordena todo: **en Angular, todo como en clase; lo que no se vio vive en la base o lo pidió la cátedra.** Si hacía falta algo nuevo, se frenaba y se elegía entre la alternativa de clase y la nueva.

Cada fila sigue el mismo molde: **el problema, la decisión, el porqué, lo que se descartó y dónde está explicado en detalle.**

### Parte A · Temas vistos en clase

#### A1. Arranque, configuración y estructura

| Problema | Decisión | Por qué | Descartado | Ver |
|---|---|---|---|---|
| Cómo arranca la app | Standalone, con `bootstrapApplication(App, appConfig)` y sin `NgModule` (2·8) | Es como se arma una app en Angular moderno y lo pide RNF-01 | `NgModule` | 4.2, 12.1 |
| Que el compilador encuentre errores antes de correr | `strict` y `strictTemplates` (2·15) | El proyecto se generó sin ellos y la cátedra los pide. Con `strictTemplates`, un campo mal escrito en el HTML no compila | Dejarlos apagados | 4.1 |
| Dónde va cada cosa | `shared/` con subcarpetas en inglés (8·3) y las pantallas agrupadas por quién las ve | Son los nombres de la lámina, y la carpeta dice qué guard lleva cada pantalla | `compartido/` en español | 13.4 |
| Configuración por entorno | `environment.ts` y `environment.prod.ts` con `fileReplacements` (6·20) | La URL y las claves públicas cambian entre local y producción sin tocar código | `environment.development.ts` del CLI | 12.2 |
| Despliegue | Vercel con el preset de Angular, sin `vercel.json` (Clase 6, errores típicos) | El preset ya manda cualquier ruta a `index.html`, que es lo que resuelve el refresco en una ruta interna | Un `vercel.json` con rewrites | 4.1 |

#### A2. Componentes y cómo se comunican

| Problema | Decisión | Por qué | Descartado | Ver |
|---|---|---|---|---|
| Piezas que se repiten en varias pantallas | Componentes en `shared/components/` con `input()` y `output()` (2·19-20) | RNF-03. El mapa recibe las butacas y emite la tocada; no sabe de Supabase | — | 13.5 |
| Controles propios que funcionen como un campo de formulario | `model()`, como el `PriorityPicker` (4·10-13): selectores de géneros, formatos, productos, días, hora y fecha; `Estrellas`; `VerClave` | El valor va y vuelve entre padre e hijo con `[(valor)]`, y Signal Forms lo trata como un campo más | — | 12.1 |
| Un menú que no entraba con las doce pantallas del admin | `MenuPrincipal`: los grupos son datos en `menu.model.ts` y el componente elige con `computed` sobre el rol | Cada rol ve lo suyo sin HTML repetido; el menú emite `salir` y no sabe de Supabase | Menú plano en `app.html` y `@angular/animations` | 2.24 |
| El mismo formulario de perfil en dos pantallas | `FormularioPerfil` con `model()`, usado por `Perfil` y `CompletarPerfil` | Las cuentas de email, GitHub y Google completan el perfil en el mismo paso | Registro en una sola pantalla (obligaba a pasar el formulario entre componentes con técnicas no vistas) | 2.21 |

#### A3. Signals, stores y acceso a datos

| Problema | Decisión | Por qué | Descartado | Ver |
|---|---|---|---|---|
| Estado que comparten varias pantallas | Stores con `signal` y `computed`, en `root` (2·21, 2·27, 5·5) | RNF-02. Los filtros de la cartelera y el resumen de la compra se recalculan solos, como el filtro de `TaskList` | — | 2.10 |
| Reaccionar a un cambio (entra un usuario, cambia la selección) | `effect` (4·25) | Carga el perfil, abre la bandeja y copia la selección de butacas al formulario | `linkedSignal` (no se vio) | 12.2 |
| Que la compra siguiente no arrastre la anterior | `reiniciar()` en `CompraStore` y `ButacasStore`, llamado en el `ngOnDestroy` del checkout, con `removeChannel` (6·13) | La limpieza queda en un método que se puede leer: libera butacas, cierra el canal y vacía | Stores provistos en el componente; `DestroyRef` | 2.10 |
| Separar el acceso a datos | Una Api por dominio que llama, lanza el error y devuelve datos, como `TaskApi` | RNF-07: la pantalla no sabe de Supabase y el store no sabe de HTML | Llamar a Supabase desde los componentes | 3 |
| Tipar lo que devuelve la base | Modelos con los nombres de columna (`fecha_estreno`) | `data as PeliculaModel`, como en clase, es cierto sin una capa de mapeo | Traducir a camelCase | 2.19 |
| Un solo lugar para la pasarela de pago | `PagoService` común en `root` (5·5) | Para cambiar la pasarela se toca un archivo | Clase abstracta con `useClass` | 2.8 |

#### A4. Plantillas y control de flujo

| Problema | Decisión | Por qué | Descartado | Ver |
|---|---|---|---|---|
| Mostrar una cosa u otra según un estado | `@if`, `@for` con `track` y `@switch` (2·26, 4·18) | RNF-04. El checkout es un `@switch` sobre el paso; el resultado de validación, otro sobre el resultado | `*ngIf` y `*ngFor` (no hay ninguno) | 13.2 |

#### A5. Formularios y validación

| Problema | Decisión | Por qué | Descartado | Ver |
|---|---|---|---|---|
| Todos los formularios | Signal Forms: `form()`, `[formField]` y `submission` (3·18-19, 4·20-25) | RNF-05. El modelo del formulario es un signal: se lee y se escribe como cualquier otro | — | 12.1 |
| Butacas que tienen que estar juntas | Validador propio `butacasContiguas` sobre `validate()` (3·19), con la lógica en `rules/contiguidad.ts` | RNF-28 y RN-25. El mensaje aparece antes de pagar; la base lo vuelve a controlar | — | 2.12 |
| Marcar todos los campos al enviar | Nada: en Angular 22.2 `submit` ya lo hace | La lámina 4·24 usa `onInvalid: 'touch'`, que en 22.2 no compila así | — | 12.3 |
| Entrar como admin sin tipear (pedido de la cátedra) | `borrador.set(...)` sobre el modelo del `form()` | Es escribir en el signal que Signal Forms ya vigila: no suma ninguna técnica | — | 2.22 |

#### A6. Rutas, carga diferida y guards

| Problema | Decisión | Por qué | Descartado | Ver |
|---|---|---|---|---|
| Que cada usuario baje solo lo que usa | `loadComponent` en cada ruta (3·5-6) | RNF-06: la app arranca más rápido | `loadChildren` con un `*.routes.ts` por área | 2.11 |
| Que nadie entre a un área que no es suya | Un guard funcional por rol en `canMatch` (6·17, 7·5-7) | Con `canMatch` el chunk ni se descarga. Cada guard se lee como el `adminGuard` de la lámina | Un guard fábrica `rolGuard(roles)` | 2.11 |
| Un admin recién logueado terminaba en la portada | Los guards esperan `sesion.listo()`, que espera el perfil (que trae el rol) | Justo después de entrar, la sesión ya cambió pero el rol todavía no llegó | Esperar solo la sesión | 2.21 |
| No perder un formulario ni dejar butacas retenidas | `pendingChangesGuard` con `canDeactivate` y `confirm()` (7·6, 8·6) | Pregunta antes de salir | — | 12.2 |
| Que la ficha no aparezca vacía | `peliculaResolver` (7·7, 8·7) y la película como `input()` con `withComponentInputBinding` | La pantalla nace con su dato | — | 12.1 |
| ¿Los guards dan seguridad? | No: "el guard no es seguridad" (6·17). La dan el RLS y las RPC | El código de Angular llega al navegador y cualquiera lo lee | — | 2.11 |

#### A7. Pipes y directivas

| Problema | Decisión | Por qué | Descartado | Ver |
|---|---|---|---|---|
| Mostrar duración, pesos y edad | Pipes puros `duracion`, `precioArs` y `edad` (7·9-11) | RNF-26. El pipe solo llama a la regla pura, que se prueba con Vitest | — | 2.12 |
| Que el mapa no dependa solo del color | `appHighlightButaca`, de atributo con `host`, como `appHighlight` (7·14) | RNF-27 y RNF-20: clases y `aria-*` en cada butaca | — | 2.12 |
| Ocultar Comprar si la edad no alcanza | `*appRestrictEdad`, estructural como `*appIfOverdue` (7·15); el motivo lo muestra el componente con `@if` | La directiva hace una sola cosa | De atributo con `exportAs` | 2.12 |
| Escribir fechas, horas y tarjetas sin tipear las barras | `appMascara`, otra directiva de atributo, con la lógica en `rules/mascara.ts` | Misma técnica que `appHighlightButaca`, aplicada a un `input` | — | 4.4 |

#### A8. Supabase visto en clase

| Problema | Decisión | Por qué | Descartado | Ver |
|---|---|---|---|---|
| Conectarse a Supabase | Un solo cliente como constante de módulo (6·6) con la clave publicable en el front (5·19) | La clave es pública por diseño: lo que protege es el RLS | — | 4.4 |
| Login | Supabase Auth con GitHub como en clase; email con clave, Google y la recuperación usan la misma API `supabase.auth` | RF-01, RF-02 y RNF-09 piden los tres proveedores | Solo GitHub | 2.21, 2.25 |
| Quién ve qué fila | RLS con políticas "dueño de la fila" más un `for all` del admin, como la tabla `tasks` | RNF-10 | — | 2.5 |
| Valores cerrados en la base | `text` + `check (... in (...))` (5·10) | Igual que la tabla `tasks` | `enum` | 12.2 |
| Mapa y bandeja en vivo | Realtime con `channel().on('postgres_changes')`, filtrando en el callback (6·10) | RNF-12 y RF-28 | `filter` en la suscripción | 12.2 |
| Imágenes | Storage con la ruta `{id}/{nombre normalizado}`, como `attachments.ts` | RNF-13 | — | 4.4 |
| Armar la base | Scripts pegados en el SQL Editor (6·5) | Cada script entre `begin;` y `commit;`: todo o nada | CLI y migraciones | 2.18 |

#### A9. PWA

| Problema | Decisión | Por qué | Descartado | Ver |
|---|---|---|---|---|
| Que se pueda instalar | `ng add @angular/pwa`, manifest e íconos propios (7·17-18, 8·20-22) | RNF-14 | — | 4.1 |
| Avisar que hay una versión nueva | `SwUpdate` en `app.ts` con un signal y `@if` (8·23) | RNF-15; copia directa de la lámina | Un componente con `toSignal` | 12.2 |
| Que el visitante no descargue el admin | **Desvío a propósito:** `namedChunks` y las pantallas de las áreas en `installMode: lazy` | La lámina 8·21 precachea todo el JS y eso anulaba el ahorro de `canMatch` | Precachear todo, como la lámina | 12.3 |

#### A10. Pruebas

| Problema | Decisión | Por qué | Descartado | Ver |
|---|---|---|---|---|
| Qué probar | Solo funciones puras de `rules/` y `utils/` con Vitest (21 archivos `.spec.ts`) | RNF-23 pide precio, cupones, edad, puntos y asignación. Se prueba la lógica, sin TestBed ni red | Los specs "should create" de taskflow | 12.3 |

### Parte B · Temas y herramientas fuera del contenido de la cátedra

Para cada uno, la primera pregunta del profe va a ser "¿por qué usaste algo que no vimos?". La columna "Quién lo pide" es la respuesta corta.

#### B1. Lógica en la base de datos (Postgres)

| Problema | Decisión | Quién lo pide | Por qué | Descartado | Ver |
|---|---|---|---|---|---|
| Que dos personas no compren la misma butaca | Retenciones y ventas en `ocupaciones`, con clave primaria `(funcion_id, butaca_id)` | RN-04 | Postgres no deja dos filas con la misma clave: uno gana y el otro choca | Dos tablas (dos canales de Realtime, `entradas` pública) | 2.2 |
| Que dos funciones no se pisen en la misma sala | Restricción de exclusión con `btree_gist` sobre un `tstzrange` `[)` | RN-02 | Es un `unique` para rangos de tiempo; un trigger fallaría con dos admins a la vez | Trigger o validación en la RPC | 2.3 |
| Vender, validar, cancelar y programar sin estados a medias | RPC `security definer` con `search_path` vacío, `for update` y `on conflict` | RNF-11 y la sección 5 del documento | Todo pasa en una transacción y la función valida todo antes de escribir | Escribir desde Angular con RLS (no puede validar entre tablas) | 2.4 |
| Pagar y guardar a la vez | El pago simulado (`simular_pago`) dentro de `confirmar_compra` | S-08 | No puede pasar "pagó pero no se guardó"; la tarjeta no se guarda en ningún lado | Pagar desde Angular y después confirmar | 2.8 |
| Un QR que no se pueda adivinar ni lleve datos | Código de 10 caracteres con `gen_random_bytes` | RF-54, S-02 | Unos 50 bits de azar; la base sabe a qué compra corresponde | Un JWT firmado | 2.9 |
| Que el cliente edite su perfil pero no su rol | `grant update` por columna | Decisión de diseño | RLS filtra filas; el grant, columnas | — | 2.5 |
| Que una vista no se saltee el RLS | Vistas con `security_invoker = true` | RNF-10 | Sin eso, la vista corre con los permisos de su dueño | — | 2.5 |
| Tipo de sangre que ni el admin vea | Tabla aparte `datos_sensibles`, solo el titular | RNF-22, S-16, Ley 25.326 | RLS no oculta columnas | Una columna en `perfiles` | 2.6 |
| Quitarle al visitante sin sesión las RPC que no le corresponden | `revoke execute` en `02_permisos_anon.sql` | RNF-10 | Por defecto Postgres deja ejecutar a todos | — | 4.7 |

#### B2. Triggers y tareas programadas

| Problema | Decisión | Quién lo pide | Por qué | Descartado | Ver |
|---|---|---|---|---|---|
| Registrar todo cambio, venga de donde venga | Trigger genérico `auditar_cambio` con `to_jsonb(old)` y `to_jsonb(new)` | RN-24 | Hasta un cambio hecho desde el panel queda anotado | Auditar desde Angular | 2.13 |
| Cosas que tienen que pasar siempre al insertar | Triggers: perfil nuevo, fecha de nacimiento fija, butacas de la sala nueva, fin de la función, autor de la reseña | RNF-11; varios por decisión | No dependen de que la pantalla se acuerde | — | 12.4 |
| Cosas que dispara el paso del tiempo | pg_cron: limpiar retenciones (1 min), recordatorios (5 min) y alertas (10 min) | S-17 | Un trigger no se dispara por el reloj | — | 2.14 |
| Puntos que se liberan cuando empieza la función | Sin job: `disponible_desde` y comparar con `now()` | S-07 | Donde alcanza con comparar fechas, no hace falta cron | Un job que mueva saldos | 2.14 |

#### B3. Supabase más allá de lo visto

| Problema | Decisión | Quién lo pide | Por qué | Descartado | Ver |
|---|---|---|---|---|---|
| Comprar sin cuenta | Sesión anónima con `signInAnonymously()` al tocar la primera butaca | RNF-09, S-04 | La retención necesita un dueño y el anónimo no deja datos | — | 2.7 |
| Que el admin cree cuentas sin perder su sesión | Un segundo cliente de Supabase que no guarda sesión, más la RPC `asignar_rol` | RF-06 (D-23) | Sin `service_role` en ningún lado | Una Edge Function con `service_role` | 2.16 |
| Avisos con la app cerrada | Edge Function `enviar-push`, pg_net, Vault, `web-push`, VAPID y `SwPush` | La cátedra, el 09/10 (D-25) | La base no puede cifrar ni firmar un push, y la clave privada no puede ir en Angular | Vercel Functions, OneSignal, Firebase | 2.23 y 16 |

#### B4. Navegador y PWA

| Problema | Decisión | Quién lo pide | Por qué | Descartado | Ver |
|---|---|---|---|---|---|
| Ver el comprobante sin conexión y sin cuenta | Los últimos 20 en `localStorage` | RNF-15, S-04 | El anónimo no tiene dónde guardarlos en la base | — | 4.4 |
| Avisar con la app abierta | `showNotification` del service worker | RNF-16 | En Chrome para Android `new Notification()` no está permitido | `new Notification()` | 2.15 |
| Desplegables con el mouse en la computadora y no en el celular | `window.matchMedia` | Pedido del 09/10 (barra) | Es la API del navegador para saber si hay mouse y el ancho | — | 2.24 |

#### B5. Librerías

| Problema | Decisión | Quién lo pide | Por qué | Descartado | Ver |
|---|---|---|---|---|---|
| Generar el QR sin red | `qrcode` | RF-53 | Funciona en modo avión; se carga con `import()` | — | 2.17 |
| Leer el QR con la cámara | `@zxing/browser` | RF-55 | Lee desde la cámara del navegador, con HTTPS | — | 4.6 |
| PDF del comprobante y del reporte | `jspdf` | RF-51, RF-82 | Se arma en el teléfono | — | 2.17 |
| Exportar a Excel | `write-excel-file` | RF-82 | Un `.xlsx` real con una función | `xlsx` (versión de npm desactualizada y con vulnerabilidades) | 2.17 |
| Gráficos | Ninguna: barras con CSS y `@for` | RF-80, RF-81 | Dos gráficos de barras no justifican una librería | Una librería de gráficos | 2.17 |

#### B6. Formato argentino

| Problema | Decisión | Quién lo pide | Por qué | Ver |
|---|---|---|---|---|
| Fechas, pesos y hora de Buenos Aires | `registerLocaleData`, `LOCALE_ID` en es-AR y `DATE_PIPE_DEFAULT_OPTIONS` en `-0300` | RNF-21 | Sin esto, el `DatePipe` muestra el formato y la hora del navegador | 4.3 |

#### B7. Herramientas de trabajo

| Problema | Decisión | Por qué | Ver |
|---|---|---|---|
| Probar el SQL antes de aplicarlo | PGlite (Postgres en memoria) con imitaciones de `auth`, `storage` y `cron`: 151 controles | Un script mal aplicado no se puede editar; mejor encontrar el error antes | 2.18 |
| Ver si un push salió de verdad | La tabla `net._http_response` y los logs de la Edge Function | Así se encontró el error del secreto el 09/10 | 16.13 |

Frase para el oral: "En Angular hice todo como en clase. Lo que no se vio son cosas de la base o del push: o las pide el documento (las reglas críticas en la base, la sesión anónima, pg_cron), o las pidió la cátedra (el push con la app cerrada), o lo de clase no alcanzaba (cifrar y firmar un push)."

---

## 16. Push notifications paso a paso

### 16.1 Qué problema resuelve

Antes de F16 los avisos llegaban por dos canales: la **bandeja** (Realtime) y el **aviso del sistema** con la app abierta (sección 2.15). Los dos necesitan que la app esté corriendo. El 09/10 la cátedra pidió que lleguen **también con la app cerrada** (D-25). Eso es un push: un mensaje que un servidor le manda al navegador por medio del **servicio de push** del fabricante, y que el **service worker** muestra aunque la página no esté abierta.

Lo que hace difícil el push es que cada mensaje tiene que ir **cifrado** para ese dispositivo y **firmado** por el servidor que lo manda. Postgres no puede hacer ninguna de las dos cosas, y la clave que firma no puede estar en Angular, porque todo lo de Angular llega al navegador. Por eso aparece una pieza nueva: una **Edge Function**.

### 16.2 Las piezas

| Pieza | Dónde vive | Qué hace |
|---|---|---|
| `AvisosSistema` | `shared/services/avisos-sistema.ts` | Pide permiso, suscribe el dispositivo con `SwPush` y lo da de baja al salir |
| `PushApi` | `shared/api/push-api.ts` | Llama a las RPC que guardan y borran la suscripción |
| `suscripciones_push` | Base (`03_push.sql`) | Una fila por dispositivo suscripto: `usuario_id`, `endpoint`, `p256dh` y `auth` |
| `guardar_suscripcion_push` y `borrar_suscripcion_push` | Base | La única forma de escribir esa tabla |
| `notificaciones` | Base | Toda fila nueva es un aviso: de ahí salen los tres canales |
| Trigger `notificaciones_validacion` | Base, `before insert` | Le suma sector, fecha y hora al aviso de QR validado |
| Trigger `notificaciones_push` (función `enviar_push`) | Base, `after insert` | Junta las suscripciones del usuario, lee el secreto y llama a la Edge Function |
| pg_net | Extensión de Postgres | Hace el pedido HTTP desde la base, después del commit |
| Vault | Base | Guarda el secreto compartido `push_secreto`, cifrado |
| Edge Function `enviar-push` | Servidores de Supabase (Deno) | Verifica el secreto, cifra y firma con `web-push` y envía a cada dispositivo |
| Servicio de push | Google (FCM), Mozilla o Apple | Recibe el mensaje y se lo entrega al dispositivo cuando está en línea |
| Service worker de Angular (`ngsw-worker.js`) | El navegador del usuario | Recibe el push y muestra el aviso; al tocarlo, abre la app |

```text
 Celular del cliente                      Supabase                                   Afuera
┌──────────────────────┐   1 suscribir   ┌────────────────────────────────────┐
│ Activar avisos       │ ──────────────► │ guardar_suscripcion_push           │
│ AvisosSistema +      │   (RPC)         │   └─► suscripciones_push            │
│ SwPush               │                 │                                    │
│                      │                 │ 2 nace un aviso (cron, validar_qr) │
│                      │                 │   INSERT en notificaciones          │
│                      │                 │   ├─ before: completa el cuerpo     │
│                      │                 │   └─ after: enviar_push ────────┐   │
│                      │                 │        lee Vault, arma el JSON  │   │
│                      │                 │        pg_net.http_post         │   │
│                      │                 └─────────────────────────────────┼───┘
│                      │                                                   ▼ 3 (después del commit)
│                      │                 ┌────────────────────────────────────┐
│                      │                 │ Edge Function enviar-push          │
│                      │                 │   ¿x-push-secreto correcto?         │
│                      │                 │   cifra con p256dh/auth             │
│                      │                 │   firma con VAPID privada ──────────┼──► 4 servicio de push
│                      │                 └────────────────────────────────────┘     (FCM, Mozilla, Apple)
│ service worker       │ ◄───────────────────────────────────────────────────────── 5 entrega el mensaje
│ muestra el aviso     │
│ 6 tocar → abre la URL│
└──────────────────────┘
```

### 16.3 Las palabras que hay que saber

| Término | Qué es |
|---|---|
| Service worker | Un script que el navegador corre aparte de la página. Sigue vivo con la app cerrada y es el que recibe el push. Acá es el de Angular (`ngsw-worker.js`) |
| Suscripción push | Lo que entrega el navegador al aceptar: un **`endpoint`** (la dirección de ese dispositivo en el servicio de push) y dos claves, **`p256dh`** y **`auth`**, para cifrarle los mensajes |
| VAPID | Un par de claves que identifica al servidor que envía. La **pública** va en Angular y se usa al suscribir; la **privada** firma cada push y solo la tiene la Edge Function |
| Edge Function | Una función de TypeScript que Supabase corre en sus servidores con Deno. Se llama por HTTP |
| `web-push` | La librería que cifra el mensaje con las claves del dispositivo y lo firma con VAPID |
| pg_net | Extensión que permite hacer un pedido HTTP desde Postgres. Es asíncrona: encola el pedido y lo manda después del commit |
| Vault | El lugar cifrado de Supabase para secretos dentro de la base |
| `tag` | Etiqueta del aviso. Dos avisos con el mismo `tag` no se duplican: el segundo reemplaza al primero |
| TTL | Cuánto tiempo guarda el servicio de push el mensaje si el dispositivo está apagado (acá, 24 horas) |

### 16.4 Lo que se configuró una sola vez

1. **Claves VAPID:** se generó un par. La pública va en `environment.ts` y `environment.prod.ts` (`vapidPublicKey`); la privada nunca se escribió en el repo ni en la base.
2. **Edge Function:** se creó `enviar-push` desde el panel (Edge Functions → Deploy a new function), pegando `supabase/functions/enviar-push/index.ts`, igual que el SQL va por el SQL Editor: sin CLI.
3. **`verify_jwt` apagado:** por defecto Supabase exige un token de usuario para llamar a una función. Acá quien llama es la base, que no tiene sesión, así que se apagó y la protección es el secreto compartido.
4. **Cuatro secretos de la función:** `VAPID_PUBLIC_KEY`, `VAPID_PRIVATE_KEY`, `VAPID_SUBJECT` (un `mailto:`, porque Apple rechaza otras formas) y `PUSH_SECRETO`.
5. **El mismo secreto en Vault:** con una consulta aparte, que no está en ningún archivo: `select vault.create_secret('<valor>', 'push_secreto');`
6. **`03_push.sql`:** pg_net, la tabla `suscripciones_push` con RLS, las dos RPC y los dos triggers.
7. **Prueba aislada:** la función envió a un solo dispositivo y respondió 201.

### 16.5 Fase 1 · El cliente activa los avisos

1. En `/cuenta/notificaciones` el cliente toca **Activar avisos**. `Notificaciones` llama a `AvisosSistema.activar()`.
2. `activar()` pide el permiso con `Notification.requestPermission()`. Tiene que ser desde un clic: el navegador no lo pregunta sin un gesto del usuario.
3. Si el permiso es `granted`, `suscribir()` llama a `swPush.requestSubscription({ serverPublicKey: environment.vapidPublicKey })`. El navegador habla con su servicio de push y devuelve la suscripción: `endpoint`, `p256dh` y `auth`.
4. `PushApi.guardar()` manda esos tres datos a la RPC `guardar_suscripcion_push`.
5. La RPC exige un cliente registrado (`es_registrado()`, si no `AVISOS_SOLO_REGISTRADOS`) e inserta la fila con `auth.uid()`. Si el `endpoint` ya existía (el mismo navegador, otra cuenta), el `on conflict (endpoint) do update` la pasa al usuario actual.
6. **Las veces siguientes no hace falta tocar nada:** cuando entra un usuario, `NotificacionesStore` llama a `suscribir()`. Si el permiso ya estaba dado, vuelve a guardar la suscripción para esa cuenta sin preguntar.

### 16.6 Fase 2 · Nace un aviso

Nadie dispara un aviso a mano. Hay tres orígenes, y los tres terminan en un `INSERT` en `notificaciones`:

| Aviso | Quién lo inserta | Cuándo |
|---|---|---|
| Recordatorio de 24 h y de 2 h (RF-87, RF-88) | `procesar_recordatorios`, desde el job `recordatorios` | Cada 5 minutos revisa qué funciones están por empezar |
| QR validado (RF-89) | `validar_qr` | En el acto, cuando el empleado valida |
| Alerta de venta (RF-69) | `procesar_alertas`, desde el job `alertas-venta` | Cada 10 minutos, cuando una película con alerta tiene entradas a la venta |

En el momento del `INSERT` pasan dos triggers, en este orden:

1. **`before insert` → `notificaciones_validacion`:** solo para los avisos de QR validado. Cambia el cuerpo antes de guardarlo y le suma "· Acceso · 09/10 21:34 h". Se hizo con un trigger para no reescribir `validar_qr`, que ya estaba aplicada.
2. Se guarda la fila. Desde acá salen los canales 1 y 2: Realtime se la entrega a `NotificacionesStore`, que la pone en la bandeja y, si la app está abierta, muestra el aviso del sistema.
3. **`after insert` → `notificaciones_push`:** el canal 3, el push. Como corre después, ya ve el cuerpo completo.

### 16.7 Fase 3 · El trigger le pasa el aviso a la Edge Function

Función `enviar_push()` en `03_push.sql`:

1. Junta en un JSON todas las suscripciones del usuario de la notificación (`jsonb_agg` sobre `suscripciones_push`).
2. **Si no tiene ninguna, termina.** No hay pedido ni costo: ese usuario no activó avisos.
3. Lee `push_secreto` de `vault.decrypted_secrets`. Puede porque es `security definer`: corre con los permisos del dueño de la base. Si el secreto no está, termina.
4. Llama a `net.http_post` con:
   - la URL de la función (`…/functions/v1/enviar-push`);
   - el cuerpo: la notificación (`id`, `titulo`, `cuerpo`, `url`) y las suscripciones;
   - los encabezados `Content-Type` y **`x-push-secreto`** con el valor de Vault;
   - un tiempo máximo de 5 segundos.
5. **pg_net no espera la respuesta:** encola el pedido y lo manda un proceso aparte, después del commit. Si el push tarda o falla, la compra, la validación o el job siguen sin enterarse, y la bandeja ya tiene el aviso.

Nadie puede llamar a `enviar_push()` a mano: el script le quita el permiso de ejecutar a `public`, `anon` y `authenticated`.

### 16.8 Fase 4 · La Edge Function envía

Archivo `supabase/functions/enviar-push/index.ts`, de arriba hacia abajo:

1. Importa `web-push` desde npm (`npm:web-push@3.6.7`) y carga las tres claves VAPID con `setVapidDetails`.
2. `Deno.serve` atiende cada pedido. Lo primero que hace es comparar `x-push-secreto` con `PUSH_SECRETO`. **Si no coincide, responde 401** y no hace nada más. Así solo la base puede usarla.
3. Lee la notificación y las suscripciones del cuerpo. **No consulta la base** ni usa `service_role`: todo lo que necesita ya viene armado por el trigger.
4. Arma el mensaje en el formato que entiende el service worker de Angular: `notification` con `title`, `body`, `icon`, `tag` (`notificacion-{id}`) y `data.onActionClick` con la operación `navigateLastFocusedOrOpen` y la `url` del aviso.
5. Por cada suscripción llama a `webpush.sendNotification`, que cifra el mensaje con `p256dh` y `auth` de ese dispositivo, lo firma con la VAPID privada y lo manda al `endpoint`. Va con `TTL` de 24 horas y urgencia alta.
6. Usa `Promise.allSettled`: si un dispositivo falla, los demás igual reciben su push.
7. Devuelve el estado de cada envío (201 si el servicio de push lo aceptó; 404 o 410 si ese dispositivo ya no existe), que queda en los logs y en `net._http_response`.

### 16.9 Fase 5 · El aviso llega al dispositivo

1. El servicio de push (FCM en Android y Chrome, Mozilla en Firefox, Apple en iPhone) guarda el mensaje y se lo entrega al dispositivo cuando está en línea, hasta 24 horas después.
2. El navegador despierta al service worker de Angular, que muestra el aviso con el título, el cuerpo y el ícono. **La app puede estar cerrada.**
3. **Sin duplicados:** si la app estaba abierta, el canal 2 ya había mostrado el mismo aviso con el mismo `tag`. El push lo reemplaza en lugar de sumar otro.
4. Al tocar el aviso, `navigateLastFocusedOrOpen` enfoca la app si estaba abierta, o la abre, en la `url` del aviso: por ejemplo `/cuenta/compras/{id}`, con el QR.

### 16.10 Fase 6 · El cliente cierra sesión

1. `MenuPrincipal` emite `salir` y `app.ts` llama a `AvisosSistema.desuscribir()` **antes** del `signOut`.
2. `desuscribir()` busca la suscripción del navegador, llama a `borrar_suscripcion_push` (que borra solo si la fila es de ese usuario) y después la da de baja en el navegador.
3. Así el próximo que use ese equipo no recibe los avisos de la cuenta anterior. Si la baja falla, la salida sigue igual: el dispositivo queda suscripto y pasa a la próxima cuenta que active los avisos.

### 16.11 Seguridad: quién puede hacer qué

| Riesgo | Qué lo impide |
|---|---|
| Que alguien llame a la Edge Function y mande avisos falsos | El secreto en `x-push-secreto`: sin él responde 401. El secreto está en Vault y en los secretos de la función, no en el repo ni en Angular |
| Que alguien firme pushes a nombre de Cineverso | La VAPID privada solo existe como secreto de la función |
| Que un cliente lea o borre suscripciones de otro | RLS: solo lee las suyas. Nadie escribe la tabla directo; las RPC usan `auth.uid()` |
| Que un anónimo se suscriba | `guardar_suscripcion_push` exige un usuario registrado (los avisos son solo de clientes, D-08) |
| Que alguien dispare el trigger a mano | `revoke execute` de `enviar_push` y `completar_aviso_validacion` |
| Que la función lea datos de la base | No tiene cómo: no usa `service_role` ni consulta tablas |

### 16.12 Qué pasa si algo falla

| Falla | Qué pasa | El aviso igual llega por… |
|---|---|---|
| El cliente no dio permiso | No hay suscripción y el trigger termina en el paso 2 | La bandeja |
| La Edge Function no responde | pg_net corta a los 5 segundos; la transacción original no se entera | La bandeja |
| El secreto no coincide (lo que pasó el 09/10) | La función responde 401 y no envía nada | La bandeja y, con la app abierta, el aviso del sistema |
| El dispositivo desinstaló la app o borró los datos | El servicio de push responde 404 o 410 para ese endpoint; los otros dispositivos reciben igual | La bandeja en los otros dispositivos |
| El celular está apagado | El servicio de push guarda el mensaje 24 horas | El push al prenderlo, y la bandeja |

**Lo que no se hizo:** cuando un endpoint responde 404 o 410, la fila queda en `suscripciones_push`. Lo correcto sería borrarla, pero la función no toca la base a propósito. No rompe nada: ese envío falla y los demás siguen.

### 16.13 El incidente del 09/10 y cómo se verifica un envío

- **Qué pasó:** hasta las 18:54, la función respondió 401 a todo, porque `PUSH_SECRETO` no coincidía con el valor de Vault. Parecía que el push andaba, pero lo que se veía eran avisos del canal 2 (la app seguía abierta o en segundo plano).
- **Cómo se detectó:** la alerta de venta, con la app cerrada de verdad, no llegó.
- **Cómo se encontró:** en `select status_code, content from net._http_response order by created desc;` y en los logs de la función.
- **Cómo se arregló:** pegando en `PUSH_SECRETO` el valor de Vault, sin redesplegar. Después, un QR validado con la app cerrada llegó en Android: la función respondió 200 y FCM, 201 para los dos dispositivos.
- **Lo que se aprendió:** el push se verifica con la app cerrada del todo y mirando las respuestas, no a ojo.

### 16.14 En cada dispositivo

| Dispositivo | Cómo llega | Cuidado |
|---|---|---|
| Android (Chrome) | Con la app y el navegador cerrados | Ahorro de batería y "No molestar". Probado |
| PC (Chrome o Edge) | Con la pestaña cerrada, mientras el navegador siga corriendo; si está cerrado, al abrirlo | Notificaciones de Chrome habilitadas en Windows |
| iPhone (iOS 16.4 o superior) | Solo con la PWA instalada (Safari → Compartir → Agregar a inicio) | Hay que iniciar sesión y tocar **Activar avisos** dentro de la app instalada. No se probó |

### 16.15 Preguntas probables

1. **¿Por qué una Edge Function si no se vio en clase?**
   Porque un push hay que cifrarlo y firmarlo, Postgres no puede, y la clave privada no puede ir en Angular. Es la única Edge Function del proyecto y hace una sola cosa.

2. **¿Por qué no Firebase u OneSignal?**
   Son servicios externos con su cuenta, su SDK y sus claves. Con una función de 61 líneas dentro del mismo Supabase alcanzaba.

3. **¿Por qué pg_net y no llamar a la función desde Angular?**
   Porque los avisos nacen en la base (el cron, `validar_qr`) sin que haya una pantalla abierta. Y si lo llamara Angular, la función quedaría abierta a cualquier usuario.

4. **¿Por qué `verify_jwt` apagado? ¿No es inseguro?**
   El JWT es para llamadas de usuarios, y acá quien llama es la base. La protección es el secreto compartido: sin él, 401.

5. **¿Qué pasa si el push falla?**
   Nada se pierde: el aviso ya está en la bandeja, que es el canal garantizado. El push es un complemento (RNF-16).

6. **¿Cómo evitás que llegue dos veces con la app abierta?**
   El aviso del sistema y el push usan el mismo `tag`, `notificacion-{id}`: el segundo reemplaza al primero.

7. **¿Qué pasa si dos personas usan el mismo celular?**
   El `endpoint` es único por navegador. Al salir se borra la suscripción; si la baja falla, la próxima cuenta que active avisos se queda con ella.

Frase para el oral: "La base escribe el aviso. Un trigger se lo pasa por pg_net a una Edge Function que solo le hace caso a la base, y la función lo cifra, lo firma y se lo manda al servicio de push del celular. Si algo falla, el aviso igual está en la bandeja."

---

## 17. Recuperación de contraseña a fondo

La sección 2.25 la resume en una tabla. Esta sección la recorre entera: qué archivos y funciones intervienen, qué hace Supabase de su lado en cada paso, y por qué termina siendo una limitación.

### 17.1 En un minuto

- **Qué pide el documento:** RF-02, "inicio y cierre de sesión, recuperación de contraseña".
- **Qué está hecho:** el flujo completo, en dos pantallas. `/recuperar-clave` pide el email y Supabase manda un mail con un enlace. El enlace abre `/restablecer-clave`, donde se elige la clave nueva.
- **Qué lo limita:** el servidor de correo que trae Supabase es de prueba. **Solo le entrega mails a las direcciones de los miembros del proyecto** y permite muy pocos envíos por hora. Para cualquier otra persona, el mail no llega nunca, y Supabase no avisa ningún error.
- **Qué faltaría para que funcione con cualquiera:** cargar un servidor de correo propio (SMTP) en el panel de Supabase. **No hay que tocar código.**

Entonces la respuesta corta es: **el código está completo y funciona; lo que no funciona para un usuario real es el envío del mail, que depende de una configuración externa que quedó fuera del alcance.**

### 17.2 Las piezas

| Pieza | Dónde | Qué hace |
|---|---|---|
| Enlace "Olvidé mi clave" | `publico/login/login.html` (línea 52) | Lleva a `/recuperar-clave` |
| Ruta `/recuperar-clave` | `app.routes.ts` (línea 45) | `loadComponent` de `RecuperarClave`, con `invitadoGuard` |
| `RecuperarClave` | `publico/recuperar-clave/recuperar-clave.ts` y `.html` | Formulario del email; muestra "te enviamos un enlace" |
| `AuthApi.recuperarClave(email)` | `shared/api/auth-api.ts` (líneas 35 a 41) | `supabase.auth.resetPasswordForEmail(email, { redirectTo })` |
| Ruta `/restablecer-clave` | `app.routes.ts` (línea 51) | `loadComponent` de `RestablecerClave`, **sin guard** |
| Cliente de Supabase | `shared/services/supabase.client.ts` | Al arrancar, lee la sesión que trae el enlace en la URL |
| `SesionStore` | `shared/stores/sesion-store.ts` | Guarda esa sesión en un signal y resuelve `inicializada` |
| `RestablecerClave` | `publico/restablecer-clave/restablecer-clave.ts` y `.html` | Espera la sesión, pide la clave dos veces y la guarda |
| `ClaveNuevaDraft` | `shared/models/auth.model.ts` | `{ clave, repetir }`: el modelo del formulario |
| `VerClave` | `shared/components/ver-clave/` | Botón para mostrar u ocultar cada clave |
| `AuthApi.cambiarClave(clave)` | `shared/api/auth-api.ts` (líneas 43 a 46) | `supabase.auth.updateUser({ password: clave })` |
| `mensajeError` | `shared/utils/errores.ts` | Traduce `weak_password`, `same_password`, `over_email_send_rate_limit`, `over_request_rate_limit` y `email_address_invalid` |
| Configuración de Auth | Panel de Supabase | Proveedor Email, largo mínimo 8, Redirect URLs, plantilla del mail y servidor de correo |

No hay nada en la base de Cineverso: la clave vive en `auth.users`, que maneja Supabase. Ninguna tabla nuestra, ningún trigger y ninguna RPC la tocan.

### 17.3 La configuración del panel

Se hizo una sola vez en Authentication (SPEC, sección 7.1, pasos A y B):

| Dónde | Qué se configuró | Para qué sirve en la recuperación |
|---|---|---|
| URL Configuration → Site URL | `https://tp1-sistema-cine.vercel.app` | Adónde manda Supabase si la dirección pedida no está permitida |
| URL Configuration → Redirect URLs | `http://localhost:4200/**` y `https://tp1-sistema-cine.vercel.app/**` | Lista de direcciones a las que el enlace puede volver. `/restablecer-clave` entra por el comodín `/**` |
| Sign In / Providers → Email | Habilitado, con largo mínimo de clave 8 y **Confirm email** apagado (D-16) | La clave nueva también la controla Supabase; si es corta responde `weak_password` |
| Emails → Reset password | La plantilla del mail (traducirla era opcional) | El texto del mail; el enlace va en `{{ .ConfirmationURL }}` |
| SMTP Settings | **Sin configurar**: se usa el servidor de correo de Supabase | Es lo que limita el envío (17.10) |

### 17.4 Paso 1 · Pedir el enlace

**La pantalla** (`recuperar-clave.ts`):

```ts
protected readonly borrador = signal({ email: '' });
protected readonly enviando = signal(false);
protected readonly enviado = signal(false);
protected readonly error = signal<string | null>(null);

protected readonly recuperarForm = form(
  this.borrador,
  (f) => {
    required(f.email, { message: 'Ingresá tu email' });
    email(f.email, { message: 'El email no es válido' });
  },
  { submission: { action: async () => this.enviar() } },
);
```

1. **Signal Forms, como en clase:** el modelo es el signal `borrador`. Las reglas son `required` y `email`, con el mensaje en español. `submission.action` es lo que corre al enviar, y solo corre si el formulario es válido; si no, `submit` marca todos los campos y aparecen los errores.
2. **Cuatro signals de estado:** `enviando` (para el botón), `enviado` (para cambiar de pantalla), `error` (el mensaje traducido) y el `borrador`.
3. **`enviar()`:** pone `enviando` en `true`, limpia el error y llama a `authApi.recuperarClave(this.borrador().email.trim())`. El `trim()` saca los espacios que se cuelan al pegar el email. Si sale bien, `enviado` pasa a `true`. Si falla, `mensajeError` traduce el error. En el `finally`, `enviando` vuelve a `false` pase lo que pase.

**La plantilla** (`recuperar-clave.html`):

- `@if (enviado())` reemplaza el formulario por "Si hay una cuenta con ese email, te enviamos un enlace para elegir una clave nueva", con `role="status"` para que un lector de pantalla lo anuncie.
- Si no, el formulario con `[formRoot]` y `(submit)="$event.preventDefault()"`, para que el navegador no recargue la página.
- El campo usa `[formField]` y `@let correo = recuperarForm.email()`. Los errores se muestran con `@for` solo si el campo fue tocado y no es válido.
- El botón tiene `[disabled]="enviando()"` y dice "Enviando…" mientras espera: un doble clic no manda dos mails.

**La Api** (`auth-api.ts`):

```ts
// manda el mail con el enlace que abre la pantalla para elegir una clave nueva
async recuperarClave(email: string): Promise<void> {
  const { error } = await supabase.auth.resetPasswordForEmail(email, {
    redirectTo: `${window.location.origin}/restablecer-clave`,
  });
  if (error) throw error;
}
```

- `redirectTo` se arma con `window.location.origin`: si pedís el enlace en local vuelve a `http://localhost:4200/restablecer-clave`, y si lo pedís en Vercel, a la de Vercel. No hay una URL fija en el código.
- Es una Api delgada, como `TaskApi`: llama, lanza el error si hay y no guarda estado.

**Qué hace Supabase cuando recibe el pedido:**

1. Si el email tiene cuenta, genera un **token de recuperación** de un solo uso, con vencimiento (por defecto una hora; se configura en Authentication).
2. Arma el mail con la plantilla *Reset password* y lo manda por el servidor de correo configurado.
3. **Si el email no tiene cuenta, responde lo mismo, sin error.** Así nadie puede usar esta pantalla para averiguar quién está registrado. Por eso la app tampoco distingue: dice "si hay una cuenta…".
4. Si se piden demasiados mails, responde `over_email_send_rate_limit` (o `over_request_rate_limit`), que la app muestra como "Se enviaron demasiados mails. Probá de nuevo en un rato".

### 17.5 Paso 2 · El mail y el enlace

1. El mail trae un enlace a Supabase, no a Cineverso: `https://xrbjzgqosscoowmkwnxf.supabase.co/auth/v1/verify?token=…&type=recovery&redirect_to=…`.
2. Al tocarlo, Supabase verifica el token:
   - **Si es válido**, lo marca como usado, crea una sesión para ese usuario y redirige a `redirect_to`, con la sesión en el **fragmento** de la URL (lo que va después del `#`):
     `https://tp1-sistema-cine.vercel.app/restablecer-clave#access_token=…&refresh_token=…&expires_in=…&type=recovery`
   - **Si venció o ya se usó**, redirige con un error en el fragmento (`#error=access_denied&error_code=otp_expired…`).
   - **Si `redirect_to` no está en las Redirect URLs**, manda a la Site URL (la portada) y el usuario nunca ve la pantalla de la clave nueva. Por eso hacen falta los comodines `/**`.
3. **El fragmento no viaja al servidor:** el navegador no lo manda en el pedido, así que el token no queda en los registros de Vercel. Solo lo lee el JavaScript de la página.

Este modo, donde la sesión llega en el fragmento, se llama **flujo implícito** y es el que usa `supabase-js` 2 por defecto cuando el cliente se crea sin opciones, como en `supabase.client.ts`. Tiene una ventaja práctica: el enlace sirve en cualquier dispositivo. Podés pedirlo en la computadora y abrir el mail en el celular. Con el otro modo (PKCE), el enlace solo funciona en el mismo navegador donde se pidió.

### 17.6 Paso 3 · Angular arranca y lee la sesión del enlace

Esto pasa solo, sin código propio para la recuperación:

1. `supabase.client.ts` crea el cliente con `createClient(url, clave)`. Las opciones por defecto incluyen `detectSessionInUrl: true` y `persistSession: true`.
2. Al iniciarse, `supabase-js` ve el `#access_token=…&type=recovery` en la URL, guarda la sesión en el `localStorage`, **borra el fragmento** de la barra de direcciones y emite el evento `PASSWORD_RECOVERY`.
3. `SesionStore`, en su constructor, escucha esos eventos con `authApi.alCambiarSesion(...)`:

   ```ts
   this.authApi.alCambiarSesion((_, sesion) => {
     this._sesion.set(sesion);
     this.sesionLeida.set(true);
   });
   ```

   Guarda la sesión que venga, sin mirar qué evento fue (el `_`).
4. Con `sesionLeida` en `true`, el `effect` del store carga el perfil del usuario. Cuando termina, resuelve la promesa `inicializada`.
5. Desde ahí, para toda la app el usuario **entró**: el menú muestra su cuenta y la sesión queda guardada.

### 17.7 Paso 4 · Elegir la clave nueva

**La pantalla** (`restablecer-clave.ts`):

```ts
protected readonly verificado = signal(false);
protected readonly borrador = signal<ClaveNuevaDraft>({ clave: '', repetir: '' });

protected readonly claveForm = form(
  this.borrador,
  (f) => {
    required(f.clave, { message: 'Elegí una clave nueva' });
    minLength(f.clave, 8, { message: 'La clave tiene que tener al menos 8 caracteres' });
    required(f.repetir, { message: 'Repetí la clave' });
    validate(f.repetir, ({ value, valueOf }) =>
      value() && value() !== valueOf(f.clave)
        ? { kind: 'repetir', message: 'Las claves no coinciden' }
        : null,
    );
  },
  { submission: { action: async () => this.cambiar() } },
);

constructor() {
  // espera a que supabase lea la sesión que trae el enlace del mail
  void this.sesion.inicializada.then(() => this.verificado.set(true));
}
```

1. **Esperar antes de decidir:** si la pantalla mirara la sesión apenas se crea, todavía no habría ninguna, porque `supabase-js` la está leyendo de la URL, y mostraría "el enlace venció" por error. Por eso el constructor espera `sesion.inicializada` y recién ahí pone `verificado` en `true`.
2. **Las reglas del formulario:** `required` y `minLength(8)` en la clave (el mismo mínimo que el panel de Supabase), `required` en la repetición y un `validate` propio que compara las dos. `valueOf(f.clave)` lee el valor de **otro** campo dentro de la regla de `repetir`.
3. **`cambiar()`:** el mismo molde que `enviar()`: `guardando`, error traducido y `finally`. Llama a `authApi.cambiarClave(this.borrador().clave)` y, si sale bien, `cambiada` pasa a `true`.

**La plantilla** (`restablecer-clave.html`) es una cadena de `@if` con cuatro estados, en este orden:

| Estado | Condición | Qué muestra |
|---|---|---|
| Esperando | `!verificado()` | "Verificando el enlace…" |
| Listo | `cambiada()` | "Listo, ya podés usar tu clave nueva" y un enlace a la cartelera |
| Enlace inválido | `!sesion.sesion()` | "El enlace no es válido o ya venció" y un enlace para pedir otro |
| Formulario | El resto | Las dos claves, cada una con `VerClave` |

El orden importa: "listo" va antes que "inválido", y los dos después de "verificando", para que nunca aparezca un mensaje equivocado mientras carga.

Cada campo usa `[type]="verClave() ? 'text' : 'password'"` y `autocomplete="new-password"`, para que el navegador ofrezca generar una clave y no complete la vieja. `<app-ver-clave [(visible)]="verClave" />` cambia ese signal con `model()`.

**La Api:**

```ts
async cambiarClave(clave: string): Promise<void> {
  const { error } = await supabase.auth.updateUser({ password: clave });
  if (error) throw error;
}
```

`updateUser` manda la clave nueva a Supabase Auth con el token de la sesión que abrió el enlace. Supabase la guarda cifrada en `auth.users`. Los errores posibles ya tienen su texto en `mensajeError`:

| Código | Cuándo | Mensaje |
|---|---|---|
| `weak_password` | No cumple el largo mínimo del panel | "La clave es muy débil: usá al menos 8 caracteres." |
| `same_password` | Es igual a la anterior | "La clave nueva tiene que ser distinta de la anterior." |

Después de guardar, **el usuario queda con la sesión iniciada**: no tiene que volver a ingresar.

### 17.8 Por qué cada ruta tiene el guard que tiene

| Ruta | Guard | Por qué |
|---|---|---|
| `/recuperar-clave` | `invitadoGuard` | Quien ya entró no necesita recuperar nada: cambia la clave desde su cuenta. El guard lo manda a la portada |
| `/restablecer-clave` | Ninguno | El enlace del mail **inicia una sesión**. Si tuviera `invitadoGuard`, el usuario recién llegado ya figuraría como registrado y el guard lo echaría antes de que pueda elegir la clave |

Las dos van con `loadComponent`: Signal Forms no entra en la carga inicial (sección 2.21).

### 17.9 Todo el recorrido

```text
 Usuario             Angular                              Supabase Auth                 Servidor de correo
    │ "Olvidé mi clave"   │                                       │                                │
    │────────────────────►│ /recuperar-clave (invitadoGuard)      │                                │
    │ escribe el email    │                                       │                                │
    │────────────────────►│ resetPasswordForEmail(email,          │                                │
    │                     │   { redirectTo: origen/restablecer })─►│ ¿tiene cuenta?                 │
    │                     │◄──────────────── OK (siempre) ─────────│ sí: crea un token de un uso    │
    │ "si hay una cuenta, │                                       │────── mail con el enlace ─────►│
    │  te enviamos…"      │                                       │                                │── solo si el email
    │                     │                                       │                                │   es del equipo
    │ toca el enlace del mail ───────────────────────────────────►│ /auth/v1/verify                │
    │                     │                                       │ token válido → crea la sesión  │
    │◄──── redirige a /restablecer-clave#access_token=…&type=recovery                              │
    │                     │ supabase-js lee el #, guarda la sesión│                                │
    │                     │ y lo borra; SesionStore la recibe     │                                │
    │                     │ inicializada → verificado = true      │                                │
    │ clave nueva x2      │                                       │                                │
    │────────────────────►│ updateUser({ password }) ────────────►│ guarda la clave en auth.users  │
    │◄──── "Listo" ───────│◄──────────────────────────────────────│                                │
```

### 17.10 La limitación: qué pasa exactamente

El problema no está en el código, sino en **quién manda el mail**. Como SMTP Settings quedó sin configurar, Supabase usa su propio servidor de correo, que está pensado solo para probar:

1. **Solo le entrega mails a las direcciones de los miembros de la organización del proyecto** en Supabase. A cualquier otra dirección no le manda nada.
2. **No avisa que no lo mandó.** `resetPasswordForEmail` responde bien igual, como cuando el email no tiene cuenta. La app muestra "te enviamos un enlace" y el mail nunca llega. Ni Angular ni el usuario tienen forma de enterarse.
3. **Tiene un límite muy bajo de envíos por hora para todo el proyecto.** Si se pasa, la app muestra "Se enviaron demasiados mails. Probá de nuevo en un rato".
4. Aun cuando llega, puede caer en spam, porque sale de un remitente genérico de Supabase.

**En la práctica:**

| Quién lo usa | ¿Funciona? |
|---|---|
| Vos, con un email que es miembro del proyecto en Supabase | Sí, de punta a punta |
| El profe, un cliente de prueba o cualquier otra persona | No: ve el mensaje de éxito, pero el mail no llega |

Por eso el README lo tiene en "Limitaciones conocidas" y el SPEC lo registra como decisión D-16.

**La misma razón explica la otra decisión del registro:** la confirmación de email está apagada. Si estuviera prendida, quien se registra con un email que no es del equipo nunca recibiría el mail de confirmación y no podría entrar nunca. Apagarla deja la cuenta activa al registrarse.

### 17.11 Por qué no se resolvió

La solución es cargar un **SMTP propio** en Authentication → SMTP Settings: el servidor, el puerto, el usuario, la clave y el remitente de un servicio de correo como Gmail (con una clave de aplicación), Brevo, SendGrid o Resend. **El código no cambia ni una línea:** `resetPasswordForEmail` hace lo mismo y solo cambia quién entrega el mail.

No se hizo por estos motivos:

1. **El documento de requerimientos no pide un SMTP propio.** RF-02 pide la recuperación, y el flujo está completo. El SMTP propio quedó fuera de alcance desde el principio (SPEC, sección 1.4).
2. **Suma una cuenta externa y credenciales que cuidar:** una clave de correo que tendría que vivir en el panel de Supabase y renovarse si se filtra.
3. **Algunos servicios piden más de lo que tiene el proyecto.** Por ejemplo, Resend pide verificar un dominio propio para escribirle a cualquier dirección, y Cineverso no tiene dominio: usa el de Vercel.
4. **Se priorizó lo que pedía el documento.** En el SPEC (riesgos, sección 13.3) quedó anotado como "un SMTP propio solo si el docente lo exige", y no lo exigió.

### 17.12 Qué haría para que funcione con cualquiera

1. Crear la cuenta de correo (por ejemplo, Gmail con verificación en dos pasos y una clave de aplicación).
2. En Supabase, Authentication → SMTP Settings: activar **Enable custom SMTP** y cargar el servidor (`smtp.gmail.com`), el puerto (465 o 587), el usuario, la clave de aplicación, el remitente y el nombre ("Cineverso").
3. En Authentication → Rate Limits, subir el límite de mails por hora, que con un SMTP propio deja de estar fijo.
4. Traducir la plantilla *Reset password* en Authentication → Emails.
5. Probar con un email que no sea del equipo.
6. Si se quisiera, prender **Confirm email**: con un SMTP propio, el mail de confirmación también llegaría.

Nada de esto toca Angular, la base ni el repo.

### 17.13 Detalles finos que conviene conocer

- **La pantalla no distingue una sesión de recuperación de una normal.** `SesionStore` guarda la sesión sin mirar el evento, así que alguien que ya entró y abre `/restablecer-clave` a mano también ve el formulario y puede cambiar su clave sin escribir la actual. No abre una puerta nueva: para eso ya tiene que tener la sesión iniciada, y con ella podría hacer cualquier otra cosa de su cuenta. Supabase tiene una opción, **Secure password change**, que exige volver a ingresar para cambiar la clave. Viene apagada y en el SPEC no figura que se haya prendido: revisalo en Authentication → Sign In / Providers → Email si te lo preguntan.
- **Un enlace vencido con otra sesión abierta:** si en ese navegador ya había una sesión, `supabase-js` no la borra al fallar el enlace, y la pantalla mostraría el formulario para esa cuenta en lugar de "el enlace venció".
- **Algunos correos "abren" los enlaces solos.** Ciertos servicios (por ejemplo, la protección de enlaces de Outlook) visitan el enlace para revisarlo antes que el usuario. Eso gasta el token, que es de un solo uso, y cuando la persona lo toca ve "el enlace no es válido o ya venció".
- **No está registrado que se haya probado de punta a punta.** El SPEC dice "en la demo se prueba con una dirección del equipo", pero no hay ninguna anotación de esa prueba. **Hacela antes del oral** con tu email (el que es miembro del proyecto), una sola vez, para no gastar el límite por hora.

### 17.14 Preguntas probables

1. **¿La recuperación funciona?**
   Sí: el flujo completo está hecho y funciona con un email del equipo del proyecto. Lo que no funciona para cualquiera es el envío del mail, porque el servidor de correo de Supabase es de prueba y solo le escribe al equipo.

2. **¿Por qué no aparece ningún error si el mail no llega?**
   Porque Supabase responde igual para no revelar qué emails tienen cuenta. Si dijera "no se pudo enviar", también diría quién está registrado.

3. **¿Cómo sabe la pantalla nueva quién es el usuario si no ingresó?**
   El enlace del mail inicia sesión. Supabase verifica el token y vuelve a la app con la sesión en la URL; `supabase-js` la lee sola y `SesionStore` la guarda.

4. **¿Por qué `/restablecer-clave` no tiene guard?**
   Porque el enlace ya inicia una sesión, e `invitadoGuard` echaría al usuario antes de que elija la clave.

5. **¿Qué impide que alguien desvíe el enlace a otro sitio?**
   La lista de Redirect URLs del panel: Supabase solo vuelve a localhost o a Vercel. Si `redirectTo` apunta a otro lado, manda a la portada.

6. **¿Dónde se guarda la clave nueva? ¿La ves vos?**
   En `auth.users`, que maneja Supabase, cifrada. No pasa por ninguna tabla nuestra y nadie la puede leer, ni el admin.

7. **¿Qué tendrías que cambiar para que funcione con cualquiera?**
   Nada del código: cargar un SMTP propio en el panel de Supabase.

Frase para el oral: "La recuperación está implementada completa: pido el mail con `resetPasswordForEmail`, el enlace inicia una sesión de recuperación y la pantalla nueva guarda la clave con `updateUser`. Lo que la limita es el correo de prueba de Supabase, que solo le escribe al equipo del proyecto. Con un SMTP propio funciona para cualquiera sin tocar una línea de código."

---

## 18. La cursada, clase por clase, en Cineverso

Esta sección recorre los ocho PDF de la cursada (`clase_1.pdf` a `clase_8.pdf`) tema por tema y dice, para cada uno, **dónde está en el proyecto, cómo está hecho y qué decisión técnica se tomó.** Las referencias van como `clase·lámina` (`4·25` es la Clase 4, lámina 25).

Las clases vienen de a pares: la impar presenta los temas y la par los baja a código sobre TaskFlow. Por eso la sección tiene cuatro bloques:

| Bloque | Clases | Temas |
|---|---|---|
| 18.1 | 1 y 2 | Introducción a Angular 22, TypeScript, componentes y binding |
| 18.2 | 3 y 4 | Ruteo, input y output, directivas estructurales y Signal Forms |
| 18.3 | 5 y 6 | Servicios, Supabase (base, Realtime, Auth y Storage) y deploy |
| 18.4 | 7 y 8 | Guards, pipes, directivas propias y service worker (PWA) |
| 18.5 | — | Lo que mostraron las clases y **no** se usó, y por qué |

### 18.1 Clases 1 y 2 · Introducción, TypeScript, componentes y binding

#### Introducción a Angular

| Tema (lámina) | Dónde está | Cómo está implementado | Decisión técnica |
|---|---|---|---|
| SPA o MPA (1·7) | Toda la app | Una SPA: un solo `index.html` y el router cambia las pantallas sin recargar | Es una app interactiva (mapa en vivo, checkout, panel), el caso que la lámina asigna a SPA. Sin SSR: no hace falta posicionamiento en buscadores y la PWA necesita funcionar en el cliente |
| La CLI: `ng new`, `ng serve`, `ng generate`, `ng build`, `ng test` (2·6) | `angular.json`, `package.json` | El proyecto se generó con el CLI 22.2 y cada componente, guard, pipe y directiva con `ng generate`. Los archivos tienen los nombres sin sufijo del CLI 22 (`login.ts`, no `login.component.ts`) | `skipTests` en los generadores de `angular.json`: no se generan los specs "should create", porque solo se prueban funciones puras (RNF-23) |
| Los archivos del proyecto (2·7) | Raíz y `src/` | `main.ts`, `index.html`, `styles.css`, `app.ts`, `app.config.ts`, `app.routes.ts` y `public/` cumplen el mismo rol que en TaskFlow | Se sumaron `environments/` (Clase 6) y `ngsw-config.json` (Clase 8) |
| `bootstrapApplication` y providers, sin NgModules (2·8) | `src/main.ts`, `src/app/app.config.ts` | `bootstrapApplication(App, appConfig)`. `appConfig.providers` tiene `provideRouter(routes, withComponentInputBinding())`, `LOCALE_ID`, `DATE_PIPE_DEFAULT_OPTIONS` y `provideServiceWorker` | No hay `provideHttpClient`: el proyecto no usa HttpClient (18.5). `main.ts` suma `registerLocaleData(es-AR)` para fechas y pesos argentinos (RNF-21) |
| Angular 22: OnPush por defecto y zoneless (1·6, 2·9) | Todos los componentes; `angular.json` | Ningún componente declara `changeDetection` (usan OnPush, que es el default) y no hay `zone.js` en los polyfills | Como pide la lámina 2·21, el estado se cambia con `set()` o `update()` con objetos nuevos, por ejemplo `this._peliculas.update((lista) => [...lista, peliculaId])` en `AlertasStore` |
| Decorador `@Service()` (1·6, 2·9) | `shared/api/`, `stores/`, `services/` | No se usa: todo servicio lleva `@Injectable({ providedIn: 'root' })` | El código de clase usa `@Injectable`, y la lámina dice que sigue siendo válido. Se priorizó lo que muestra el código de TaskFlow |

#### TypeScript

| Tema (lámina) | Dónde está | Cómo está implementado | Decisión técnica |
|---|---|---|---|
| Tipado estático e inferencia (2·11) | `tsconfig.json` | `strict` y `strictTemplates` activados | El proyecto se generó sin ellos; con `strictTemplates`, un campo mal escrito en el HTML no compila (RNF-08) |
| Union types en lugar de enums (2·12) | `shared/models/` | `Formato = '2D' \| '3D' \| '4D' \| '5D'`, `Rol`, `Clasificacion`, `EstadoButaca`, `PasoCompra`, `Sector` | Como recomienda la lámina: no generan código y se comparan en línea en el template (`butaca().tipo === 'vip'`). En la base, el mismo criterio: `text` con `check (... in (...))` |
| Uniones discriminadas (2·12) | `shared/models/validacion.model.ts` | `ResultadoValidacionModel` tiene una variante por `resultado` (`valido`, `ya_usado`, `fuera_de_horario`…), cada una con sus campos | `ResultadoValidacion` hace `@switch` sobre `resultado` y en cada rama el compilador sabe qué campos existen |
| Interfaces con sufijo `Model` (2·13) | `shared/models/*.model.ts` (18 archivos) | `PeliculaModel`, `FuncionModel`, `CompraModel`, `PerfilModel`… | Con los nombres de columna de la base (`fecha_estreno`), para que `data as PeliculaModel` sea cierto sin mapear (sección 2.19) |
| Genéricos (2·13) | `rules/sala.ts`, `rules/precio.ts`, `models/sala.model.ts` | `agruparButacas<T>`, `entradasConCombos<T>` y `FilaButacas<T>` sirven para la butaca del editor y para la del mapa | Una sola función para dos tipos parecidos, sin duplicar |
| Utility types (2·14) | `shared/models/` | `Omit<CategoriaModel, 'id'>` para los borradores, `Pick<…>` para `ButacaCambios` y `PeliculaFila`, `Partial<…>` para `PerfilPatch`, `Record<Rol, string>` para las etiquetas, `Exclude<Rol, 'cliente'>` para `RolInterno` | "Un solo modelo de dominio y todo lo demás derivado": si cambia una columna, el compilador marca cada derivado |
| Clases y decoradores (2·15) | Todo el proyecto | `@Component`, `@Injectable`, `@Pipe` y `@Directive`; las interfaces para los datos y las clases para el comportamiento | — |

#### Componentes

| Tema (lámina) | Dónde está | Cómo está implementado | Decisión técnica |
|---|---|---|---|
| Árbol de componentes: los datos bajan, los eventos suben (2·5) | `publico/compra/` | `Checkout` contiene los cinco pasos; `PasoButacas` contiene `MapaButacas`, que emite la butaca tocada | Los pasos no reciben `input`: comparten `CompraStore` y `ButacasStore` (sección 13.5) |
| Componente standalone con selector, template e imports (1·13, 2·17) | Los 62 componentes | Cada uno con `selector`, `templateUrl`, `styleUrl` e `imports` propios | Prefijo `app-` en todos (`app-mapa-butacas`) |
| `input()`: `required`, valor por defecto, `transform` y `alias` (2·19) | `MapaButacas`, `TarjetaPelicula`, `VistaComprobante`, `HighlightButaca`, `Checkout` | `input.required<ButacaVista[]>()`, `input(false)`, `input.required<number, unknown>({ transform: numberAttribute })` y `input.required({ alias: 'appHighlightButaca' })` | `required` donde el componente no tiene sentido sin el dato: si el padre se olvida, no compila |
| `output()` (2·20) | `MapaButacas` (`alternada`), `EscanerQr` (código leído), `MenuPrincipal` (`salir`), `ContadorRetencion` (vencida), `SelectorHorario`, `FormularioPerfil` (`guardado`) | El hijo avisa y el padre decide | `MenuPrincipal` emite `salir` y la raíz da de baja el push y cierra la sesión: el menú no sabe de Supabase |
| `model()` (2·20) | `admin/selector-*`, `selector-dias`, `selector-fecha`, `selector-hora`, `estrellas`, `ver-clave`, `formulario-perfil` | `visible = model(false)` en `VerClave`; el padre usa `[(visible)]="verClave"` | Controles propios que funcionan como un campo más del formulario |
| Ciclo de vida y hooks (1·15, 2·21) | `checkout.ts`, `escaner-qr.ts`, stores | `ngOnDestroy` en el checkout (llama a `reiniciar()`) y en el escáner (apaga la cámara). La carga inicial va en el constructor con `effect`, sin `ngOnInit` | Como dice la lámina: `computed` y `effect` reemplazan a `ngOnChanges`, y `ngOnDestroy` queda para limpiar recursos |
| Encapsulación de estilos y `:host` (2·22) | Los `.css` de cada componente | Encapsulación emulada (la de por defecto); `:host` aparece en 56 hojas de estilo | Los colores y espacios comunes van como variables en `styles.css`; cada componente estila solo lo suyo |

#### Binding y templates

| Tema (lámina) | Dónde está | Cómo está implementado | Decisión técnica |
|---|---|---|---|
| Interpolación y property binding (1·17, 2·24) | Todas las plantillas | `{{ pelicula().titulo }}`, `[disabled]="enviando()"`, `[type]="verClave() ? 'text' : 'password'"` | — |
| Event binding y `$event` (1·18, 2·25) | Formularios, `MenuPrincipal`, `BandejaNotificaciones` | `(submit)="$event.preventDefault()"` en cada formulario; `(keydown.escape)` cierra el menú y la bandeja | Las expresiones de `{{ }}` y `[ ]` leen signals o `computed`; los cálculos van en `( )` o en el store, como pide la lámina 2·25 |
| Class y style binding (1·18, 2·26) | `HighlightButaca`, `MenuPrincipal`, `ContadorRetencion` | `[class.butaca--vip]` en cada butaca, `[class.abierto]` en los grupos del menú, `[class.urgente]` en el contador cuando queda poco tiempo y `[style.--i]` (una variable de CSS por elemento del menú) | Los estados visuales se declaran en el template, sin tocar el DOM a mano |
| Control flow `@if`, `@for` y `@switch` (1·19, 2·26) | Todas las plantillas | `@switch` en el checkout, el resultado de validación y el estado de la compra; `@for` con `track` siempre | No hay ningún `*ngIf` ni `*ngFor` (RNF-04) |

### 18.2 Clases 3 y 4 · Ruteo, input y output, directivas estructurales y formularios

#### Ruteo

| Tema (lámina) | Dónde está | Cómo está implementado | Decisión técnica |
|---|---|---|---|
| `provideRouter` y `<router-outlet>` (4·5) | `app.config.ts`, `app.html` | `provideRouter(routes, withComponentInputBinding())`; el outlet va en la raíz, debajo del menú | — |
| `Routes` como array tipado: parámetros, redirección y comodín (3·6, 4·6) | `app.routes.ts` | 37 rutas; `peliculas/:peliculaId`, `{ path: 'admin', redirectTo: 'admin/peliculas', pathMatch: 'full' }` y `{ path: '**', redirectTo: '' }` | Como en la tabla final de TaskFlow (8·8), lo desconocido vuelve a la portada en lugar de una página 404. El orden importa: `admin/peliculas/nueva` va antes de `admin/peliculas/:peliculaId` |
| Carga diferida con `loadComponent` (3·6) | `app.routes.ts` | Todas las pantallas menos `Inicio`, `Comprobantes` y `Privacidad` | `Comprobantes` va sin diferir porque tiene que abrir sin conexión. Se descartó `loadChildren` por área (sección 2.11) |
| Navegación declarativa y programática (3·7, 4·7) | `MenuPrincipal`, pantallas | `routerLink` y `routerLinkActive="activo"` en el menú y en la botonera del candy; `queryParams` para el `volver` del login (`BotonAlerta`, ficha); `router.navigateByUrl(...)` después de ingresar o salir | `volver` pasa por `rutaSegura` (`utils/rutas.ts`): solo acepta rutas internas, para que un enlace armado no lleve a otro sitio |
| El parámetro de la URL como `input` (3·10, 4·8) | `checkout.ts`, `pelicula-form.ts`, `funcion-form.ts`, `sala-butacas.ts` | `peliculaId = input.required<number, unknown>({ transform: numberAttribute })` gracias a `withComponentInputBinding` | Sin `ActivatedRoute` ni suscripciones, como en `TaskDetail` |

#### Input y output en profundidad

| Tema (lámina) | Dónde está | Cómo está implementado | Decisión técnica |
|---|---|---|---|
| `input.required`, `transform` y `alias` (4·10, 4·11) | `checkout.ts`, `highlight-butaca.ts`, `mascara-entrada.ts` | `numberAttribute` convierte el texto de la URL en número; `alias` deja que la directiva reciba su dato con su propio nombre (`[appHighlightButaca]="butaca"`) | No se usa `booleanAttribute`: ningún componente recibe un atributo booleano suelto |
| `model()` como el `PriorityPicker` (4·12) | `admin/selector-generos/` y el resto de los selectores | `SelectorGeneros` recibe y devuelve la lista de géneros elegidos con `model()`, y `PeliculaForm` lo usa con `[formField]` | Es la técnica de clase para un control propio (sección 2.21) |
| `output()` tipado: el hijo avisa, el padre decide (4·13) | `escaner-qr.ts` → `validacion.ts` | `EscanerQr` emite el código leído y `Validacion` decide qué hacer con él | El escáner no sabe de validaciones: se podría reutilizar |

#### Directivas estructurales y control flow

| Tema (lámina) | Dónde está | Cómo está implementado | Decisión técnica |
|---|---|---|---|
| `ng-template`, `TemplateRef` y `ViewContainerRef` (3·13, 4·15) | `shared/directives/restrict-edad.ts` | `*appRestrictEdad` inyecta `TemplateRef` y `ViewContainerRef` y llama a `createEmbeddedView` o `clear` | Es la única estructural propia; el resto es control flow |
| `*ngIf` y `*ngFor` como legado (3·14, 4·16) | — | No aparecen en el proyecto | "Código nuevo → control flow, siempre" (4·18) |
| `@empty`, `@let` y variables del loop (4·18) | Listados y formularios | `@empty` en 23 plantillas (la lista vacía sin un `@if` aparte); `@let` en 29 (por ejemplo, `@let correo = recuperarForm.email()`); `$index` en 8 | — |

#### Signal Forms

| Tema (lámina) | Dónde está | Cómo está implementado | Decisión técnica |
|---|---|---|---|
| El formulario nace de un signal del dominio (3·17, 4·20) | Las 23 pantallas con formulario | `borrador = signal<ClaveNuevaDraft>({ clave: '', repetir: '' })` y los `*Draft` de `models/` | Los borradores son tipos aparte (`PeliculaDraft`), porque el formulario trabaja con texto y la fila con fechas y números |
| `form()` y `[formField]` (3·18, 4·21) | Ídem | `form(this.borrador, reglas, { submission })` y `[formField]="claveForm.clave"` | — |
| Validación declarada junto al formulario (3·19, 4·22) | Ídem | `required` (55 usos), `validate` (31), `min` (15), `maxLength` (14), `max` (7), `pattern` (6), `email` (4) y `minLength` (3), todos con `{ message }` en español | Las reglas que también garantiza la base (edad, precios, fechas) llaman a la misma regla pura de `rules/` |
| Los errores son signals: `@if` + `@for` (4·23) | Ídem | `@if (campo.touched() && !campo.valid()) { @for (e of campo.errors(); track e.kind) {...} }` | — |
| Envío con `[formRoot]` y `submission.action` (4·24) | Ídem | `[formRoot]="claveForm"` y `submission: { action: async () => this.cambiar() }` | `onInvalid: 'touch'` no se escribe: en Angular 22.2 `submit` ya marca todos los campos y escrito como en la lámina no compila (sección 12.3) |
| El mismo formulario da de alta y edita (4·25) | `admin/pelicula-form/` | `/admin/peliculas/nueva` y `/admin/peliculas/:peliculaId` usan `PeliculaForm`; un `effect` lee el `peliculaId` y, si hay uno, precarga con `borrador.set(borradorDesdePelicula(pelicula))` | Idéntico a `TaskForm`: la ruta decide y un `set()` precarga |
| Validador propio (3·19) | `shared/validators/butacas-contiguas.ts` | `butacasContiguas(campo)` es una regla sobre `validate()` que llama a `motivoNoContiguas` | RNF-28 y RN-25: el motivo se muestra antes de pagar y la base lo vuelve a controlar |

### 18.3 Clases 5 y 6 · Servicios, Supabase, Auth, Storage y deploy

#### Servicios e inyección

| Tema (lámina) | Dónde está | Cómo está implementado | Decisión técnica |
|---|---|---|---|
| El componente muestra, el servicio consulta (5·5) | `shared/api/` (21 clases) | Una Api por dominio que llama a Supabase, lanza el error y devuelve datos | Separa la pantalla del backend (RNF-07) |
| `providedIn: 'root'` e `inject()` (5·5) | Stores, Apis y servicios | Una sola instancia para toda la app; `private readonly authApi = inject(AuthApi)` | `inject()` también se usa dentro de los guards funcionales, como dice la lámina |
| HttpClient, `toSignal` y `httpResource` (5·6, 5·7) | — | No se usan | Supabase trae su propio cliente con promesas, como anticipa la lámina 5·7: "mismo principio, todo termina en un signal" (18.5) |

#### Supabase: base de datos

| Tema (lámina) | Dónde está | Cómo está implementado | Decisión técnica |
|---|---|---|---|
| Crear el proyecto y la tabla (6·5) | `supabase/sql/` | Scripts pegados en el SQL Editor, cada uno entre `begin;` y `commit;` | Sin CLI ni migraciones, como en clase (sección 2.18) |
| La tabla en SQL: el dominio se valida dos veces (5·10) | `tp1-sistema-cine.sql` y `rules/` | `check` en las columnas (por ejemplo, el descuento de preventa entre 0 y 100) y las mismas reglas en el formulario | Es la idea de la sección 2.1 llevada a todo el proyecto: la interfaz anticipa y la base garantiza |
| RLS con `auth.uid()` (5·10) | Políticas de las 28 tablas | Políticas "dueño de la fila" (`usuario_id = auth.uid()`) más un `for all` del admin | Lo crítico (comprar, validar, cancelar) va además por RPC, porque RLS no puede validar entre tablas (sección 2.4) |
| El cliente: `npm install`, environment y `createClient` (6·6) | `shared/services/supabase.client.ts` | `export const supabase = createClient(environment.supabaseUrl, environment.supabaseAnonKey)` | Un solo cliente para toda la app; solo lo importan las Apis |
| El store sobre datos reales (6·7) | `stores/cartelera-store.ts` | `cargar()` trae la cartelera y la vuelca al signal | En TaskFlow el store llama directo a la base. Acá se separó: el store llama a la Api, como `TaskApi` del código de clase |
| CRUD con `insert().select().single()` y "error primero" (6·8) | `candy-api.ts`, `pelicula-api.ts`, `genero-api.ts`, `perfil-api.ts`, `configuracion-api.ts` | Cada mutación revisa `error` antes de seguir: `if (error) throw error` | Los ABM del admin usan la Api directa, sin store (sección 13.5) |

#### Supabase: Realtime

| Tema (lámina) | Dónde está | Cómo está implementado | Decisión técnica |
|---|---|---|---|
| Habilitar la tabla y suscribirse (5·11, 6·10) | `tp1-sistema-cine.sql`, `ocupacion-api.ts`, `notificacion-api.ts` | `alter publication supabase_realtime add table public.ocupaciones, public.notificaciones` y `.channel('ocupaciones-cambios').on('postgres_changes', ...)` | Toda la tabla, filtrada en el callback, como el canal de `tasks` (se descartó el `filter` en la suscripción, sección 12.2) |
| Los eventos actualizan el signal (6·11) | `butacas-store.ts`, `notificaciones-store.ts` | `switch (cambio.eventType)` con `INSERT`, `UPDATE` y `DELETE`, igual que `applyChange` | — |
| Buenas prácticas: cerrar el canal, RLS y carga inicial (6·13) | `ButacasStore.reiniciar()`, `NotificacionesStore` | `removeChannel` en `reiniciar()`, llamado desde el `ngOnDestroy` del checkout. RLS activa: la bandeja solo recibe los avisos propios | Un ajuste: **primero escucha y después lee**, para no perder un cambio que llegue entre la carga y la suscripción |

#### Supabase: Auth y Storage

| Tema (lámina) | Dónde está | Cómo está implementado | Decisión técnica |
|---|---|---|---|
| Registro y login con email (5·13) | `auth-api.ts`, `publico/registro/`, `publico/login/` | `signUp` y `signInWithPassword` | El registro tiene dos pasos para reutilizar el formulario de perfil (sección 2.21) |
| OAuth con GitHub (5·14, 6·15) | `auth-api.ts` (`ingresarConProveedor`) | `signInWithOAuth({ provider, options: { redirectTo } })` | Se sumó Google con el mismo mecanismo (RF-01) |
| La sesión como signal con `onAuthStateChange` (5·13) | `stores/sesion-store.ts` | `alCambiarSesion((_, sesion) => this._sesion.set(sesion))` | `listo()` espera además el perfil, porque el rol vive en `perfiles` (sección 2.21) |
| El componente de login (6·16) | `publico/login/` | Email y clave, GitHub y Google, y el "Acceso para la corrección" | Mismo nombre y ruta que en la lámina (`Login`, `/login`) |
| Proteger la ruta con un guard; "el guard no es seguridad" (6·17) | `shared/guard/` | Guards funcionales con `inject()` que devuelven `true` o `router.parseUrl(...)` | La seguridad la dan el RLS y las RPC (sección 2.11) |
| Storage: subir y `getPublicUrl` (5·15, 6·18) | `api/imagen-api.ts` | `upload` a `posters` o `productos` y `getPublicUrl`; la ruta es `{id}/{nombre normalizado}` | Buckets **públicos** (son imágenes del catálogo, sin datos personales) que **solo el admin escribe**. TaskFlow usa uno privado por usuario porque guarda adjuntos personales |

#### Deploy

| Tema (lámina) | Dónde está | Cómo está implementado | Decisión técnica |
|---|---|---|---|
| Environments y claves: la publicable va al navegador, `service_role` nunca (5·19, 6·20) | `src/environments/`, `angular.json` | `environment.ts` y `environment.prod.ts` con `supabaseUrl`, `supabaseAnonKey` y `vapidPublicKey`; `fileReplacements` en producción | Nadie usa `service_role`: ni para las cuentas internas (sección 2.16) ni para el push (16.11) |
| GitHub y Vercel: cada push a `main` despliega (5·18, 6·21 a 6·23) | Repo privado y Vercel | Preset Angular con salida `dist/tp1-sistema-cine/browser` | Las claves van en los archivos de entorno y no en las variables de Vercel: son públicas por diseño y Angular las necesita al compilar |
| Errores típicos: el refresco de una ruta (6·27) | — | Sin `vercel.json`: el preset de Vercel manda cualquier ruta a `index.html` | — |

### 18.4 Clases 7 y 8 · Guards, pipes, directivas y service worker

#### Guards

| Tema (lámina) | Dónde está | Cómo está implementado | Decisión técnica |
|---|---|---|---|
| La familia completa: funciones con `inject()` que devuelven `true` o un `UrlTree` (7·5) | `shared/guard/` | Los seis guards son funciones (`CanMatchFn`, `CanActivateFn`, `CanDeactivateFn`) y redirigen con `router.parseUrl(...)` | Se usó la variante con `await sesion.listo()`, porque el rol llega después que la sesión |
| `canActivate` (7·6, 8·5) | `invitado-guard.ts`, `puede-comprar-guard.ts` | `invitadoGuard` saca al registrado del login; `puedeComprarGuard` manda a completar el perfil antes de comprar | — |
| `canDeactivate` (7·6, 8·6) | `pending-changes-guard.ts` | Recibe la pantalla y le pregunta `cambiosPendientes()`; si hay, `confirm(pantalla.mensajeSalida())` | Una interfaz `ConCambiosPendientes` en lugar de atarlo a un solo componente: lo usan 19 rutas. En los formularios, `cambiosPendientes()` lee el `dirty()` del formulario, como el `dirty` de `TaskForm` |
| `canMatch` con carga diferida (7·7) | `admin-guard.ts`, `empleado-guard.ts`, `cliente-guard.ts` | `canMatch: [adminGuard]` en cada ruta del admin: sin el rol, el chunk ni se descarga | Un guard por rol y no una fábrica `rolGuard(roles)`, para que cada uno se lea como el `adminGuard` de la lámina |
| `resolve` con `withComponentInputBinding` (7·7, 8·7) | `pelicula-resolver.ts`, `pelicula-detalle.ts` | `ResolveFn` que trae la película; llega a la ficha como `input` | La ficha nace con su película, sin estado vacío |
| La tabla de rutas como política de acceso (8·8) | `app.routes.ts` | Pública, `canActivate`, `canMatch` por rol, `canDeactivate` en formularios y `resolve` en la ficha | Se lee de arriba hacia abajo como la tabla final de TaskFlow |

#### Pipes

| Tema (lámina) | Dónde está | Cómo está implementado | Decisión técnica |
|---|---|---|---|
| `@Pipe` + `transform()` (7·10, 8·10) | `shared/pipes/` | `duracion` (minutos → "2 h 15 min"), `precioArs` (monto → "$ 12.500,00") y `edad` | La lógica está en `rules/formato.ts` y `rules/edad.ts`: el pipe solo la llama, y la regla se prueba con Vitest |
| Pipe con argumento (8·11) | `edad-pipe.ts` | `transform(fechaNacimiento, aFecha = new Date())`: `{{ nacimiento \| edad }}` o con una fecha | Con valor por defecto, el uso simple queda limpio, como `priorityBadge` |
| Pura o impura (7·11, 8·12) | — | Las tres son puras | Para el tiempo que corre (la cuenta regresiva de la retención) se usó la "alternativa seria" de la lámina 8·12: un signal `_ahora` con intervalo en `ButacasStore` y un `computed`, no una pipe impura |

#### Directivas

| Tema (lámina) | Dónde está | Cómo está implementado | Decisión técnica |
|---|---|---|---|
| Tres sabores: componente, atributo y estructural (7·13) | `shared/directives/` | Dos de atributo (`appHighlightButaca`, `appMascara`) y una estructural (`*appRestrictEdad`) | El criterio de la lámina: si modifica un elemento que ya existe, atributo; si decide si un bloque existe, estructural |
| Directiva de atributo con `host` y signals (7·14, 8·15) | `highlight-butaca.ts` | `host` con `[class.butaca--vip]`, `[class.butaca--vendida]`, `[attr.aria-label]`, `[attr.aria-pressed]`; el dato entra por `input.required` con `alias` | En lugar de un hover, pone el estado de cada butaca y su descripción para lectores de pantalla (RNF-27, RNF-20) |
| La misma directiva en varias pantallas (8·16) | `mascara-entrada.ts` | `appMascara` se usa en siete plantillas: perfil, pago, película, cupones, reportes, auditoría y el selector de hora | "Si el diseño cambia, se toca un archivo" |
| Estructural propia: `TemplateRef`, `ViewContainerRef` y `effect` (7·15, 8·17-18) | `restrict-edad.ts` | Un `effect` lee la clasificación y la edad y llama a `createEmbeddedView` o a `clear` | Muestra el bloque Comprar solo si la edad alcanza. El motivo no lo muestra la directiva: lo muestra el componente con `@if` (sección 2.12) |

#### Service worker y PWA

| Tema (lámina) | Dónde está | Cómo está implementado | Decisión técnica |
|---|---|---|---|
| `ng add @angular/pwa` (7·17-18, 8·20) | `ngsw-config.json`, `app.config.ts`, `public/` | El comando generó el service worker, el manifest y los íconos | Íconos propios en lugar de los de ejemplo (8·22) |
| La política de caché: `assetGroups` y `dataGroups` (7·18, 8·21) | `ngsw-config.json` | Grupos `inicial` (prefetch), `diferido` (lazy) y `recursos` (fuentes e íconos); datos `cartelera` con `freshness` e `imagenes` con `performance` | **Desvío a propósito:** la lámina precachea todo el JS; acá las pantallas de las áreas van en `lazy`, para que `canMatch` siga ahorrando la descarga (sección 12.3) |
| `freshness` o `performance` (7·19) | `ngsw-config.json` | La cartelera, red primero; las imágenes, caché primero | Como la tabla de la lámina: lo que cambia seguido, red primero; lo casi estático, caché primero |
| El manifest (8·22) | `public/manifest.webmanifest` | `display: standalone`, nombre, `lang` es-AR, colores e íconos `any` y `maskable` | — |
| Actualizaciones con `SwUpdate` (7·19, 8·23) | `app.ts`, `app.html` | `swUpdate.versionUpdates` con `VERSION_READY` pone un signal en `true` y un `@if` muestra el aviso | Copia directa de la lámina |
| La prueba sin red (8·24) | `ng build` y `npx http-server` | Se prueba con el build servido, porque `ng serve` no registra el service worker | Además, `publico/comprobantes/` abre sin conexión con `localStorage` (RNF-15) |

### 18.5 Lo que mostraron las clases y no se usó

| Tema (lámina) | Por qué no se usó | Qué se usó en su lugar |
|---|---|---|
| HttpClient, `toSignal` y `httpResource` (5·6, 5·7) | Todo el backend es Supabase, que trae su propio cliente con promesas | `supabase-js` en las Apis, y el resultado se vuelca a un signal en el store |
| `@Service()` (1·6, 2·9) | El código de clase usa `@Injectable` y la lámina dice que sigue siendo válido | `@Injectable({ providedIn: 'root' })` |
| `*ngIf`, `*ngFor` y `ng-template` (3·14, 4·16) | Son el modelo anterior; la lámina 4·18 dice "código nuevo → control flow, siempre" | `@if`, `@for` y `@switch` |
| Pipe impura (8·12) | La única cosa que "envejece sola" es la cuenta regresiva, y la lámina propone la alternativa con un signal | Un signal con intervalo y un `computed` en `ButacasStore` |
| Página 404 propia (4·6) | Con 37 rutas, una URL desconocida vuelve a la portada, como la tabla final de 8·8 | `{ path: '**', redirectTo: '' }` |
| `booleanAttribute` (4·11) | Ningún componente recibe un atributo booleano suelto | — |
| Chequeo exhaustivo con `never` (2·12) | Los `@switch` de las plantillas cubren todos los casos a la vista; no hay `switch` de TypeScript que lo necesite | — |
| `ngOnInit`, `afterNextRender` y `afterEveryRender` (2·21) | La carga inicial va en el constructor con `effect`, y ningún componente mide el DOM | `effect` y `ngOnDestroy` |
| SSR e hidratación (1·7) | No hace falta posicionamiento en buscadores, y la app tiene que funcionar sin conexión | SPA con service worker |
| Animaciones con `animate.enter` y `animate.leave` (1·6) | Los desplegables del menú no agregan ni quitan elementos: abren y cierran | `transition` de CSS (sección 2.24) |
| Variables de entorno en Vercel (5·19, 6·20) | La URL y las claves del front son públicas por diseño y Angular las necesita al compilar | `environment.ts` y `environment.prod.ts` con `fileReplacements` |
| Bucket privado por usuario (5·15, 6·18) | Las imágenes son del catálogo, no de los usuarios | Buckets públicos de solo lectura, con escritura solo del admin |

Frase para el oral: "Cada tema de la cursada tiene su lugar en Cineverso, con la técnica de la lámina. Donde me aparté fue a propósito: el service worker no precachea el admin para que `canMatch` siga sirviendo, la cuenta regresiva usa un signal en vez de una pipe impura, como sugiere la propia lámina, y no usé HttpClient porque todo el backend es Supabase."
