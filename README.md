# HOLLOW BEASTS
## Documento de análisis y diseño: videojuego RPG por turnos

| Campo | Detalle |
|---|---|
| Proyecto | Hollow Beasts |
| Tipo | Aplicación de escritorio, RPG 2D por turnos |
| Lenguaje | Python 3 |
| Interfaz gráfica | tkinter |
| Persistencia | JSON |
| Paradigma | Programación orientada a objetos |
| Patrón de diseño | Factory Method |
| Asignatura | Programación IV |
| Integrantes | [completar] |
| Versión | 2.0 |

---

## 1. Descripción general

Hollow Beasts es un juego de combates por turnos. El jugador controla a un guerrero que se enfrenta, en orden, a cuatro monstruos: Goblin, Esqueleto, Orco y Dragón oscuro. En cada turno elige una acción (atacar, usar una habilidad, defenderse o recuperar maná). Si el monstruo sigue vivo, responde con su propio turno.

El proyecto se centra en los combates y en la aplicación de los temas del curso: clases, herencia, herencia múltiple, polimorfismo, encapsulamiento, modularidad, persistencia en JSON y el patrón Factory Method.

### 1.1 Objetivo general

Desarrollar un juego de combates por turnos en Python, con interfaz en tkinter, que aplique programación orientada a objetos y guarde el progreso en un archivo JSON.

### 1.2 Alcance

Incluye: nueva partida, selección de monstruo, combate por turnos con cuatro acciones, victorias y derrotas, avance entre monstruos, pantalla de resultados y guardado/carga de la partida.

No incluye: mapas, multijugador, inventario, tienda, sonidos ni animaciones.

---

## 2. Reglas del juego

### 2.1 Personaje principal

| Atributo | Valor inicial |
|---|---|
| Vida máxima / actual | 150 |
| Maná máximo / actual | 100 |
| Ataque | 20 |
| Defensa | 10 |
| Nivel | 1 |
| Experiencia | 0 |
| Victorias / derrotas | 0 / 0 |

Cada 100 de experiencia sube un nivel: aumenta 10 la vida máxima y 2 el ataque.

### 2.2 Monstruos

| Monstruo | Vida | Ataque | Defensa | Experiencia | Comportamiento |
|---|---|---|---|---|---|
| Goblin | 60 | 12 | 3 | 20 | Ataque rápido |
| Esqueleto | 90 | 18 | 6 | 35 | Ataque equilibrado |
| Orco | 130 | 23 | 9 | 50 | Ataque pesado (+5 de daño) |
| Dragón oscuro | 250 | 30 | 12 | 100 | Alterna ataque normal y especial |

El dragón alterna sus ataques según el número de turno (par o impar). Se evita `random` para mantenerse dentro de lo visto en clase.

### 2.3 Acciones del jugador

| Acción | Efecto | Costo |
|---|---|---|
| Ataque básico | Daño = máximo entre 1 y (ataque del jugador − defensa del monstruo) | 0 MP |
| Ataque especial (Golpe devastador) | Daño base 45, reducido por la defensa del monstruo | 30 MP |
| Defenderse | Reduce a la mitad el daño del siguiente ataque enemigo | 0 MP, consume turno |
| Recuperar maná | Recupera 25 MP sin superar el máximo | 0 MP, consume turno |

### 2.4 Reglas generales

- La vida y el maná nunca superan su máximo ni bajan de cero.
- Un personaje con vida cero está derrotado y no puede actuar.
- Las habilidades no se ejecutan si no hay maná suficiente.
- Una acción válida consume el turno del jugador. Después actúa el monstruo, si sigue vivo.
- El combate termina cuando uno de los dos cae.
- Una victoria da experiencia, registra el resultado y permite avanzar. Una derrota registra el resultado y permite reintentar o volver al menú.
- Derrotar al dragón muestra la pantalla de victoria final.

---

## 3. Requerimientos

### 3.1 Requerimientos funcionales

| Código | Requerimiento |
|---|---|
| RF01 | Iniciar una nueva partida solicitando el nombre del jugador. |
| RF02 | Mostrar las estadísticas del jugador. |
| RF03 | Mostrar el siguiente monstruo disponible y sus datos. |
| RF04 | Ejecutar ataque básico, ataque especial, defensa y recuperación de maná. |
| RF05 | Validar que haya maná suficiente antes de usar una habilidad. |
| RF06 | Calcular el daño y actualizar vida y maná después de cada acción. |
| RF07 | Ejecutar el turno del monstruo cuando siga con vida. |
| RF08 | Determinar el final del combate y registrar victoria o derrota. |
| RF09 | Avanzar al siguiente monstruo después de una victoria. |
| RF10 | Guardar la partida en un archivo JSON. |
| RF11 | Cargar una partida guardada desde el menú principal. |
| RF12 | Informar cuando no exista partida guardada. |

