# Guion de la demostración (defensa)

El sistema corre contra el **servidor del curso** (base `db_grupo12sa`, buzón `grupo12sa@tecnoweb.org.bo`) y se opera con **3 actores**,
uno por departamento de Canal 11:

| Actor | Rol | Correo (remitente) | Qué muestra |
|---|---|---|---|
| Programación | `ADMINISTRADOR` | `jtorricoavila@gmail.com` | usuarios, programas, materiales, parrilla, pautas, spots, auditoría, reportes |
| Prensa | `ENCARGADO` | `jtorricoavila+prensa@gmail.com` | aprueba o rechaza materiales y ordena el noticiero |
| Producción | `PRODUCCION` | `jtorricoavila+produccion@gmail.com` | registra lo que salió al aire |

Los comandos de cada paso son los de [`pruebas/proceso_3_actores.txt`](../pruebas/proceso_3_actores.txt) (probado completo: 58 comandos OK).

## Preparación (una sola vez)

1. **Alias en Gmail:** Configuración → Cuentas e importación → **Enviar como** → Añadir otra dirección:
   `jtorricoavila+prensa@gmail.com` y `jtorricoavila+produccion@gmail.com` (se confirman con el código que llega a la misma bandeja).
   Al escribir, se elige el remitente en «De:».
2. **Registro del Administrador:** si `jtorricoavila@gmail.com` no está en la base del servidor, el administrador actual
   (`larzekaobnc@gmail.com`) envía `insUsu["9876543","J","Torrico","jtorricoavila@gmail.com","","ADMINISTRADOR"]`.
3. **Sistema corriendo desde la red de la facultad** (el SMTP del curso solo responde a Gmail desde esa red):

```bat
scripts\compilar.bat
scripts\ejecutar.bat --instalar
scripts\ejecutar.bat
```

Cada comando es un correo nuevo a `grupo12sa@tecnoweb.org.bo` con el comando en el **asunto** (cuerpo vacío).

## Horarios (las reglas comparan contra la hora real del canal)

| Pasos | Cuándo |
|---|---|
| 1 a 7 | el día de la parrilla, **antes de las 20:00** (la escaleta solo se cambia antes de que empiece el bloque) |
| `camBlo` (paso 7) | antes de las **21:05** (el bloque de la película no tiene que haber empezado) |
| 8 (emisión) | **después de las 23:35** o al día siguiente (solo se registra lo que ya salió al aire) |

Si la defensa es otro día, cambiar la fecha `2026-11-20` del archivo por la fecha de la defensa (la parrilla no se puede crear para un día pasado).

## 1. Usuarios y roles (CU1) — Administrador

`help`, alta de Prensa y Producción con `insUsu`, `lisUsu["*"]`. Mostrar que `help` responde distinto según el rol de quien escribe.

## 2. Catálogo de programas (CU2) — Administrador

`insPro` de Informe Once (NOTICIERO), Butaca Once (PELICULA) y Deportivo 11 (REVISTA); `lisPro["*"]`.

## 3. Materiales de video (CU4) — Administrador

Las tres notas del noticiero, las dos películas, el Deportivo 11 y el spot. `lisMat["PENDIENTE"]`: todo queda pendiente hasta que Prensa lo revise.

## 4. Validación (CU5) — Prensa

Prensa aprueba los materiales y **rechaza** `DEP-2011` («Audio desfasado»). El Administrador lo corrige con `modMat` (archivo nuevo) y Prensa lo aprueba.
Regla a mostrar: **quien registra un material no lo valida** (si el Administrador intenta `valMat` sobre un material suyo, el sistema lo rechaza).

## 5. Parrilla y escaleta (CU3) — Administrador y Prensa

El Administrador crea la parrilla y sus 4 bloques. Prensa arma la escaleta del noticiero (`insLea`), la reordena (`modLea`) y la muestra con
`lisLea`: el orden de las notas que antes iba por WhatsApp, con la hora estimada de entrada de cada una.

## 6. Pauta y spots (CU6) — Administrador

`insPau` de Banco Sol y dos `insSpo` en la pausa de las 21:00; `mosPau` muestra los spots programados.

## 7. Revisar, publicar y cambio de último momento — Administrador

`estPar` (todo LISTO) → `pubPar` (publicada; avisa los huecos del día, que no impiden publicar) → `camBlo` cambia la película con la parrilla ya
publicada: **llega el aviso por correo a Prensa y a Producción** (los alias). Eso es lo que reemplaza al WhatsApp.

## 8. Emisión real (CU7) — Producción

Cuatro `insEmi` (EMITIDO, EMITIDO, PARCIAL y NO_EMITIDO) y `conPar`: **50 % de cumplimiento** (2 de 4 bloques completos).
El Administrador muestra el historial con `lisAud["2026-11-20"]`.
Error a mostrar: un `insEmi` de un bloque que todavía no empieza se rechaza.

## 9. Reportes (CU8) — Administrador

`repPro`, `repMat`, `repPau` y `repCum` del mes.

## Avisos automáticos que se ven solos

- Material pendiente sin revisar: recordatorio a Prensa y luego escalamiento al Administrador (plazos en `config.properties`: `revision.minutos_aviso` y `revision.minutos_escalar`).
- Parrilla en riesgo (sin publicar o con materiales sin aprobar): aviso al Administrador y, si hay pendientes, a Prensa.
- Bloque terminado sin `insEmi`: alerta a Producción a los `alerta.emision_minutos`.

## Si algo falla

- **No llegan las respuestas a Gmail:** el sistema tiene que correr conectado a la red de la facultad.
- **«no está registrado en el sistema»:** el remitente no es un usuario (revisar el «De:» elegido en Gmail).
- **«Ya existe…»:** el paso ya se hizo antes (por ejemplo, la parrilla o el programa ya están creados); seguir con el siguiente.
- **Ver qué llegó al buzón sin procesar:** `scripts\ejecutar.bat --buzon`.
