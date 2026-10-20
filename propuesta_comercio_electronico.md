# Comercio electrónico: qué queremos hacer y con qué datos

Documento para discutir con el equipo antes de diseñar la solución técnica y antes de escribir código.

## Qué estamos intentando hacer

El proyecto del curso pide un sistema que reciba un flujo continuo de eventos de una tienda en línea y, a medida que llegan, pueda:

- atender primero lo urgente (un pago o una alerta antes que una simple visita);
- consultar rápido un producto, su precio y su disponibilidad;
- sugerir productos parecidos o que suelen comprarse juntos;
- marcar actividad sospechosa sin bloquear una compra solo por una sospecha;
- mostrar qué productos influyen en el catálogo y qué grupos de productos se compran juntos.

La salida que tendría que verse al final es concreta: un autocompletado de búsqueda, un ranking de productos, una recomendación y un panel de ventas y alertas.

Elegimos el escenario de **comercio electrónico** porque el comportamiento de un cliente ya viene ordenado en el tiempo (miró, agregó al carrito, compró) y porque una compra conjunta es fácil de explicar: dos productos que salen en la misma orden.

## El dataset que usaríamos

**Sí: es data real.** No es un dataset sintético ni una encuesta. Es el registro de lo que hicieron los visitantes de una tienda en línea grande, de varias categorías, entre octubre de 2019 y abril de 2020.

La tienda no publica su nombre. Los usuarios están anonimizados: cada persona tiene un número fijo y cada visita tiene un identificador de sesión. No hay nombres, correos ni direcciones.


