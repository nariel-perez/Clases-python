[Contenidos](../Contenidos.md) \| [Próximo (2 Especificación, Documentación y contratos+)](02_Especificación.md)

# 6.1 Control de errores

Aunque ya hablamos de *excepciones*, en esta sección hablaremos de administración de excepciones y control de errores con mayor detalle.

### Formas en que los programas fallan

Python no hace ningún control ni validación sobre los tipos de los argumentos que las funciones reciben ni los valores de estos argumentos. Las funciones trabajarán sobre todo dato que sea compatible con las instrucciones dentro de la función. 

```python
def add(x, y):
    return x + y

add(3, 4)               # 7
add('Hello', 'World')   # 'HelloWorld'
add('3', '4')           # '34'
```

Si existen errores en una función, serán evidentes durante la ejecución de la función (en forma de una excepción).

```python
def add(x, y):
    return x + y

>>> add(3, '4')
Traceback (most recent call last):
...
TypeError: unsupported operand type(s) for +:
'int' and 'str'
>>>
```

Python acusa los errores en el idioma en inglés. El error acusado aquí puede traducisrse como:
```
Recapitulando (llamada más reciente al final)
...
Error de tipo (de datos): tipo de argumento no admitido para +: 'int' y 'str'.
```

Es decir: la función intentó aplicar el operador + (suma) a dos argumentos de tipos distintos (entero y cadena) y no supo hacerlo. Luego levantó una excepción. 

### Excepciones

Como ya dijimos, las excepciones son una forma de señalar errores en tiempo de ejecución.  Para levantar una excepción, usá la instrucción `raise` . 

```python
if nombre not in autorizados:
    raise RuntimeError(f'{nombre} no autorizado')
```

Para _atrapar_ una excepción, usá us bloque `try-except`. 

```python
try:
    authenticate(nusuario)
except RuntimeError as e:
    print(e)
```

### Administración de excepciones

Una excepción se propagará hasta el primer `except` que coincida con ella.

```python
def grok():
    ...
    raise RuntimeError('Whoa!')   # Levanta una excepción aquí

def spam():
    grok()                        # Esta llamada va a levantaruna excepción

def bar():
    try:
       spam()
    except RuntimeError as e:     # Aquí atrapamos la excepción
        ...

def foo():
    try:
         bar()
    except RuntimeError as e:     # Por lo tanto la excepción no llega aquí
        ...

foo()
```

[oski]: # (To handle the exception, put statements in the `except` block. You can add any statements you want to handle the error.)

Para administrar la excepción, usá instrucciones en el bloque `except`. Es pertinente realizar acciones relacionadas con la excepción en particular. 

```python
def grok(): ...
    raise RuntimeError('Whoa!')

def bar():
    try:
      grok()
    except RuntimeError as e:   # Excepción atrapada
        statements              # Ejecute estos comandos
        statements
        ...

bar()
```

Una vez atrapada la excepción, la ejecución continúa en la primera instrucción a continuación del `try-except`.

```python
def grok(): ...
    raise RuntimeError('Whoa!')

def bar():
    try:
      grok()
    except RuntimeError as e:   # Excepción atrapada
        statements
        statements
        ...
    statements                  # La ejecución del programa 
    statements                  # continúa aquí
    ...

bar()
```

### Excepciones integradas