### 3.2 Requerimientos no funcionales

| Código | Requerimiento |
|---|---|
| RNF01 | El programa se desarrolla en Python 3. |
| RNF02 | Las interfaces se construyen con tkinter. |
| RNF03 | El código se organiza en módulos separados por responsabilidad. |
| RNF04 | Los atributos internos son privados y se acceden con `get_` y `set_`. |
| RNF05 | Los `set_` validan los valores antes de guardarlos. |
| RNF06 | La información persistente se guarda en JSON. |
| RNF07 | El programa valida las acciones antes de ejecutarlas. |
| RNF08 | El programa informa los errores de archivo sin cerrarse de forma inesperada. |

---

## 4. Diseño de clases

### 4.1 Personaje (clase base)

Atributos privados: `nombre`, `vida_maxima`, `vida_actual`, `mana_maximo`, `mana_actual`, `ataque`, `defensa`.

Métodos principales: `recibir_danio(cantidad)`, `sanar(cantidad)`, `recuperar_mana(cantidad)`, `esta_vivo()`, `atacar(objetivo)`.

Ejemplo de encapsulamiento:

```python
class Personaje:
    def __init__(self, nombre, vida_maxima, mana_maximo, ataque, defensa):
        self.__nombre = nombre
        self.__vida_maxima = vida_maxima
        self.__vida_actual = vida_maxima
        self.__mana_maximo = mana_maximo
        self.__mana_actual = mana_maximo
        self.__ataque = ataque
        self.__defensa = defensa

    def get_vida_actual(self):
        return self.__vida_actual

    def set_vida_actual(self, valor):
        if 0 <= valor <= self.__vida_maxima:
            self.__vida_actual = valor
        else:
            print("Vida fuera de rango, se conserva el valor anterior")

    def recibir_danio(self, cantidad):
        nueva_vida = self.__vida_actual - cantidad
        if nueva_vida < 0:
            nueva_vida = 0
        self.set_vida_actual(nueva_vida)

    def esta_vivo(self):
        return self.__vida_actual > 0
```

Todos los atributos siguen el mismo patrón: `get_` para leer y `set_` para cambiar, con validación.

### 4.2 Jugador (hereda de Personaje)

Atributos adicionales: `experiencia`, `victorias`, `derrotas`, `habilidades` (lista de objetos Habilidad).

Métodos: `usar_habilidad(habilidad, objetivo)`, `ganar_experiencia(cantidad)`, `subir_nivel()`, `registrar_victoria()`, `registrar_derrota()`, `obtener_estadisticas()`.

### 4.3 Monstruo (hereda de Personaje)

Atributos adicionales: `experiencia_recompensa`, `dificultad`.

Métodos: `elegir_accion(numero_turno)` y `atacar(objetivo)`. Ambos se sobrescriben en cada subclase, lo que demuestra polimorfismo.

### 4.4 Goblin, Esqueleto, Orco (heredan de Monstruo)

Cada clase define sus estadísticas en el constructor y redefine `atacar()` con su propio comportamiento.

### 4.5 Volador (clase de apoyo)

Clase sin atributos, con un único método `volar()`. Su función es dar la capacidad de volar a un monstruo sin repetir código.

### 4.6 Dragon (herencia múltiple: Monstruo y Volador)

```python
class Dragon(Monstruo, Volador):
    def __init__(self):
        Monstruo.__init__(self, "Dragon oscuro", 250, 30, 12, 100, "Jefe final")

    def atacar(self, objetivo):
        print("El dragon ataca con sus garras")

    def ataque_especial(self, objetivo):
        print("El dragon lanza una llamarada desde el aire")
```

Como `Volador` no tiene constructor propio ni atributos, no hay conflicto al inicializar la clase. Si `Monstruo` y `Volador` tuvieran un método con el mismo nombre, el orden de la herencia (Monstruo primero) decidiría cuál se usa. Esto se puede comprobar con `Dragon.__mro__`.