|                  |                                                                                                                                                                                            |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Nombre           | eCommerce behavior data from multi category store                                                                                                                                          |
| Autor en Kaggle  | Michael Kechinov                                                                                                                                                                           |
| Origen           | Proyecto Open CDP, con agradecimiento a REES46 Marketing Platform                                                                                                                          |
| Ficha y descarga | [https://www.kaggle.com/datasets/mkechinov/ecommerce-behavior-data-from-multi-category-store](https://www.kaggle.com/datasets/mkechinov/ecommerce-behavior-data-from-multi-category-store) |
| Periodo          | Octubre 2019 – abril 2020 (7 meses)                                                                                                                                                        |
| Volumen total    | 285 millones de eventos                                                                                                                                                                    |
| Licencia de uso  | Libre, citando la página de Kaggle y a REES46                                                                                                                                              |


No descargamos el dataset. Noviembre de 2019, por sí solo, pesa unos 9 GB en CSV (2,69 GB comprimido) y tiene **67.501.979 eventos**. El enunciado pide trabajar con 10 millones, así que más adelante bastaría un recorte de ese mes, no los siete archivos.

### Qué es una fila

Cada fila es **una acción de una persona sobre un producto**, en un momento dado. No es una orden armada ni una ficha de producto con nombre y descripción.

El propio autor lo lee así: el usuario `user_id`, durante la sesión `user_session`, hizo `event_type` sobre el producto `product_id` de la marca `brand` y la categoría `category_code`, al precio `price`, en el instante `event_time`.


| Columna         | Qué guarda                                                                                                    |
| --------------- | ------------------------------------------------------------------------------------------------------------- |
| `event_time`    | Fecha y hora del evento, en UTC                                                                               |
| `event_type`    | Qué hizo la persona                                                                                           |
| `product_id`    | Identificador del producto                                                                                    |
| `category_id`   | Identificador numérico de la categoría                                                                        |
| `category_code` | Categoría legible, en forma de ruta (`electronics.smartphone`). A veces viene vacía, sobre todo en accesorios |
| `brand`         | Marca, en minúsculas. También puede venir vacía                                                               |
| `price`         | Precio del producto en ese momento. El archivo no dice la moneda                                              |
| `user_id`       | Identificador permanente de la persona                                                                        |
| `user_session`  | Identificador de la visita. Cambia cuando la persona vuelve después de una pausa larga                        |


Los tipos de evento documentados son:

- `view`: vio el producto
- `cart`: lo agregó al carrito
- `remove_from_cart`: lo sacó del carrito
- `purchase`: lo compró

En el archivo de noviembre de 2019 el conteo público es solo de tres tipos, y suma exactamente las 67.501.979 filas:


| Tipo       | Eventos en noviembre 2019 | Parte del mes |
| ---------- | ------------------------- | ------------- |
| `view`     | 63.556.110                | 94,2 %        |
| `cart`     | 3.028.930                 | 4,5 %         |
| `purchase` | 916.939                   | 1,4 %         |


Casi todo el flujo es gente mirando. Las compras son la minoría, que es lo esperable en una tienda real. En la ficha de Kaggle de ese mismo mes, la categoría viene vacía en alrededor de un tercio de las filas y la marca en alrededor de un 14 %. La categoría con más eventos es `electronics.smartphone` y la marca más frecuente es `samsung`.

El autor aclara un punto que nos sirve: **una sesión puede tener varias compras, y eso cuenta como una sola orden.** No existe una columna `order_id`.

### Muestra real

Estas filas son las primeras del 1 de noviembre de 2019, leídas del mismo archivo de noviembre (sin bajar el CSV). En esos primeros segundos solo hay vistas: a esa hora el flujo es, sobre todo, gente navegando.


| Hora (UTC)          | Acción | Producto | Categoría                     | Marca   | Precio | Usuario   |
| ------------------- | ------ | -------- | ----------------------------- | ------- | ------ | --------- |
| 2019-11-01 00:00:00 | view   | 1003461  | electronics.smartphone        | xiaomi  | 489,07 | 520088904 |
| 2019-11-01 00:00:00 | view   | 5000088  | appliances.sewing_machine     | janome  | 293,65 | 530496790 |
| 2019-11-01 00:00:01 | view   | 17302664 | (vacía)                       | creed   | 28,31  | 561587266 |
| 2019-11-01 00:00:01 | view   | 3601530  | appliances.kitchen.washer     | lg      | 712,87 | 518085591 |
| 2019-11-01 00:00:01 | view   | 1004775  | electronics.smartphone        | xiaomi  | 183,27 | 558856683 |
| 2019-11-01 00:00:01 | view   | 1306894  | computers.notebook            | hp      | 360,09 | 520772685 |
| 2019-11-01 00:00:02 | view   | 1004258  | electronics.smartphone        | apple   | 732,07 | 532647354 |
| 2019-11-01 00:00:03 | view   | 17200570 | furniture.living_room.sofa    | (vacía) | 437,33 | 518780843 |
| 2019-11-01 00:00:05 | view   | 4600658  | appliances.kitchen.dishwasher | samsung | 411,83 | 526595547 |


La primera compra del archivo aparece poco después, en la fila 168:


| Hora (UTC)          | Acción   | Producto | Categoría              | Marca  | Precio | Usuario   | Sesión                               |
| ------------------- | -------- | -------- | ---------------------- | ------ | ------ | --------- | ------------------------------------ |
| 2019-11-01 00:01:04 | purchase | 1005161  | electronics.smartphone | xiaomi | 211,92 | 513351129 | e6b7ce9b-1938-4e20-976c-8b4163aea11d |




### Un cliente, seguido en el tiempo

El mismo usuario `513351129` deja un recorrido que se puede leer sin conocer el resto del archivo. Es data real del mes, no un ejemplo inventado.

**1 de noviembre, una sesión**


| Hora (UTC) | Acción   | Producto | Qué es              | Precio |
| ---------- | -------- | -------- | ------------------- | ------ |
| 00:00:17   | view     | 1005161  | smartphone Xiaomi   | 211,92 |
| 00:01:04   | purchase | 1005161  | el mismo smartphone | 211,92 |


Miró el teléfono y lo compró 47 segundos después, en la misma visita.

**25 de noviembre, otra sesión**


| Hora (UTC) | Acción   | Producto | Qué es                    | Precio |
| ---------- | -------- | -------- | ------------------------- | ------ |
| 09:44:42   | view     | 5100562  | electronics.clocks, Apple | 334,60 |
| 09:44:57   | cart     | 5100562  | lo agrega al carrito      | 334,60 |
| 09:45:21   | view     | 5100562  | vuelve a verlo            | 334,60 |
| 09:45:23   | cart     | 5100562  | lo agrega otra vez        | 334,60 |
| 09:46:11   | purchase | 5100562  | lo compra                 | 334,60 |


Ahí está el embudo que el proyecto quiere procesar: ver, agregar al carrito y comprar. También se ve que la misma acción puede repetirse (dos `cart` del mismo producto en la misma visita), así que limpiar duplicados no es un caso artificial.

## Qué podemos hacer con esta data, y qué no trae

Con lo que sí está en el archivo se puede armar el núcleo del escenario:

- el flujo de visitas, carritos y compras, con hora real;
- el catálogo implícito (producto, categoría, marca y precio);
- productos vistos o comprados por la misma persona;
- productos comprados en la misma sesión, que el autor trata como una orden;
- rankings de marcas, categorías y productos a partir del comportamiento.

Lo que el enunciado también pide y **este archivo no trae**:

- si la orden es express, estándar o programada;
- medio de pago, dispositivo o IP;
- una etiqueta de fraude;
- devoluciones ni reseñas;
- el nombre del producto (por eso el autocompletado tendría que apoyarse en categoría y marca, por ejemplo `electronics.smartphone samsung`, salvo que más adelante consigamos nombres).

Eso no invalida el dataset. El enunciado dice que ningún dataset público trae los 10 millones de eventos con prioridades, fraude y duplicados, y recomienda usar la fuente real como base y generar esa parte. La propuesta, si el equipo está de acuerdo, es:

1. usar noviembre de 2019 (o un recorte de 10 millones) como comportamiento real;
2. inventar solo lo que no existe: prioridad de la orden, dispositivo, medio de pago y una fracción pequeña de fraude, apoyada en los usuarios y productos reales.



## Qué queda para la siguiente conversación

Este documento no fija algoritmos ni estructuras. Cuando el equipo confirme el dataset, el siguiente paso es decidir cómo recorre un evento el sistema (ingesta, prioridad, consulta, recomendación, alerta y grafo de productos comprados juntos) y recién ahí implementar, en notebooks.

## Para citar

Kechinov, M. *eCommerce behavior data from multi category store*. Kaggle. [https://www.kaggle.com/datasets/mkechinov/ecommerce-behavior-data-from-multi-category-store](https://www.kaggle.com/datasets/mkechinov/ecommerce-behavior-data-from-multi-category-store)