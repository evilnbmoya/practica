# Cineverso

Sistema de venta de entradas y candy bar para un cine de un solo edificio. Es una aplicación web instalable (PWA) hecha con Angular 22 y Supabase.

Trabajo Práctico N.º 1 · Programación IV · Universidad Tecnológica Nacional, Facultad Regional Avellaneda · Comisión 141.

- **Alumno:** Nahuel Boris Moya Tellez
- **Docente:** Lic. Ricardo Gastón Plazas

## Demo

- **App desplegada:** https://tp1-sistema-cine.vercel.app
- **Usuarios de prueba:** ver [Usuarios de prueba](#usuarios-de-prueba).
- **Tarjetas de prueba:** ver [Tarjetas de prueba](#tarjetas-de-prueba).

## Estado

| Fase | Contenido | Estado |
|---|---|---|
| F0 | Proyecto, Supabase y despliegue | terminada |
| F1 | Autenticación, perfiles, roles y áreas | terminada |
| F2 | Catálogo y cartelera | terminada |
| F3 | Salas, precios y funciones | terminada |
| F4 | Mapa de butacas en tiempo real | terminada |
| F5 | Compra, pago simulado y comprobante | terminada |
| F6 | Validación de QR | terminada |
| F7 | PWA, avisos y recordatorios | terminada |
| F8 | Candy y combos | terminada |
| F9 | Cupones | terminada |
| F10 | Cancelaciones y crédito | terminada |
| F11 | Reportes y auditoría | terminada |
| F16 | Push con la app cerrada | terminada |
| F14 | Alerta de venta de los próximos estrenos y preventa configurable desde el panel | terminada |
| F12 | Reseñas y Mis películas | terminada |
| F13 | Puntos (pendientes hasta la función), catálogo de recompensas y canje al pagar | terminada |

## Tecnologías

- Angular 22: componentes standalone, Signals, Signal Forms, control de flujo integrado y carga diferida.
- TypeScript 6 con `strict` y `strictTemplates`.
- Supabase: Auth, Postgres con RLS, Realtime y Storage.
- Vitest para las pruebas unitarias.
- Vercel para el despliegue.

## Cómo correrlo

Requisitos: Node.js `^22.22.3`, `^24.15.0` o `>=26.0.0`, y npm 11.

| Comando | Qué hace |
|---|---|
| `npm ci` | Instala las dependencias |
| `npm start` | Levanta la app en `http://localhost:4200` |
| `npm test -- --watch=false` | Corre las pruebas unitarias una vez |
| `npm run build` | Genera el build de producción en `dist/` |

## Configuración

### Environments

| Archivo | Uso |
|---|---|
| `src/environments/environment.ts` | Desarrollo (`ng serve`) |
| `src/environments/environment.prod.ts` | Producción. Lo reemplaza `fileReplacements` en el build |

Los dos tienen la URL del proyecto de Supabase y, en `supabaseAnonKey`, su clave **publicable**. Esa clave es pública por diseño: lo que protege los datos son las políticas RLS. La clave secreta (`service_role`) nunca está en el frontend.

### Base de datos

La base está en Supabase y garantiza las reglas críticas de salas, butacas, QR y dinero:

| Pieza | Para qué |
|---|---|
| Tablas con RLS | Cada rol ve y modifica solo lo que le corresponde |
| Funciones transaccionales (RPC) | La retención de butacas, la compra, la validación del QR y la cancelación se hacen enteras o no se hacen |
| Triggers | Crean el perfil de cada usuario nuevo, protegen la fecha de nacimiento, calculan el fin de cada función y registran la auditoría |
| Restricción de exclusión | Impide que dos funciones se superpongan en la misma sala, aunque dos administradores programen al mismo tiempo |
| pg_cron | Libera las retenciones vencidas y genera los recordatorios y las alertas |
| Realtime | Actualiza solos el mapa de butacas y la bandeja de avisos |

El esquema se aplicó desde el SQL Editor. El proyecto necesita además, desde el panel:

- **Storage:** buckets públicos `posters` y `productos`, de hasta 2 MB y solo imágenes JPEG, PNG o WebP.
- **Authentication:** email y contraseña sin confirmación de email y con clave de al menos 8 caracteres, GitHub y Google. Site URL `https://tp1-sistema-cine.vercel.app` y Redirect URLs de esa dirección y de `http://localhost:4200`. Sesión anónima habilitada (**Allow anonymous sign-ins**), para comprar sin cuenta.

## Arquitectura del proyecto

```text
Componentes  →  Stores (signals)  →  Servicios de Api  →  Supabase (RLS, RPC, triggers)
                     ↑
           reglas puras de TypeScript (anticipan las reglas de negocio)
```

- Las reglas críticas (salas, butacas, QR y dinero) las **garantiza la base de datos**, con restricciones, funciones transaccionales y triggers. La interfaz solo las anticipa.
- El código se organiza por área, y lo que comparten las áreas va en `shared/`:

| Carpeta | Contenido |
|---|---|
| `src/app/publico/` | Cartelera, ficha, compra y comprobantes guardados |
| `src/app/cliente/` | Cuenta, compras y avisos del cliente |
| `src/app/empleado/` | Validación de QR |
| `src/app/admin/` | Panel de administración |
| `src/app/shared/` | Lo que usan varias áreas |

### Árbol de carpetas

Estado al cierre de F13. Se actualiza al terminar cada fase.

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
        │   ├── mis-peliculas/
        │   ├── detalle-compra/
        │   └── notificaciones/
        ├── empleado/
        │   ├── validacion/
        │   ├── escaner-qr/
        │   └── resultado-validacion/
        └── admin/
            ├── peliculas-admin/
            ├── pelicula-form/
            ├── selector-generos/
            ├── generos-admin/
            ├── salas-admin/
            ├── sala-butacas/
            ├── funciones-admin/
            ├── programar-funciones/
            ├── funcion-form/
            ├── selector-formatos/
            ├── precios-admin/
            ├── categorias-admin/
            ├── productos-admin/
            ├── combos-admin/
            ├── selector-productos/
            ├── cupones-admin/
            ├── recompensas-admin/
            ├── cuentas-admin/
            ├── reportes/
            └── auditoria/
```

## Decisiones técnicas

| Decisión | Motivo |
|---|---|
| `strict` y `strictTemplates` activados | El compilador detecta errores de tipos en el código y en las plantillas |
| `environment.ts` y `environment.prod.ts` con `fileReplacements` | Configuración separada para desarrollo y producción |
| Carpetas de organización en inglés (`shared/api`, `shared/stores`…) y dominio en español | Las carpetas técnicas siguen la convención habitual de Angular y los nombres del negocio quedan en el idioma del cliente |
| Pruebas unitarias solo de funciones puras (`shared/rules/`, `shared/utils/`) | Las reglas de negocio se prueban sin depender de Angular ni de la red |
| Cliente de Supabase como constante de módulo (`shared/services/supabase.client.ts`) | Una sola instancia para toda la app |
| Esquema aplicado desde el SQL Editor de Supabase, sin CLI ni migraciones locales | No hace falta instalar la CLI de Supabase ni Docker |
| Colores, tipografías y espaciados como variables CSS en `styles.css` | La identidad visual se cambia en un solo lugar, sin tocar los componentes |
| Identidad: Bree Serif para títulos y Poppins para el resto (negrita en botones y precios); fondo casi negro `#12100D` y un solo violeta, `#8226EF`, con dos variantes más oscuras | El violeta da 3,1:1 sobre el fondo: va en rellenos, líneas y texto grande, nunca en texto chico. Todo el texto cumple contraste AA |
| Sombras en lugar de bordes; tarjetas de 12 px con el póster en los 3/4 de arriba | La sombra y una superficie apenas más clara separan las tarjetas sin líneas |
| Barra con grupos desplegables (`MenuPrincipal`): Cartelera y Próximamente sueltos; para el admin, Cine, Ventas, Gestión y Reportes; y "Mi cuenta" a la derecha con el nombre, el rol y Salir | Doce pantallas del admin en una sola fila no entraban. Los grupos se abren al pasar el mouse o con un clic (cada botón tiene `aria-expanded`, y Escape cierra), con un fundido que baja y crece apenas. En el celular, un menú lateral con "Mi cuenta" primero y los grupos como acordeón. Los grupos salen de una constante (`menu.model.ts`) y se recorren con `@for` |
| Botón de alerta (`BotonAlerta`) debajo de cada tarjeta de Próximamente y en la ficha, con las alertas del cliente en `AlertasStore` | La tarjeta entera es un enlace y el HTML no admite un botón adentro. Las alertas se traen una sola vez por sesión, no una vez por tarjeta. El botón aparece solo mientras la película no está a la venta; sin sesión invita a iniciarla. La tabla `alertas` deja crear, leer y borrar solo las propias, y solo de películas en Próximamente |
| Reseñas solo desde Mis películas, con `Estrellas` como control propio de Signal Forms (`model()`, igual que `SelectorGeneros`) | Mis películas lista solo lo que el cliente ya vio (vista `mis_peliculas`), así que el botón de reseñar no aparece donde la base lo rechazaría. Igual, el RLS de `resenas` exige `vio_pelicula` para crearla y deja editar solo la propia; el `unique` deja una por película. El autor lo pone un trigger con el nombre y la inicial del apellido |
| Los puntos son movimientos con `disponible_desde` (el inicio de la función), no un saldo que cambia un cron | Pendientes y disponibles se calculan comparando con la hora actual, así que pasan a disponibles solos cuando empieza la función. El canje se arma en `CompraStore` (las entradas canjeadas cubren butacas después de los combos, como en la base) y `confirmar_compra` vuelve a controlar los puntos disponibles en la misma transacción |
| Locale `es-AR` y hora de Buenos Aires (`-03:00`) en el `DatePipe` | Interfaz en español con montos en pesos y horarios de Buenos Aires |
| Página `/privacidad` con los datos que guarda la app, para qué se usan y un contacto | Google la exige para publicar el inicio de sesión con Google |
| Registro en dos pasos: email y clave, y después el perfil | Las cuentas de email, GitHub y Google completan el perfil con el mismo formulario |
| Los guards esperan la sesión y también el perfil | Justo después de entrar, el rol todavía se está cargando; sin esa espera, un admin recién logueado terminaría en la portada |
| Las pantallas de las áreas, las de ingreso, la ficha y Próximamente se cargan con `loadComponent` | Cada usuario descarga solo lo que usa. En la carga inicial quedan la cartelera, la privacidad y la sesión |
| La cartelera se trae entera de la vista `peliculas_cartelera` y se filtra en el navegador con `computed` | Son pocas películas: buscar y filtrar por género, clasificación, formato o idioma no vuelve a consultar la base. La vista respeta el RLS de quien consulta (`security_invoker`), así que nadie ve los borradores |
| `CarteleraStore` guarda los filtros | Al volver de la ficha, la cartelera sigue filtrada como estaba |
| La ficha recibe la película de un resolver (`peliculaResolver`) como input | La pantalla se muestra con los datos ya cargados, sin un estado intermedio vacío |
| Pósters en el bucket público `posters`, en `{id de la película}/{nombre normalizado}`, y la URL con `?v=` | Los ve cualquiera sin sesión y solo el admin los sube. La versión en la URL evita que el navegador muestre el póster anterior al reemplazarlo |
| Formularios del admin con aviso al salir (`pendingChangesGuard`) | Salir de una película, un género, una sala, sus butacas, una programación o los precios a medio cargar pide confirmación |
| Géneros elegidos con un control propio (`SelectorGeneros`, con `model()`) | Se enlaza al formulario con `[formField]` como cualquier otro campo |
| Las funciones se crean y se modifican solo con RPC (`programar_funciones` y `modificar_funcion`) | La tabla no tiene políticas de alta ni de cambio: la base elige la sala, calcula el fin con la limpieza y rechaza superposiciones aunque dos administradores programen al mismo tiempo |
| Vista previa de la programación con la regla pura `asignarSalas` | La pantalla anticipa la sala de cada fecha con el mismo criterio que la base (activa, con el formato, libre y de menor número) y después muestra lo que la base realmente creó. La regla tiene sus pruebas |
| Días, hora e inicio de la programación con chips (`SelectorDias`, `SelectorHora`, `SelectorFecha`) y la cantidad de semanas | Se programa sin calendarios ni relojes con scroll; los tres controles usan `model()` y se enlazan con `[formField]` |
| Horarios armados con `-03:00` fijo (`inicioBA`) y mostrados con `Intl` en la zona de Buenos Aires | Argentina no tiene horario de verano; así la hora no depende de la zona del navegador del admin |
| Butacas de cada sala editadas con una herramienta (tipo o fuera de servicio) y guardadas juntas | Se cambian muchas butacas con pocos toques; al guardar, se manda una actualización por grupo de butacas iguales |
| Precios con el pipe propio `precioArs` | Muestra pesos con `Intl` sin registrar el locale; la lógica está en `formatearPrecio`, que tiene pruebas |
| Los mensajes de error reconocen restricciones por nombre (`salas_numero_key`, `funciones_sin_solapamiento`) | El usuario lee "Ya existe una sala con ese número" en lugar del mensaje genérico de Postgres |
| Mapa de butacas con una sola tabla `ocupaciones` (retenidas y vendidas) escuchada por Realtime | Cada mapa abierto recibe los cambios de la tabla, descarta los de otras funciones en el callback y libera la butaca con el `DELETE`, que trae la clave `(funcion_id, butaca_id)` |
| Retención con la RPC `retener_butacas`, que recibe la selección completa en cada toque | La base suelta lo que ya no está elegido, mantiene un único vencimiento por usuario y rechaza todo si otra persona tomó una butaca. La pantalla muestra el cambio enseguida y lo deshace si la base lo rechaza |
| Un toque por vez: el mapa queda bloqueado mientras la base contesta | Dos pedidos de retención en vuelo podrían llegar en otro orden, y el segundo borraría lo que eligió el primero |
| Sesión anónima al tocar la primera butaca | La retención necesita saber de quién es cada butaca, y el visitante puede comprar sin crear una cuenta |
| Contador de 10 minutos con un `setInterval` que actualiza un signal solo mientras hay retención | Al llegar a cero libera las butacas en la base. Si el usuario cierra la pestaña, las borra el job de pg_cron que corre cada minuto. Como el vencimiento lo pone la base, el contador corrige la diferencia entre el reloj de la PC y el del servidor (`desfaseReloj`) |
| Directiva `appHighlightButaca` con la propiedad `host` | Pone las clases de tipo y estado, el `aria-label` (con el precio si es VIP), `aria-pressed` y `aria-disabled`. Las butacas ocupadas siguen en el orden del teclado, así se pueden recorrer y escuchar |
| Butacas distinguidas por forma e ícono además del color | Accesible más ancha y redondeada, VIP con estrella, elegida con ✓, vendida con ✕, retenida rayada y fuera de servicio con borde punteado. La leyenda está siempre visible |
| Validador propio `butacasContiguas` de Signal Forms, más `maxLength` con el máximo de la configuración | El motivo (pasillo, otra fila o lugares sin elegir) aparece apenas cambia la selección y **Continuar** queda deshabilitado. La base lo vuelve a validar al confirmar la compra |
| Checkout en pasos (función, butacas, candy, pago y comprobante) con un `@switch` sobre el paso de `CompraStore` | Un solo componente de ruta muestra cada paso con `IndicadorPasos`. Al salir, `ngOnDestroy` libera las butacas y vacía los dos stores, así la próxima compra arranca de cero |
| El contador de la retención vive en el checkout, arriba de los pasos | Sigue visible al pasar al candy y al pago; si llega a cero en cualquiera de ellos, libera las butacas y vuelve al mapa |
| Pago simulado dentro de la RPC `confirmar_compra`, detrás de un `PagoService` | La base aprueba el pago y registra la compra en la misma transacción, así nadie confirma sin pagar. Si la tarjeta se rechaza, libera las butacas. Para conectar una pasarela real se cambia solo `PagoService` |
| Los datos de la tarjeta viven solo en el formulario de pago y se borran apenas se envían | No pasan por ningún store, no se guardan en ninguna tabla y no aparecen en los errores |
| Validaciones de la tarjeta con reglas puras (`luhnValido`, `vencimientoValido`, `cvvValido`) | El formulario avisa antes de enviar, con la misma regla que usa la base. Las reglas tienen pruebas |
| Directiva estructural `*appRestrictEdad` en la ficha y en el paso de función | Muestra el bloque de compra solo si el cliente tiene la edad el día de la función. El motivo lo muestra la pantalla con la misma regla (`motivoRestriccionEdad`), así nunca se contradicen. El visitante sin cuenta declara su edad al pagar |
| Código del QR aleatorio de 10 caracteres, impreso como `ABCD-EFGH-JK` | El QR y el texto son el mismo código, que no contiene el id de la compra ni datos personales |
| QR y PDF generados en el navegador con `qrcode` y `jspdf`, cargados solo al usarlos (`import()`) | No pesan en la carga inicial y funcionan sin conexión |
| Comprobantes guardados en el dispositivo (`localStorage`, los últimos 20) y la pantalla `/comprobantes` en la carga inicial | El visitante sin cuenta también puede volver a ver su entrada, incluso sin conexión |
| Top 3 de la portada con la RPC `top_peliculas` | Ordena por entradas vendidas en los últimos 30 días y completa con los estrenos más recientes |
| Estado de cada compra (vigente, usada o cancelada) calculado en la vista `mis_compras` | La pantalla muestra el estado con un `@switch` y un estilo propio para cada uno, no solo con el color |
| Validación con la RPC `validar_qr`, en modo acceso o candy | La base bloquea la compra mientras valida, controla el horario, marca el sector como usado, lo audita y avisa al cliente, todo en una transacción. Dos empleados no pueden validar dos veces el mismo QR |
| Lectura del QR con `@zxing/browser` y la cámara trasera, encendida con un botón | Funciona en Chrome, Firefox y Safari. La cámara se pide recién cuando el empleado la necesita y se apaga al salir de la pantalla. La librería queda en el chunk de `/empleado`: nadie más la descarga |
| El escáner se pausa mientras valida y mientras el resultado está a la vista | La cámara sigue leyendo el mismo QR varias veces por segundo; sin la pausa, la segunda lectura mostraría "Ya usado" encima de "Válido" |
| Ingreso manual del código, con o sin guiones | Si la cámara falla o el QR está dañado, el empleado tipea el código impreso. `normalizarCodigo` lo deja como lo guarda la base y tiene pruebas |
| Resultado grande con `@switch`, ícono, texto y color | Válido en verde, fuera de horario y sin candy en ámbar, y el resto en rojo, con las butacas o el candy a entregar y el sector que falta validar |
| Bandeja de avisos en la cabecera, alimentada por Realtime (`NotificacionesStore`) | El cliente ve "Entrada validada" sin recargar. El store abre el canal cuando entra un usuario registrado y lo cierra cuando sale |
| PWA con `@angular/pwa`, manifest propio e íconos de Cineverso | Se instala desde Chrome en Android y en escritorio, y abre en su propia ventana |
| El service worker precachea solo lo inicial, `/comprobantes` y los chunks del QR y del PDF; las pantallas de las áreas se guardan recién al visitarlas (`installMode: lazy`, con `namedChunks`) | Un visitante no descarga el código del admin ni del empleado. La seguridad la dan el RLS y las RPC; esto es una mejora de rendimiento |
| Cartelera con estrategia `freshness` (red primero, 5 s de espera) y pósters con `performance` (7 días) | Sin conexión se ve la última cartelera guardada con sus pósters. El top 3 es una RPC que no se guarda: sin red, la portada se muestra sin él |
| Aviso de versión nueva con `SwUpdate`, un signal y un `@if` en `app.ts` | Cuando el service worker baja una versión nueva, aparece "Nueva versión: recargar" en lugar de seguir con la vieja sin avisar |
| Aviso del sistema con `registration.showNotification` del service worker, con permiso pedido desde **Activar avisos** en `/cuenta/notificaciones` | Con la app abierta, los recordatorios y la validación del QR también aparecen como notificación del dispositivo, y al tocarla se abre la compra. Se usa el service worker porque Chrome para Android no permite `new Notification()`. La bandeja sigue siendo el canal garantizado |
| Recordatorios de 24 h y 2 h generados por la base cada 5 minutos (pg_cron) | No dependen de que la app esté abierta: quedan en la bandeja y llegan por Realtime. Solo se avisa a las compras hechas antes del plazo, y la restricción `(compra_id, tipo)` impide repetirlos |
| Categorías, productos y combos con la lista y el formulario en la misma pantalla, y las fotos en el bucket público `productos` | Son formularios de pocos campos: una ruta aparte no suma nada. Las fotos van en `{id del producto}/` y en `combos/{id del combo}/`. Lo que ya se vendió no se puede borrar, porque la compra lo guarda: se marca como no disponible o inactivo y deja de aparecer en la compra |
| Productos de cada combo con un control propio (`SelectorProductos`, con `model()`) | Se enlaza con `[formField]` como `SelectorGeneros` y suma o resta unidades con la regla pura `cambiarCantidad`, que tiene pruebas |
| Paso de candy con los combos destacados arriba y los productos agrupados por categoría (`agruparPorCategoria`) | El cliente ve primero la oferta más conveniente y encuentra cada producto en su categoría. Se llega después de elegir las butacas, así que no hay candy sin entrada |
| Cada combo incluye una entrada 2D estándar; la entrada que cubre paga solo la diferencia de formato y el recargo VIP (`entradasConCombos`) | Es la misma cuenta que hace `confirmar_compra`: los combos cubren primero las butacas no VIP. Así el resumen muestra exactamente lo que cobra la base. El botón para sumar combos se apaga cuando hay uno por entrada, y la base lo vuelve a controlar |
| La compra guarda tres subtotales: entradas, candy y combos | Son las tres líneas del reporte de facturación. El precio fijo del combo no se reparte entre entrada y candy, y las diferencias de formato o VIP cuentan como entradas |
| Reportes con tres RPC (`reporte_facturacion`, `reporte_mas_vistas` y `reporte_productos`) que exigen ser administrador | La base suma y agrupa en una sola consulta, sin bajar cada compra al navegador. Las compras canceladas no cuentan. La facturación se agrupa por el día de la compra y las más vistas por el día de la función |
| Gráficos de barras con CSS y `@for` (`GraficoBarras`), con una tabla oculta para lectores de pantalla | Son dos gráficos simples: una librería sumaría peso sin aportar nada. La tabla repite los datos para quien no ve las barras |
| Exportación a PDF con `jspdf` y a Excel con `write-excel-file`, cargadas solo al exportar (`import()`) | El Excel es un `.xlsx` real, con los montos como números, que se abre en Excel y en LibreOffice. `xlsx` se descartó porque la versión de npm está desactualizada y tiene vulnerabilidades conocidas |
| Auditoría escrita solo por la base (triggers y RPC) y leída por el admin con filtros por usuario, acción y fechas | No depende de que la interfaz informe cada acción, y nadie puede escribirla desde la app. Los cambios muestran solo los campos que cambiaron |
| Push con la app cerrada: un trigger de la base llama con `pg_net` a una Edge Function (`enviar-push`) que envía con `web-push`; el navegador se suscribe con `SwPush` | Los avisos tienen que llegar aunque la app esté cerrada. Cifrar y firmar un push necesita un servidor: Postgres no puede y el navegador no debe tener la clave privada. La clave VAPID privada y el secreto que comparten la base y la función viven solo en Supabase; la pública está en los environments. La función no lee la base ni usa la clave `service_role`: el trigger le manda la notificación y los dispositivos del usuario |
| El aviso por Realtime y el push usan el mismo `tag` | Con la app abierta, uno reemplaza al otro y no aparece dos veces |
| Al cerrar sesión, el dispositivo se da de baja del push | Si otra persona entra en el mismo navegador, no recibe avisos de la cuenta anterior |
| Botón "Acceso para la corrección" en el login | Lo pidió la cátedra: completa los datos de una cuenta de administrador exclusiva para corregir, y se entra con "Ingresar" como siempre. Lo que puede hacer lo limita el RLS, igual que a cualquier admin, y cada cambio queda en la auditoría |
| Botón para mostrar u ocultar la clave (`VerClave`, con `model()`) en todos los campos de clave y en el código de seguridad | Permite revisar lo que se escribió antes de enviar, sobre todo en el celular |
| Aviso de presupuesto por componente en 6 kB (el error sigue en 8 kB) | Los estilos del menú, con la barra, los desplegables, el panel lateral y sus animaciones, pesan 5,8 kB. Angular no permite fijar el presupuesto para un solo componente |

## Limitaciones conocidas

- **Recuperación de contraseña:** el flujo está implementado completo. Pero el servidor de correo que trae Supabase solo envía mails a las direcciones del equipo del proyecto y tiene un límite bajo de envíos por hora. En la demo se prueba con una de esas direcciones. Para usarlo con cualquier dirección haría falta configurar un servidor SMTP propio.
- **Confirmación de email:** está desactivada por la misma razón. Las cuentas quedan activas al registrarse.
- **Avisos al dispositivo:** los recordatorios, la validación del QR y las alertas llegan siempre a la bandeja, y como notificación aunque la app esté cerrada si el usuario tocó "Activar avisos". En la computadora llegan mientras el navegador esté abierto, aunque no tenga la pestaña de Cineverso; si está cerrado, llegan al abrirlo. En iPhone hace falta instalar la app en la pantalla de inicio (iOS 16.4 o superior) y activar los avisos desde ahí.

## Usuarios de prueba

Hay una cuenta de prueba por rol: cliente, empleado y administrador. Los emails y las claves no están acá: se envían al docente por mail. Son cuentas creadas solo para la evaluación.

Para entrar como administrador sin escribir nada, el login tiene el botón **Acceso para la corrección**: completa los datos y se entra con **Ingresar**.

## Tarjetas de prueba

El pago es simulado: no se cobra nada y los datos de la tarjeta no se guardan. Sirve cualquier vencimiento futuro (por ejemplo `12/30`), cualquier código de 3 dígitos y cualquier nombre.

| Tarjeta | Resultado |
|---|---|
| `4242 4242 4242 4242` | Pago aprobado |
| `4000 0000 0000 0002` | Pago rechazado: la compra no se hace y las butacas se liberan |
| `4000 0000 0000 0119` | Error del procesador: la compra no se hace y las butacas se liberan |

Cualquier otro número que pase el control de Luhn se aprueba.
# practica
