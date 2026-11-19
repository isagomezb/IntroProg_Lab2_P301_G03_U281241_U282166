# Lab 2 - Creació d'un joc de pong

**PRÀCTICA 2, P301, GRUP 03**  
**INTEGRANTS:** Isabella Gomez i Neus Fenés (U281241 i U282166)

---

## 1. Equip i distribució del treball

En aquest treball ens hem dividit la feina de la manera següent:

* **Isabella Gómez:** Configuració inicial del repositori a GitHub; implementació de les mecàniques principals del joc (`move_paddle()`, `move_ball()`, `collide_ball_walls()`, `collide_ball_paddle()` i `reset_ball()`); proves del comportament de la pilota i les pales i neteja final dels comentaris.
* **Neus Fenés:** Implementació de les funcions del menú de configuració i validació d'entrades (`ask_yes_no()`, `read_int_in_range()`, `read_int_in_range_w_default()`, `read_color()`, `change_colors()` i `get_config()`); implementació de la funció opcional `compute_ai_direction()`, correcció de l'error de configuració descrit a la secció 4 i redacció de l'informe.

Tot i fer cadascuna les meves línies de codi, ens hem ajudat mútuament al llarg de tot el procés per tal que totes dues entenguéssim el funcionament complet del programa, detectar errors i fer revisions exhaustives. Per tant, ambdós membres de l'equip hem revisat, executat, provat i depurat conjuntament totes les funcions i podem explicar qualsevol part del codi de `pong.py`.

---

## 2. Visió general del disseny

El programa està estructurat de manera modular com una cadena de petites funcions que es comuniquen exclusivament a través de paràmetres i valors de retorn.

### Fase de configuració (abans d'obrir la finestra del joc):
1. Primer, `main()` crida a `get_config()`, que utilitza `ask_yes_no()` (la qual només accepta `"y"`, `"yes"`, `"n"` o `"no"`, tant en minúscules com en majúscules) per demanar a l'usuari si vol modificar els paràmetres del joc. En cas de resposta afirmativa, crida cinc vegades a `read_int_in_range_w_default()` per configurar l'alçada de la pala, la velocitat de la pala, la velocitat horitzontal X de la pilota, la velocitat vertical Y de la pilota i el radi de la pilota, i retorna els cinc valors com una tupla.
2. Seguidament, `main()` crida a `change_colors()`, que torna a utilitzar `ask_yes_no()` per preguntar si es volen canviar els colors del programa. Si la resposta és afirmativa, crida dues vegades a `read_color()` (primer per determinar el color de fons i després per al color de primer pla). Al seu torn, `read_color()` crida tres vegades a `read_int_in_range()` per obtenir components $(R, G, B)$ vàlids dins el rang $[0, 255]$.
3. En cas que l'usuari doni respostes negatives a alguna o a totes dues preguntes anteriors, les funcions retornen directament les constants definides per defecte.

### Inicialització i bucle principal (`main()`):
Un cop obtinguda la configuració, `main()` crea la finestra de Pygame, inicialitza les posicions verticals de les pales al centre (`p1_y`, `p2_y`) i crida a `reset_ball()` per col·locar la pilota al centre de la pantalla amb una direcció horitzontal definida i una direcció vertical aleatòria.

Cada fotograma ($FPS = 60$) es processa en quatre passos seqüencials:
1. **Entrada:** Llegir el teclat amb `read_player_direction()` i, si el mode màquina està activat (`PLAY_AGAINST_MACHINE = True`), crida a `compute_ai_direction()` per controlar la pala dreta de manera automàtica.
2. **Actualització d'estat:** `main()` crida a `move_paddle()`, `move_ball()`, `collide_ball_walls()` i `collide_ball_paddle()` per a ambdues pales.
3. **Puntuació:** Es detecta un gol quan la pilota surt completament de la pantalla, s'actualitza el marcador, fa una breu pausa (`SERVE_PAUSE`) i crida a `reset_ball()`. Si un jugador arriba a `POINTS_END` (5 punts), `show_message()` mostra *"Game Over!"* i finalitza la partida.
4. **Renderització:** Dibuixa l'escena amb `draw_scene()` i espera al següent fotograma.

---

## 3. Decisions clau d'implementació

### 3.1 Col·lisió pilota-pala i prevenció d'encallaments (`collide_ball_paddle`)
La pilota circular es tracta mitjançant una aproximació de caixa delimitadora alineada als eixos (AABB): un quadrat de costat $2 \cdot \text{radius}$ centrat en $(x, y)$. Cada pala es representa com el rectangle (`paddle_x`, `paddle_y`, `paddle_w`, `paddle_h`). Les dues caixes només se solapen si s'intersequen simultàniament en tots dos eixos:

```python
overlaps = (ball_left < paddle_right and 
            ball_right > paddle_left and
            ball_top < paddle_bottom and 
            ball_bottom > paddle_top)
```

Quan es detecta una col·lisió, s'inverteix la velocitat horitzontal `vx`. La part clau és la recol·locació: si $vx > 0$, la pilota es movia cap a la dreta i es col·loca just a l'exterior de la cara esquerra (`paddle_left - radius`); en cas contrari ($vx < 0$) es col·loca just fora de la cara dreta (`paddle_right + radius`). 

