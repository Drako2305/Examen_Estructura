# Taller con Frontend · Caso de estudio 3 · Integrador

## Despacho de mensajería urbana en moto: ruta y maletero

Lista + Cola + Pila · Con interfaz gráfica · TypeScript + Vite

## 1. Contexto del problema

Una empresa de mensajería urbana recibe sobres y paquetes pequeños durante todo el día y los reparte con motos. Los envíos vienen en el almacén de la moto, un lugar rígido donde las paquetes quedan ordenados por prioridad y por la distancia estimada al destino.

El flujo del sistema es el siguiente:

- La moto recoge los envíos pendientes del almacén.
- La ruta se construye con una lista doblemente enlazada.
- La moto puede recorrer la ruta en ambos sentidos para ir entregando los paquetes.
- El maletero de la moto representa la zona de carga: solo admite una cantidad limitada de paquetes.
- El sistema valida si la carga del maletero es correcta antes de entregar.
- Cuando una entrega falla, la carga se reintegra o se reordena según la lógica de negocio.
- Al cerrar la ruta, se contabilizan los envíos devueltos y los reintentos.

El objetivo del proyecto es modelar la lógica de la operación con estructuras de datos correctas y una interfaz gráfica simple pero funcional.

## 2. Estructuras de datos exigidas

Este caso integrador trae tres estructuras son obligatorias y cada una cumple un papel dentro del sistema:

| Estructura | Qué modela | Por qué se usa |
| --- | --- | --- |
| Lista doblemente enlazada | La ruta de entregas | Permite recorrer la ruta en ambos sentidos, insertar, mover y eliminar entregas sin perder el orden. |
| Cola | Los envíos pendientes y los reintentos | Cumple la lógica FIFO: primero en entrar, primero en salir. |
| Pila | El maletero de la moto | Permite apilar la carga y manejar la validación del último paquete cargado. |

## 3. Descripción del sistema

### Atributos principales

| Atributo | Tipo | Descripción |
| --- | --- | --- |
| envio.guia | cadena | Número de guía, por ejemplo G-1045. |
| envio.destino | cadena | Dirección o barrio de entrega. |
| envio.tipo | cadena | SOBRE o PAQUETE. |
| envio.recibido | entero | Momento de la jornada en que se recibió el envío. |
| envio.intentos | entero | Número de intentos de entrega realizados. |
| parada.estado | cadena | PENDIENTE, ENTREGADO o FALLIDA. |
| parada.envio | objeto | Envío asociado a la parada. |
| carga | pila | Carga del maletero. |
| ruta | lista enlazada | Ruta actual del recorrido. |
| pendientes | cola | Envios no procesados. |
| reintentos | cola | Envios devueltos para nueva entrega. |
| parada.estado | cadena | Estado del paquete dentro de la ruta. |

## 4. Reglas de negocio

| ID | Regla |
| --- | --- |
| R1 | Los envíos entran a la cola de pendientes en el orden en que se reciben. No se puede sacar un envío del medio. |
| R2 | Armar una ruta consiste en tomar hasta N envíos del frente de la cola (N = capacidad del maletero) y convertirlos en paradas de la ruta, en ese mismo orden. El despacho puede luego mover o eliminar paradas. |
| R3 | Carga del maletero: se recibe la ruta desde la última parada hacia la primera, y se apila en el maletero. Al terminar, se aplica la validación de carga. |
| R4 | Validación de carga del maletero: el sistema valida que el maletero esté exactamente lleno con la ruta actual. Si no, no puede cargar o entregar. |
| R5 | Toda modificación de la ruta después de cargar (mover, insertar o eliminar una parada) invalida la carga. |
| R6 | Entrega: se entrega el paquete del tope, se marca como ENTREGADA y el apuntador avanza a la siguiente parada. |
| R7 | Entrega fallida: el sistema marca la entrega como FALLIDA, mueve el paquete al final de la ruta y reubica la carga. |
| R8 | Un envío con 2 intentos fallidos sale de la ruta actual y entra a la cola de reintentos. |
| R9 | La ruta no puede tener más paradas que la capacidad del maletero, y en vivo no puede estar en dos rutas a la vez. |

## 5. Requisitos funcionales

| ID | Operación | Comportamiento esperado |
| --- | --- | --- |
| RF-01 | recibirEnvio(guia, destino, tipo, minuto) | Encola el envío en pendientes. Valida guía duplicada y capacidad de la cola. |
| RF-02 | armarRuta() | Aplica R2: toma hasta N envíos del frente de la cola y construye la ruta. |
| RF-03 | moverParada(guia, posicion) | Reorganiza la ruta. Aplica R5 invalidando la carga si el maletero estaba cargado. |
| RF-04 | recorrerRuta(sentido) | Recorre la ruta en el sentido indicado y devuelve la secuencia asociada. |
| RF-05 | cargarMaletero() | Aplica R3. Si el maletero no está vacío, debe rechazar la operación. |
| RF-06 | validarCarga() | Aplica R4 y devuelve el resultado del estado de la carga. |
| RF-07 | entregar() | Aplica R6. |
| RF-08 | entregaFallida() | Aplica R7. |
| RF-09 | cerrarRuta() | Aplica R8 y genera la cola de reintentos. |
| RF-10 | reporte() | Calcula y publica las métricas del numeral 7. |

## 6. Restricciones de implementación (lógica)

- Debe implementar su propia lista doblemente enlazada, su propia cola y su propia pila, con las restricciones de lenguaje del anexo que le corresponda.
- Las operaciones de cola y pila deben ser O(1) en la inserción y extracción, salvo la validación de la estructura y la gestión de memoria.
- La validación del maletero debe asegurarse en cada operación. Si se desea cargar y no es posible, se debe rechazar.
- La reorganización de la ruta debe conservar el orden relativo de los paquetes que no se mueven.
- La lógica de las estructuras no puede estar escrita dentro del código de la interfaz.
- El punto clave del caso: la ruta se recorre de la primera parada a la última y el maletero del tipo de fondo. La interfaz debe hacer visible esta relación, no esconderla.