### 4.7 Habilidad

Atributos: `nombre`, `costo_mana`, `danio`, `descripcion`.

Métodos: `puede_ejecutarse(usuario)`, `calcular_danio(objetivo)`.

### 4.8 Combate

Atributos: `jugador`, `monstruo`, `turno` (número), `terminado`, `resultado`, `jugador_defendiendo`.

Métodos: `accion_jugador(accion)`, `turno_monstruo()`, `verificar_resultado()`, `finalizar()`.

Combate contiene las reglas. No dibuja nada: la interfaz solo muestra su estado.

### 4.9 Juego

Atributos: `jugador`, `monstruos` (lista), `indice_monstruo`, `combate_actual`.

Métodos: `nueva_partida(nombre)`, `siguiente_monstruo()`, `iniciar_combate()`, `guardar()`, `cargar()`.

### 4.10 FabricaMonstruos

Crea los monstruos a partir de un texto. Se explica en la sección 6.

### 4.11 GestorPartida

Lee y escribe el archivo `partida.json`. Se explica en la sección 8.

---

## 5. Diagrama de clases

```
                 Personaje
        (nombre, vida, mana, ataque, defensa)
           /                        \
       Jugador                   Monstruo  ------------ Volador
  (experiencia, victorias,      (recompensa,            (volar)
   derrotas, habilidades)        dificultad)               \
                                 /   |   \                  \
                            Goblin Esqueleto Orco        Dragon
                                                      (Monstruo + Volador)

  Habilidad  <-- usa --  Jugador
  Combate    --> Jugador, Monstruo
  Juego      --> Jugador, Combate, FabricaMonstruos, GestorPartida
  FabricaMonstruos --> crea Goblin, Esqueleto, Orco, Dragon
```

---

## 6. Patrón de diseño: Factory Method

### 6.1 Problema

El juego necesita crear monstruos distintos, pero no debería conocer los detalles de cada uno. Si la creación se escribe directamente en la interfaz o en `Juego`, cualquier cambio en un monstruo obliga a modificar varios archivos.

### 6.2 Solución

Una clase `FabricaMonstruos` decide qué clase instanciar según el tipo solicitado.

```python
class FabricaMonstruos:
    def crear_monstruo(self, tipo):
        if tipo == "goblin":
            return Goblin()
        elif tipo == "esqueleto":
            return Esqueleto()
        elif tipo == "orco":
            return Orco()
        elif tipo == "dragon":
            return Dragon()
        else:
            print("Tipo de monstruo no valido")
            return None
```

Uso desde `Juego`:

```python
fabrica = FabricaMonstruos()
self.monstruos = [
    fabrica.crear_monstruo("goblin"),
    fabrica.crear_monstruo("esqueleto"),
    fabrica.crear_monstruo("orco"),
    fabrica.crear_monstruo("dragon")
]
```

### 6.3 Beneficios

- `Juego` y la interfaz no dependen de cada clase de monstruo.
- Para agregar un monstruo nuevo, solo se crea su clase y se agrega un `elif` en la fábrica.
- La fábrica se puede probar sola.

---

## 7. Principios de POO aplicados

| Principio | Dónde se aplica |
|---|---|
| Clases y objetos | Todas las clases del juego. |
| Herencia simple | Jugador y Monstruo heredan de Personaje; Goblin, Esqueleto y Orco heredan de Monstruo. |
| Herencia múltiple | Dragon hereda de Monstruo y Volador. |
| Polimorfismo | `atacar()` y `elegir_accion()` tienen una versión por clase. Combate llama a `monstruo.atacar(jugador)` sin saber qué monstruo es. |
| Encapsulamiento | Atributos con `__`, acceso con `get_` y `set_`, validación en los `set_`. |
| Modularidad | Cada tema en su archivo (ver sección 9). |
| Persistencia | Datos guardados en JSON mediante GestorPartida. |

---

## 8. Persistencia en JSON

### 8.1 Archivo

Se usa un solo archivo, `partida.json`, con dos partes:

```json
{
    "jugador": {
        "nombre": "Guerrero",
        "nivel": 1,
        "vida_maxima": 150,
        "vida_actual": 150,
        "mana_maximo": 100,
        "mana_actual": 100,
        "ataque": 20,
        "defensa": 10,
        "experiencia": 0,
        "victorias": 0,
        "derrotas": 0
    },
    "progreso": {
        "indice_monstruo": 0,
        "combate_en_progreso": false,
        "juego_completado": false
    }
}
```