Sense aquesta correcció, la pilota continuaria solapada amb la pala en el fotograma següent, `vx` s'invertiria de nou repetidament i la pilota semblaria quedar-se atrapada o vibrant dins la pala. La funció retorna la posició corregida, les noves velocitats i el booleà `hit`, de manera que `main()` només actualitza les velocitats quan `hit` és `True`.

### 3.2 Validació d'entrades al menú de configuració
Cada entrada numèrica passa per `read_int_in_range()` (o `read_int_in_range_w_default()`): manté la pregunta dins d'un bucle `while True`, llegeix l'entrada amb `input()` i realitza la conversió `int()` dins d'un bloc `try / except ValueError` per comprovar que el valor inserit és correcte. Si la conversió funciona, un bloc `else:` comprova que es compleixi `low <= value <= high`. D'aquesta manera, lletres, línies buides, nombres decimals o valors fora de rang mostren un missatge d'error clar i repeteixen la pregunta sense penjar mai el programa.

Per la seva banda, `ask_yes_no()` normalitza el text amb `.lower()` i accepta tant formes en minúscula com en majúscula de (`"y"`, `"n"`, `"yes"`, `"no"`), repetint la pregunta davant qualsevol altra entrada.

### 3.3 Diferències respecte als docstrings originals
* `get_config()` utilitza `read_int_in_range_w_default()` en lloc de `read_int_in_range()`: Això permet que quan l'usuari premi ENTER directament es conservi cada constant per defecte, implementant la funcionalitat opcional.
* `PLAY_AGAINST_MACHINE = True`: Quan s'activa aquest indicador a la secció de constants és per a què la pala dreta sigui controlada per `compute_ai_direction()`. Per jugar dues persones amb teclat, només cal canviar aquest valor a `False` (el bucle principal no cal que sigui modificat).
* **Idioma:** Els comentaris del codi s'han redactat en anglès per mantenir la coherència amb els docstrings de la plantilla, mentre que els missatges interactius per al jugador es mostren en català (o anglès segons la versió final del menú).

---

## 4. Problemes trobats i solucions

* **Error a la configuració:** La primera versió de `get_config()` cridava a `read_int_in_range_w_default()` passant-li només 3 arguments, però la funció en requereix 4 (el missatge, el mínim, el màxim i el valor per defecte). En obrir el menú de configuració es produïa un `TypeError`. El vam detectar en provar el menú, vam afegir les constants per defecte que faltaven (`PADDLE_H`, `SPEED_PADDLE`, `SPEED_BALL_X`, `SPEED_BALL_Y` i `BALL_RADIUS`) i vam tornar a verificar el menú amb entrades vàlides i invàlides.
* **Vibració de la pala controlada per la IA:** Una pala que intenta alinear-se exactament al píxel amb `ball_y` oscil·la contínuament amunt i avall. A `compute_ai_direction()` vam introduir una banda de tolerància de 10 píxels al voltant del centre de la pala: dins d'aquesta franja la funció retorna `0` i la pala es manté quieta de manera natural.

---

## 5. Proves realitzades

Hem executat el joc localment amb Pygame (`pygame 2.6.1`) i hem verificat els comportaments següents:

* **Moviment i límits:** Les pales s'aturen exactament als marges superior (`0`) i inferior (`HEIGHT - paddle_h`); la pilota avança segons $(vx, vy)$ i rebota correctament a les parets superior i inferior.
* **Col·lisions i puntuació:** La pilota rebota en ambdues pales sense quedar-se enganxada; els gols es detecten únicament quan la pilota supera completament els marges esquerre o dret, incrementant el marcador corresponent i finalitzant la partida en arribar a `POINTS_END` (5 punts).
* **Robustesa del menú de configuració:** Hem provat d'introduir lletres (`"abc"`), línies buides, nombres decimals (`"3.5"`), nombres negatius, valors per sota del mínim i per sobre del màxim, així com els límits exactes (`low` i `high`). El programa no s'ha penjat en cap cas.
* **Colors RGB:** Els valors fora del rang $[0, 255]$ es rebutgen i es torna a demanar el component corresponent.
* **Mode IA:** La pala dreta segueix la trajectòria vertical de la pilota i es queda quieta dins la banda de tolerància.

---

## 6. Funcionalitats opcionals implementades

* `read_int_in_range_w_default()`: Funciona igual que `read_int_in_range()`, però si l'usuari prem ENTER (cadena buida `""`), retorna immediatament el `default_val` passat per paràmetre sense tornar a preguntar. S'utilitza a `get_config()` per als cinc paràmetres del joc.
* `compute_ai_direction()`: Calcula el centre vertical de la pala (`paddle_y + paddle_h / 2`) i el compara amb `ball_y` utilitzant una tolerància de 10 píxels per retornar `-1` (pujar), `0` (quiet) o `+1` (baixar). S'activa mitjançant la constant `PLAY_AGAINST_MACHINE = True` per controlar la pala dreta (es pot canviar a `False` per jugar 2 jugadors).

---

## 7. Ús d'eines d'IA

Hem utilitzat Google Gemini durant la revisió del codi per suggerir millores d'estil i resoldre errors puntuals d'execució a la terminal.

No hem acceptat cap resposta sense verificar-la prèviament: hem contrastat cada suggeriment amb el fitxer `pong.py` final, hem executat el joc per comprovar tots els casos límit i hem descartat o corregit qualsevol proposta que no s'ajustés als docstrings.