## 7. Requisitos de frontend (obligatorios)

Este taller no se entrega como programa de consola. La lógica de estructuras de datos debe quedar detrás de una interfaz gráfica que permita operar el sistema y visualizar el estado de la ruta y la pila.

| ID | Requerimiento | Criterio de aceptación |
| --- | --- | --- |
| RFE-01 | Separación de capas | La interfaz no puede leer ni modificar los datos internos de las estructuras: solo invoca los métodos públicos del TDA y mostrar el estado. |
| RFE-02 | Pantallas mínimas | La aplicación debe tener una pantalla de 1366x768 sin desplazamiento horizontal, y los estados vacío y lleno se distinguen a simple vista. |
| RFE-03 | Pila visible | El maletero se dibuja en vertical con el topo arriba, al lado de la ruta, y un indicador de carga en verde o rojo según el estado. |
| RFE-04 | Cola visible | La cola de pendientes y la de reintentos se dibujan en horizontal con el frente a la izquierda. |
| RFE-05 | Ruta visible | La ruta actual se dibuja en un orden de izquierda a derecha. |
| RFE-06 | Operación por interfaz | Toda operación del numeral de requerimientos funcionales debe poder ejecutarse desde un control de la interfaz (botón, formulario o menú). |
| RFE-07 | Error en pantalla | Las validaciones fallidas muestran un mensaje visible en la interfaz, citando la regla R1, R2, etc. |
| RFE-08 | Biblioteca | Permite el historial cronológico de las operaciones ejecutadas y su resultado. |
| RFE-09 | Panel de métricas | Las métricas del numeral 7 se muestran en pantalla y se actualizan después de cada operación. |
| RFE-10 | Datos de ejemplo | Botón o acción que cargue datos de ejemplo para probar el caso de uso completo. |

## 8. Pantallas exigidas

### Pantalla de carga

- Bandeja de envíos: la cola de pendientes y la cola de reintentos, con su frente marcado.
- Editor de ruta: la ruta como secuencia de paradas y su estado.
- Maletero: la pila dibujada en vertical con el paquete más reciente arriba.
- Cierre de ruta: resultado del cierre y estadísticas.

### Pantalla de operación

| Pantalla | Contenido mínimo |
| --- | --- |
| 1. Bandeja de envíos | La cola de pendientes y la cola de reintentos, con su frente marcado. Formulario para recibir un nuevo envío y botón para armar la ruta. |
| 2. Editor de ruta | La ruta como secuencia de paradas numeradas, con la posibilidad de mover o eliminar paradas. |
| 3. Maletero | La pila dibujada en vertical con el topo arriba, al lado de la ruta y un indicador de carga en verde o rojo. |
| 4. Cierre y reportes | Resultado del cierre de ruta, panel de métricas y bitácora cronológica. |

Nota: el sistema debe estar funcional incluso si el tamaño de la pantalla no es exactamente 1366x768, siempre que la interfaz se adapte sin romper el diseño.

## 9. Métricas del reporte

- Envíos recibidos, asignados a ruta, entregados y devueltos a la cola al cerrar.
- Entregas fallidas y movimientos de reorganización del maletero que costaron (métrica de eficiencia).
- Número de veces que la carga fue invalidada por modificar la ruta (R5) y recargas realizadas.
- Tiempo que cada envío esperó en la cola de pendientes, con promedio y máximo.
- Envíos que llegaron a 2 intentos y pasaron a la cola de reintentos (R8).

## 10. Ejecución del proyecto

```bash
npm install
npm run dev
```

Para validar el build final:

```bash
npm run build
```

## 11. Objetivo de la solución

La aplicación debe demostrar correctamente el uso de:

- una lista doblemente enlazada para la ruta;
- una cola para los envíos pendientes y reintentos;
- una pila para el maletero;
- validación de reglas de negocio;
- una interfaz gráfica funcional y legible;
- métricas de operación del sistema.

## 12. Observaciones de calidad

- El proyecto debe ser escalable en la lógica: no conviene mezclar la estructura de datos con el render de la UI.
- La interfaz debe ser clara, sin elementos ocultos ni estados ambiguos.
- El nombre de clases, métodos y mensajes debe reflejar el dominio del problema, no una solución genérica.
- La solución debe ser mantenible, reutilizable y fácil de ampliar con nuevos requisitos.

## 13. Resultado esperado

El sistema debe permitir:

- recibir envíos;
- construir rutas;
- mover, insertar y eliminar paradas;
- cargar y validar el maletero;
- entregar y fallar entregas;
- cerrar la ruta;
- visualizar métricas y reportes.

Si la aplicación realiza estas operaciones con una separación clara entre lógica y interfaz, y cumple las reglas de negocio del caso, entonces responde correctamente al enunciado del taller.

-----

## Estado del repositorio

Este proyecto está pensado para resolver el caso de estudio con TypeScript y Vite, usando una arquitectura simple y un formato de interfaz que se ejecute en navegador.

La intención final es que la aplicación muestre no solo la lógica de estructuras, sino también la interacción con la capa visual siguiendo la rúbrica del examen.

------

## Nota de corrección

La versión original del README estaba desalineada con el enunciado real del examen: era genérica, estaba redactada en inglés y no reflejaba las reglas, pantallas y métricas exigidas. Esta versión reescribe la documentación para que responda exactamente al caso de estudio entregado por el profesor.