### 8.2 Clase GestorPartida

Métodos: `existe()`, `guardar(datos)`, `cargar()`.

- `guardar()` escribe el archivo con modo `"w"` y `json.dump`.
- `cargar()` devuelve los datos o `None` si el archivo no existe o está vacío.
- `existe()` devuelve `True` o `False`.

### 8.3 Manejo de errores

Sin `try/except`, el programa valida antes de leer: que el archivo exista, que no esté vacío y que contenga las claves `jugador` y `progreso`. Si el archivo está dañado y el profesor permite `try/except`, la lectura se encierra en un bloque `try` para mostrar un mensaje en lugar de cerrarse.

---

## 9. Estructura de archivos

```
hollow_beasts/
├── modelos.py       Personaje, Jugador, Habilidad
├── monstruos.py     Monstruo, Goblin, Esqueleto, Orco, Volador, Dragon
├── combate.py       Combate (turnos, daño, defensa, resultado)
├── fabrica.py       FabricaMonstruos
├── persistencia.py  GestorPartida (partida.json)
├── interfaz.py      Pantallas de tkinter
├── principal.py     Clase Juego y arranque del programa
└── partida.json     Se crea al guardar la primera partida
```

Todos los archivos deben estar en la misma carpeta.

---

## 10. Interfaces gráficas

El juego tiene cuatro pantallas, construidas con tkinter.

### 10.1 Menú principal
- Título, botones Nueva partida, Continuar y Salir.
- Nueva partida muestra un campo para escribir el nombre.
- Continuar comprueba que exista partida guardada. Si no existe, muestra un mensaje.

### 10.2 Selección de monstruo
- Estadísticas del jugador: nombre, nivel, vida y maná (con barras), ataque, defensa, victorias y derrotas.
- Datos del siguiente monstruo: nombre, vida, ataque, defensa y dificultad.
- Botón Combatir y botón Guardar.

### 10.3 Combate
- Dibujo del jugador y del monstruo con `Canvas` (rectángulos y óvalos).
- Barras de vida y maná del jugador, barra de vida del monstruo.
- Registro de acciones (un `Label` o un cuadro de texto que se actualiza).
- Botones: Ataque básico, Ataque especial, Defenderse, Recuperar maná.
- Mientras actúa el monstruo, los botones se desactivan.

### 10.4 Resultados
- Mensaje de victoria o derrota, daño producido, experiencia ganada y victorias acumuladas.
- Botones: Continuar (si hay siguiente monstruo), Reintentar (si perdió) y Menú principal.
- Si el dragón fue derrotado, muestra la pantalla de victoria final.

---

## 11. Casos de uso

| Código | Caso de uso | Actor | Flujo resumido |
|---|---|---|---|
| CU01 | Iniciar partida | Jugador | Elige Nueva partida, escribe el nombre, se crea el jugador. |
| CU02 | Continuar partida | Jugador | Se verifica el archivo y se cargan los datos. |
| CU03 | Ver estadísticas | Jugador | Se muestran los datos del jugador en la pantalla de selección. |
| CU04 | Seleccionar monstruo | Jugador | Se muestra el siguiente monstruo y se inicia el combate. |
| CU05 | Atacar | Jugador | Calcula el daño, descuenta vida y ejecuta el turno del monstruo. |
| CU06 | Usar habilidad | Jugador | Valida el maná, descuenta el costo y aplica el daño. |
| CU07 | Defenderse | Jugador | Activa la defensa y el monstruo ataca con la mitad de daño. |
| CU08 | Recuperar maná | Jugador | Suma 25 MP sin superar el máximo y pasa el turno al monstruo. |
| CU09 | Ver resultado | Jugador | Se determina victoria o derrota y se muestra la pantalla. |
| CU10 | Guardar partida | Jugador | Se escribe el estado actual en `partida.json`. |
| CU11 | Salir | Jugador | Se cierra la ventana y se termina el programa. |

---

## 12. Tarjetas CRC

