# SPEC 01 — Personalidades de fantasmas (Blinky, Pinky, Inky, Clyde)

> **Estado:** Implementado
> **Depende de:** — (primer spec del proyecto)
> **Fecha:** 2026-09-21
> **Objetivo:** dar a los cuatro fantasmas del juego una personalidad propia y distinta, siendo Blinky un perseguidor agresivo de Pac-Man, y liberarlos del pen de forma escalonada como el original.

## Scope

**In:**

- 4 fantasmas con el set clásico de personalidades: Blinky (perseguidor directo), Pinky (emboscada 4 celdas delante de Pac-Man), Inky (ataque con ancla en Blinky) y Clyde (cobarde frente a Pac-Man).
- Salida escalonada del pen estilo original: Blinky a los 0s, Pinky 7s, Inky 14s, Clyde 21s (a 60 fps: 0/420/840/1260 frames).
- Colores clásicos: rojo, rosa, cian y naranja, asignados por personalidad.
- Reset del escalonado al perder una vida.
- Efecto visual de espera en el pen (bobbing) para los fantasmas aún no liberados.

**Out of scope (para futuros specs):**

- Modo scatter por fases (retirada periódica a esquinas).
- Píldoras de poder y modo asustado (azul).
- Coreografía clásica de salida del pen (cada fantasma sale por la puerta existente, sin patrones distintos).
- Colisiones entre fantasmas.

## Data model

Los fantasmas dejan de usar `kind: 'hunter' | 'random'` y pasan a una configuración centralizada en `maze.js`:

```js
// maze.js
const GHOST_STARTS = [
  { x: 13, y: 14, kind: 'blinky', releaseFrame: 0 },
  { x: 14, y: 14, kind: 'pinky',  releaseFrame: 420 },
  { x: 11, y: 14, kind: 'inky',   releaseFrame: 840 },
  { x: 16, y: 14, kind: 'clyde',  releaseFrame: 1260 },
];
```

En `game.js` se añade un reloj de liberación derivado (no se guarda `released` como flag, se deduce):

```js
// game.js
game.elapsedFrames = 0;          // se incrementa 1 por update mientras se juega
// fantasma: { x, y, dir, speed, kind, releaseFrame }
// liberado si  game.elapsedFrames >= g.releaseFrame
```

Objetivos por personalidad (celda objetivo a la que el fantasma minimiza su distancia Manhattan entre las opciones válidas):

- Blinky → celda de Pac-Man.
- Pinky → celda de Pac-Man + `DIRS[pacman.dir] * 4`.
- Inky → `2 * celda(Pac-Man) - celda(Blinky)` (doble vector Blinky→Pac-Man; busca a Blinky por `kind`).
- Clyde → celda de Pac-Man si distancia ≥ 8 celdas; si no, esquina inferior izquierda `{ x: 0, y: 30 }`.

## Implementation plan

Cada paso deja el juego funcional y abrefiable en `src/index.html`.

1. **Configurar los 4 fantasmas en `maze.js`.** Sustituir `GHOST_STARTS` por las 4 entradas del bloque anterior (quedan dentro del interior de la pen, fila 14). Prueba manual: en consola `GHOST_STARTS.length === 4`.
2. **Reloj de liberación en `game.js`.** Añadir `game.elapsedFrames`, incrementarlo en `update`, mover solo fantasmas con `elapsedFrames >= g.releaseFrame` y reiniciar `elapsedFrames = 0` en `resetPositions`. Prueba manual: Pinky/Inky/Clyde permanecen unos segundos en la pen.
3. **`decideGhost` por personalidad en `game.js`.** Refactor a una función auxiliar `pickDirToward( game, g, tx, ty )` (basada en el hunter actual) y 4 ramas según `kind`. Eliminar `'hunter'` y `'random'`. Prueba manual: observar que Blinky persigue directo, Pinky corta camino delante de Pac-Man, Inky usa a Blinky y Clyde huye cuando se acerca.
4. **Colores y espera en `render.js`.** Sustituir `GHOST_COLORS` por un mapa por `kind` (blinky `#ff0000`, pinky `#ffb8ff`, inky `#00ffff`, clyde `#ffb852`) y dibujar bobbing vertical en fantasmas no liberados. Prueba manual: 4 colores clásicos y movimiento de espera en el pen.

No se añade ningún archivo JS nuevo: `index.html` no cambia.

## Acceptance criteria

- [ ] Abrir `src/index.html` no muestra errores en consola.
- [ ] Se ven 4 fantasmas con los colores clásicos: rojo, rosa, cian y naranja.
- [ ] Blinky sale del pen a los 0s y persigue a Pac-Man eligiendo en cada cruce la dirección que minimiza la distancia Manhattan a su celda.
- [ ] Pinky no sale hasta ~7s y su objetivo va 4 celdas delante del Pac-Man según su dirección de movimiento.
- [ ] Inky usa la celda de Blinky como ancla para calcular su objetivo.
- [ ] Clyde persigue a Pac-Man a ≥8 celdas y se retira a la esquina inferior izquierda (0,30) a menos de 8 celdas.
- [ ] Los fantasmas no liberados quedan en la pen oscilando suavemente arriba/abajo.
- [ ] Al perder una vida, los 4 vuelven al pen y el escalonado reinicia (0/7/14/21 s).
- [ ] El comportamiento de cada fantasma es visiblemente distinto de los otros tres.
- [ ] No queda código muerto: no se referencia `'hunter'`, `'random'` ni `GHOST_COLORS`.

## Decisions

- **Sí:** set clásico de personalidades (fiel al juego original y satisface "cada uno se comporta distinto").
- **Sí:** liberación escalonada estilo original (decisión explícita del usuario).
- **Sí:** contador de frames para el reloj (coherente con velocidades por frame existentes, p. ej. `PACMAN_SPEED`).
- **Sí:** distancia Manhattan para elegir la mejor dirección (reutiliza la heurística del hunter actual).
- **Sí:** bobbing en la pen como feedback visual de "aún no liberado".
- **Sí:** colores mapeados por `kind` en `render.js` (el orden del array deja de ser significativo).
- **No:** corrección del "bug" de Pinky del original (arriba y luego izquierda). No aporta al objetivo.
- **No:** modo scatter por fases; la retirada de Clyde a su esquina es su personalidad, no un modo global.
- **No:** píldoras de poder / modo azul; se añadirán junto a la píldora en otro spec.

## Risks

| Riesgo | Mitigación |
| --- | --- |
| El contador de frames asume ~60 fps (rAF real puede variar) y el escalonado en segundos deriva | Aceptable: el resto de velocidades ya asume ~60 fps. Si se exige exactitud, migrar a cronómetro con `Date.now()` en un spec futuro |
| Inky depende de la posición de Blinky | Se busca por `kind`, no por índice; funciona aunque se reordene el array |

## What is **not** in this spec

- Modo scatter por fases.
- Píldoras de poder y fantasmas asustados (azules) que se comen.
- Coreografía clásica de salida del pen.
- Colisiones fantasma-fantasma.

Cada uno de esos, si llega, va en su propio spec.