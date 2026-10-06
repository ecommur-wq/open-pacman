# SPEC 01 — Cuatro fantasmas con personalidad y modo asustado

> **Status:** Aprobado
> **Depends on:** —
> **Date:** 2026-10-06
> **Objective:** Añadir los 4 fantasmas con comportamientos de movimiento propios (uno persigue a Pacman siempre) y el modo asustado activado por power pellets.

## Scope

**In:**

- 4 fantasmas con comportamiento propio cada uno: `hunter` (persigue siempre), `ambusher` (apunta a 4 celdas por delante de Pacman), `random` (ya existe) y `timid` (huye de Pacman).
- Los 4 arrancan dentro de la pen: `(12,14)`, `(13,14)`, `(14,14)`, `(15,14)`.
- 4 power pellets clásicos en las esquinas: `(1,3)`, `(26,3)`, `(1,23)`, `(26,23)`.
- Modo asustado: 5 segundos, los 4 invierten dirección al entrar y al salir, y quedan azules.
- Comer un fantasma asustado: +200 puntos y reaparece en su posición de la pen en modo normal.
- Color distinto por comportamiento en el render.

**Out of scope (for future specs):**

- Fases scatter/chase del original.
- Parpadeo blanco en los últimos 2 s del modo asustado.
- Escalera de puntos 200/400/800/1600 y animación de ojos al comer un fantasma.
- Velocidad reducida para fantasmas asustados.
- Vida extra por 10.000 puntos.

## Data model

```js
// maze.js — tile nuevo
const POWER_PELLET = 4; // MAZE_STR: nueva letra 'o' en las 4 esquinas

// maze.js — los 4 arrancan dentro de la pen (x 11–16, y 13–15)
const GHOST_STARTS = [
  { x: 13, y: 14, kind: 'hunter'  },
  { x: 14, y: 14, kind: 'ambusher'},
  { x: 12, y: 14, kind: 'random'  },
  { x: 15, y: 14, kind: 'timid'   },
];

// game.js — estado nuevo en el objeto que devuelve createGame()
frightened: {
  active: false,
  timer: 0, // frames restantes; 5 s ≈ 300 frames a 60 fps
}
// cada fantasma, además:
scared: false // el comido queda en false aunque siga el timer global
```

Conventions:

- Timer en frames, igual que las velocidades (celdas/frame). Consistente con el modelo actual.
- `dotsRemaining` cuenta tiles `2` **y** `4` — ganar exige comer los power pellets.

## Implementation plan

1. `maze.js`: añadir `'o'` en las 4 esquinas de `MAZE_STR`, mapearla a `4` en `parseTile`, reemplazar `GHOST_STARTS` por los 4 anteriores, actualizar el comentario de cabecera. Prueba manual: el laberinto dibuja 4 pellets grandes.
2. `render.js`: dibujar tile `4` como círculo grande pulsante. Prueba: los 4 pellets se ven y desaparecen al comerlos.
3. `game.js`: contar tiles `4` en `dotsRemaining`. Al comer un tile `4`: `+50` puntos, activar `frightened` (timer 300) e invertir `dir` de los 4 fantasmas. Prueba: comer un pellet suma 50 y los 4 giran.
4. `game.js`: restar `frightened.timer` en `update()`; al llegar a 0, desactivar e invertir `dir` de los 4 de nuevo. Prueba: a los 5 s vuelven a la normalidad.
5. `game.js`: ramas nuevas en `decideGhost()` — `ambusher` elige la dirección más cercana a la celda 4 delante de Pacman; `timid` elige la que más se aleja. Prueba: los 4 muestran patrones distintos en una partida.
6. `game.js`: en la colisión, si el fantasma tiene `scared`, comerlo (`+200`, reposicionar en su `GHOST_STARTS`, `scared = false`) y no restar vida; si no, restar vida como hasta ahora. Prueba: fantasma azul comido no quita vida.
7. `render.js`: mapa de color por `kind` en lugar de por índice + cuerpo azul cuando `scared`. Prueba: cada fantasma tiene su color y se tiñe de azul.
8. Verificar los criterios de aceptación completos.

## Acceptance criteria

- [ ] El juego carga sin errores en la consola.
- [ ] Hay 4 fantasmas y los 4 salen de la pen por la puerta.
- [ ] `hunter` siempre reduce su distancia a Pacman en cada intersección.
- [ ] `timid` siempre incrementa su distancia a Pacman en cada intersección.
- [ ] `ambusher` toma decisiones apuntando a la celda 4 por delante de Pacman.
- [ ] `random` no muestra patrón direccional.
- [ ] Los 4 fantasmas tienen colores distintos.
- [ ] Comer un power pellet suma exactamente 50 puntos y desaparece del mapa.
- [ ] Al comer un power pellet, los 4 invierten su dirección y se pintan azules.
- [ ] A los 5 segundos exactos de juego vuelven a su color normal.
- [ ] Comer un fantasma asustado suma exactamente 200 puntos y no resta vida.
- [ ] El fantasma comido reaparece en su celda de la pen sin modo asustado.
- [ ] Chocar con un fantasma no asustado resta 1 vida (comportamiento previo intacto).
- [ ] El juego solo se gana al comer todos los dots **y** los 4 power pellets.

## Decisions

- **Yes:** 4 comportamientos semi-clásicos (hunter/ambusher/random/timid). Distintos y simples.
- **No:** lógica completa de Inky (usa 2 fantasmas como referencia). Complejidad sin beneficio proporcional.
- **Yes:** los 4 dentro de la pen. Arranque uniforme y sin caso especial para el de fuera.
- **No:** fases scatter/chase. El requisito pide perseguir constantemente.
- **Yes:** modo asustado en esta spec. El usuario lo incluyó.
- **Yes:** 4 power pellets en esquinas. Fiel al original y son celdas válidas ya existentes.
- **No:** que cualquier dot active el miedo. Rompería la economía de puntos.
- **Yes:** 5 s fijos, sin parpadeo. El flash queda para otra spec.
- **Yes:** 200 puntos fijos + reaparición en la pen. Sin escalera ni animación de ojos.
- **Yes:** invertir dirección al entrar y al salir del modo. Regla clásica, 2 líneas de código.
- **Yes:** sin cambio de velocidad para fantasmas asustados (confirmado por el usuario).
- **Yes:** power pellet = 50 puntos (valor clásico, confirmado por el usuario).
- **Yes:** timer global + flag `scared` por fantasma. Evita que el fantasma recién comido sea re-comible.
- **Yes:** color por `kind` con mapa en `render.js`, no por índice del array.

## Risks

| Risk | Mitigation |
| --- | --- |
| El timer en frames solo dura 5 s si el rAF va a 60 fps | Aceptado: igual que las velocidades (celdas/frame). Todo el juego ya depende de esto. |
| Invertir `dir` a mitad de celda podría cruzar un muro | Solo se invierte hacia la celda de la que el fantasma vino, que es transitables por definición. |

## What is **not** in this spec

- Fases scatter/chase.
- Parpadeo del modo asustado.
- Escalera de puntos y ojos de fantasma comido.
- Velocidad reducida asustados.