Hay más de una veintena de tipos de excepciones ya integradas en Python. Normalmente, el nombre de la excepción indica qué anduvo mal (por ejemplo, se levanta un  `ValueError` si el valor suministrado no es adecuado). La siguiente no es una lista completa. Vas a encontrar más en la [documentación del lenguaje](https://docs.python.org/3/library/exceptions.html#bltin-exceptions).

```python
ArithmeticError
AssertionError
EnvironmentError
EOFError
ImportError
IndexError
KeyboardInterrupt
KeyError
MemoryError
NameError
ReferenceError
RuntimeError
SyntaxError
SystemError
TypeError
ValueError
```

### Valores asociados a excepciones

Usualmente las excepciones llevan valores asociados, que proveen más información sobre la causa detallada del error. Este valor puede ser una cadena (*string*) o una tupla de valores diversos (por ejemplo un código de error y un texto explicando ese código). 

```python
raise RuntimeError('Nombre de usuario inválido')
```

La instancia de la variable suministrada a `except` lleva este valor asociado. 

[oski]: # (no me gusta como queda. Confuso. El original tambien confuso. En los ejemplos no se muestra como usar este valor asociado.  Igual sigo.
This value is part of the exception instance that's placed in the variable supplied to `except`.)

```python
try:
    ...
except RuntimeError as e:   # `e` contiene la excecpcion lanzada
    ...
```

`e` es una instancia del mismo tipo que la excepción, aunque si lo imprimís suele tener aspecto de una cadena de caracteres.

```python
except RuntimeError as e:
    print('Failed : Reason', e)
```

### Podés atrapar múltiples excepciones

Es posible atrapar diferentes tipos de excepciones en la misma porción de código, si incluís varios `except` en tu `try:`.

```python
try:
  ...
except LookupError as e:
  ...
except RuntimeError as e:
  ...
except IOError as e:
  ...
except KeyboardInterrupt as e:
  ...
```

Como alternativa, si las vas a procesar a todas de la misma manera, las podés agrupar:

```python
try:
  ...
except (IOError,LookupError,RuntimeError) as e:
  ...
```

### Todas las excepciones

Para atrapar todas y cualquier excepción, se usa `Exception` así: 

```python
try:
    ...
except Exception:       # PELIGRO. (ver abajo)
    print('An error occurred')
```

En general es mala idea "administrar" las excepciones de este modo, porque no te dá ninguna pista de porqué falló el programa.

### Así NO se atrapan excepciones. 

Así es como NO debe hacerse la administración de excepciones.

```python
try:
    hacer_algo()
except Exception:
    print('Hubo un error.')
```

Esto atrapa todos los errores posibles, y puede complicar mucho el debugging cuando el código falla por algún motivo que no esperabas (por ejemplo, falta algún módulo de Python y lo único que te dice es "Hubo un error").

### Así es un poco mejor.

Si va a atrapar todas las excepciones, aquí hay un modo algo más decente:

```python
try:
    hacer_algo()
except Exception as e:
    print('Hubo un error. Porque...', e)
```

`Exception` incluye toda excepción posible, de modo que no sabés cuál atrapaste.
Al menos esta última versión te informa el motivo específico del error. Siempre es bueno tener alguna forma de ver o informar errores cuando atrapás todas las excepciones posibles. 

Sin embargo, por lo general es mejor atrapar errores específicos, y sólo aquellos que podés administrar. Errores que no sepas como manejar adecuadamente, déjalos correr (tal vez alguna otra porción de código los atrape y administre correctamente o sea adecuado que se detenga la ejecución).

### Re-lanzar una excepción

Si necesitás hacer algo en respuesta a una excepción pero no querés atraparla, podés usar `raise` para volver a lanzar la misma excepción.

```python
try:
    hacer_algo()
except Exception as e:
    print('Hubo un error. Porque...', e)
    raise
```

Esto te permite, por ejemplo, llevar un registro de las excepciones (*log*) sin administrarla, y re-lanzarla para administrarla adecuadamente más tarde.

### Buenas prácticas al administrar excepciones

No atrapes excepciones que no vayas a manejar adecuadamente. Dejalas caer ruidosamente. Si es importante, alguien se va a encargar del problema. Sólo atrapá excepciones si *sos ese "alguien"*. Es decir: sólo atrapá aquéllos errores que podés administrar elegantemente y permitir que el programa siga ejecutando.   

### La instrucción `finally`.

`finally` especifica que esa porción de código debe ejecutarse sin importar si una excepción fué atrapada o no.

```python
lock = Lock()
...
lock.acquire()
try:
    ...
finally:
    lock.release()  # esto SIEMPRE se ejecuta. Haya o no haya excepciones.
```

Una estructura como ésa resulta en un manejo seguro de los recursos disponibles (seguros, archivos, hardware, etc.)

## Ejercicios

### Ejercicio 6.1: Levantando excepciones

Modifcá tu código para que lance una excepción en caso que ambos parámetros `select` y `has_headers=False` sean pasados juntos. Y que resulte: 

```python
>>> parse_csv('Data/precios.csv', select=['name','precio'], has_headers=False)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "fileparse.py", line 9, in parse_csv
    raise RuntimeError("para seleccionar argumentos necesito encabezados")
RuntimeError: select argument requires column headers
>>>
```

Ahora que agregaste este control, te estarás preguntando si no deberías comprobar otras cosas también en tu función. Por ejemplo, ¿deberías comprobar que `nombre_archivo` sea una cadena, que `tipos` sea una lista y otras cosas de ese estilo? 

Como regla general, es mejor no controlar esas cosas, y dejar que el programa dé un error ante entradas inválidas. El mensaje de error va a darte una idea del origen del problema y te va ayudar a solucionarlo.

El motivo principal para agregar controles de calidad sobre los argumentos de entrada es evitar que tu programa sea ejecutado en condiciones que no tienen sentido (pedirle que haga algo que requiere encabezados y simultáneamente decirle que no existen encabezados) lo cual implicaria un error en el código que llamado a tu función. La idea general es estar protegido contra situaciones que "no deberían suceder" pero podrían. 


### Ejercicio 6.2: Atrapar excepciones
La función `parse_csv()` que escribiste está destinada a procesar un archivo completo. Pero en una situacion real, es posible que los archivos CSV de entrada estén "rotos", ausentes, o su contenido no conforme con el formato esperado. Probá esto:  

```python
>>> camion = parse_csv('Data/missing.csv', types=[str, int, float])
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "fileparse.py", line 36, in parse_csv
    row = [func(val) for func, val in zip(types, row)]
ValueError: invalid literal for int() with base 10: ''
>>>
```

Modificá la función `parse_csv()` de modo que atrape todas las excepciones de tipo `ValueError` generadas durante el armado de los registros a devolver e imprima un mensaje de advertencia para las filas que no pudieron ser convertidas.
Este mensaje deberá incluír el número de fila que causó el fallo y el motivo por el cual falló la conversión. Para probar tu nueva función, intentá procesar `Data/missing.csv`. Debería darte algo así:  

```python
>>> camion = parse_csv('Data/missing.csv', types=[str, int, float])
Row 4: No pude convertir ['Mandarina', '', '51.23']
Row 4: Motivo: invalid literal for int() with base 10: ''
Row 7: No pude convertir ['Naranja', '', '70.44']
Row 7: Motivo: invalid literal for int() with base 10: ''
>>>
>>> camion
[{'precio': 32.2, 'nombre': 'Lima', 'cajones': 100}, {'precio': 91.1, 'nombre': 'Naranja', 'cajones': 50}, {'precio': 83.44, 'nombre': 'Caqui', 'cajones': 150}, {'precio': 40.37, 'nombre': 'Durazno', 'cajones': 95}, {'precio': 65.1, 'nombre': 'Mandarina', 'cajones': 50}]
>>>
```

### Ejercicio 6.3: Silenciar errores
Modificá `parse_csv()` de modo que el usuario pueda silenciar los informes de errores en el parseo de los datos que agregaste antes. Por ejemplo:

```python
>>> camion = parse_csv('Data/missing.csv', types=[str,int,float], silence_errors=True)
>>> camion
[{'precio': 32.2, 'name': 'Lima', 'cajones': 100}, {'precio': 91.1, 'name': 'Naranja', 'cajones': 50}, {'precio': 83.44, 'name': 'Caqui', 'cajones': 150}, {'precio': 40.37, 'name': 'Durazno', 'cajones': 95}, {'precio': 65.1, 'name': 'Mandarina', 'cajones': 50}]
>>>
```

### Comentarios

Lograr un buen manejo o administración de errores es una de las partes más difíciles en la mayoría de los programas. Estás intentando preveer imprevistos. Como regla general, no silencies los errores. Es mejor informar los problemas y darle al usuario la opción de silenciarlos explícitamente. Un buen diálogo entre el código y el usuario facilita el debugging y el buen uso del programa. 


[Contenidos](../Contenidos.md) \| [Próximo (2 Especificación, Documentación y contratos+)](02_Especificación.md)