| Clase | Responsabilidades | Colaboradores |
|---|---|---|
| Personaje | Guardar estadísticas, recibir daño, sanar, validar valores. | Jugador, Monstruo |
| Jugador | Usar habilidades, ganar experiencia, subir de nivel, llevar victorias y derrotas. | Personaje, Habilidad, Combate |
| Monstruo | Atacar, elegir acción, entregar recompensa. | Personaje, Combate |
| Goblin / Esqueleto / Orco | Definir sus estadísticas y su forma de atacar. | Monstruo |
| Volador | Permitir volar. | Dragon |
| Dragon | Atacar y usar ataque especial, combinando Monstruo y Volador. | Monstruo, Volador |
| Habilidad | Guardar datos, validar maná y calcular daño. | Jugador, Combate |
| Combate | Administrar turnos, daño, defensa y resultado. | Jugador, Monstruo, Habilidad |
| Juego | Crear el jugador y los monstruos, controlar la progresión. | FabricaMonstruos, Combate, GestorPartida |
| FabricaMonstruos | Crear el monstruo según el tipo pedido. | Goblin, Esqueleto, Orco, Dragon |
| GestorPartida | Guardar, cargar y verificar el archivo JSON. | Juego |

---

## 13. Plan de pruebas

Las pruebas se hacen manualmente desde el juego y con líneas de prueba en `principal.py`.

| Código | Prueba | Resultado esperado |
|---|---|---|
| P01 | Crear jugador | Estadísticas iniciales correctas. |
| P02 | `set_vida_actual(-10)` | Mensaje y valor anterior conservado. |
| P03 | Recibir daño mayor que la vida | Vida queda en 0 y el personaje figura como derrotado. |
| P04 | Ataque básico | El monstruo pierde vida. |
| P05 | Ataque especial con 20 MP | No se ejecuta y se muestra mensaje. |
| P06 | Defenderse y recibir ataque | El daño se reduce a la mitad. |
| P07 | Recuperar maná con 90 MP | El maná queda en 100, sin pasarse. |
| P08 | Acción fuera de turno | El sistema la rechaza. |
| P09 | Derrotar monstruo | Se registra la victoria y se suma experiencia. |
| P10 | Guardar y cerrar, luego Continuar | Los datos se recuperan. |
| P11 | Continuar sin archivo | Mensaje de que no hay partida guardada. |
| P12 | `Dragon.__mro__` | Muestra el orden de búsqueda con Monstruo antes que Volador. |
| P13 | Fábrica con tipo inválido | Mensaje y retorno de None. |

---

## 14. Manual de uso

1. Ejecutar `principal.py` desde la carpeta del proyecto.
2. En el menú, elegir Nueva partida y escribir el nombre del personaje.
3. En la pantalla de selección, revisar las estadísticas y pulsar Combatir.
4. En el combate, elegir una acción por turno. Las acciones se bloquean mientras actúa el monstruo.
5. Al terminar el combate, revisar el resultado. Después de una victoria, continuar con el siguiente monstruo.
6. Guardar la partida desde la pantalla de selección para continuarla después con la opción Continuar.

---

## 15. Cumplimiento de los requisitos del trabajo

| Requisito | Implementación | Sección |
|---|---|---|
| Clases y métodos | Personaje, Jugador, Monstruo, Goblin, Esqueleto, Orco, Volador, Dragon, Habilidad, Combate, Juego, FabricaMonstruos, GestorPartida | 4 |
| Objetos | Instancias creadas en `Juego` y en la fábrica | 4, 6 |
| Herencia simple | Personaje → Jugador / Monstruo; Monstruo → Goblin / Esqueleto / Orco | 4 |
| Herencia múltiple | Dragon(Monstruo, Volador) | 4.6 |
| Polimorfismo | `atacar()` y `elegir_accion()` por clase | 4, 7 |
| Encapsulamiento | Atributos privados, `get_` y `set_` con validación | 4.1 |
| Modularidad | Siete archivos por responsabilidad | 9 |
| Interfaces gráficas | Cuatro pantallas con tkinter | 10 |
| Persistencia | Archivo `partida.json` | 8 |
| Patrón de diseño | Factory Method en FabricaMonstruos | 6 |
| Documentación | Requisitos, CRC, casos de uso, diagrama, manual | 3, 11, 12, 5, 14 |

---

## 16. Conclusión

Hollow Beasts aplica en un solo proyecto los temas de Programación IV: clases, herencia simple y múltiple, polimorfismo, encapsulamiento con `get_` y `set_`, modularidad, persistencia en JSON, interfaces con tkinter y el patrón Factory Method. El alcance está limitado a combates por turnos con cuatro monstruos, lo que permite terminarlo con calidad y explicarlo completo en la sustentación.
