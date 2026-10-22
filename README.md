# Práctica: Encender y apagar un LED desde Python en Jetson Nano

**Reto TC1004B, Etapa 2: Sensores y Actuadores**
**Equipo:** Jetson Nano Developer Kit (JetPack / Ubuntu 18.04 ya instalado)

## Objetivo

Que el alumno controle un LED (encendido y apagado) desde un script de Python usando los pines GPIO de la Jetson Nano, dando el primer paso de la etapa de Sensores y Actuadores del reto, y de paso deje la Jetson con Python 3 y las librerías de GPIO listas para las siguientes prácticas.

## Conocimientos previos

- Conceptos básicos de un circuito con LED (ánodo/cátodo, por qué se necesita una resistencia limitadora de corriente).
- Nociones básicas de GPIO (pines de entrada/salida de propósito general).
- Python básico (variables, ciclos, manejo de excepciones).

## Materiales

- Jetson Nano Developer Kit, ya configurada (red, SSH, actualizaciones) según las notas de puesta a punto.
- Protoboard (breadboard).
- 1 LED de 5mm (cualquier color).
- 1 resistencia de 220 a 330 ohms.
- 2 o 3 cables jumper macho-hembra.
- Acceso a la terminal de la Jetson (directo con monitor/teclado, o por SSH).

## Duración estimada

50 minutos.

## Instrucciones

### Parte 1: Verificar e instalar Python 3

La imagen de Ubuntu 18.04 que trae JetPack usa Python 2.7 como el `python` del sistema (muchas herramientas internas de Ubuntu dependen de eso), pero normalmente ya incluye Python 3.6 instalado por separado como `python3`. Antes de instalar nada, hay que confirmar qué hay disponible.

1. Revisa qué versión responde el comando `python`:
   ```bash
   python --version
   ```
   Debería mostrar algo como `Python 2.7.17`.
2. Revisa si ya existe Python 3:
   ```bash
   python3 --version
   ```
   Si muestra algo como `Python 3.6.9`, Python 3 ya está instalado y puedes saltar al paso 4.
3. Si el comando anterior falla (`python3: command not found`), instálalo:
   ```bash
   sudo apt update
   sudo apt install -y python3 python3-dev
   ```
4. Instala (o verifica) `pip3`, el instalador de paquetes de Python 3:
   ```bash
   sudo apt install -y python3-pip
   python3 -m pip --version
   ```

**Importante:** no cambies el comando `python` del sistema para que apunte a Python 3 (por ejemplo con `update-alternatives`). En esta versión de Ubuntu varios scripts internos del sistema (incluyendo partes de `apt`) todavía esperan Python 2, y romperlos puede dejar la Jetson en un estado difícil de reparar. Para esta práctica (y las siguientes del reto), siempre invocamos explícitamente `python3` y `pip3`.

### Parte 2: Instalar la librería Jetson.GPIO y dar permisos

1. Instala la librería `Jetson.GPIO` (la que permite controlar los pines desde Python 3):
   ```bash
   sudo pip3 install Jetson.GPIO
   ```
2. Verifica que se instaló correctamente:
   ```bash
   python3 -c "import Jetson.GPIO as GPIO; print(GPIO.VERSION)"
   ```
3. Para no tener que correr todos los scripts con `sudo`, crea el grupo `gpio` y agrega tu usuario:
   ```bash
   sudo groupadd -f -r gpio
   sudo usermod -a -G gpio $USER
   ```
4. Localiza la carpeta donde quedó instalado el paquete (la ruta exacta cambia según la versión de JetPack):
   ```bash
   python3 -c "import Jetson.GPIO; print(Jetson.GPIO.__file__)"
   ```
5. En esa misma carpeta debe existir un archivo `99-gpio.rules`. Cópialo a udev (ajusta la ruta según lo que haya mostrado el paso anterior):
   ```bash
   sudo cp /usr/lib/python3.6/dist-packages/Jetson/GPIO/99-gpio.rules /etc/udev/rules.d/
   sudo udevadm control --reload-rules
   sudo udevadm trigger
   ```
6. Reinicia la Jetson para que el nuevo grupo y las reglas tomen efecto:
   ```bash
   sudo reboot
   ```

### Parte 3: Armar el circuito

1. Con la Jetson **apagada**, identifica en el header de 40 pines el pin físico 12 (numeración BOARD) y cualquier pin de tierra (por ejemplo, el pin físico 6).
2. Conecta el pin 12 a una terminal de la resistencia en el protoboard.
3. Conecta la otra terminal de la resistencia al ánodo del LED (la pata más larga).
4. Conecta el cátodo del LED (la pata más corta) al pin de tierra (GND) de la Jetson.
5. Revisa dos veces la polaridad del LED antes de encender la Jetson: conectado al revés simplemente no prende, pero es buena práctica verificar.

### Parte 4: Script de encendido y apagado

1. Crea `led_blink.py`:
   ```python
   import Jetson.GPIO as GPIO
   import time

   LED_PIN = 12  # pin físico en el header de 40 pines (numeración BOARD)

   GPIO.setmode(GPIO.BOARD)
   GPIO.setup(LED_PIN, GPIO.OUT)

   try:
       while True:
           GPIO.output(LED_PIN, GPIO.HIGH)  # encender
           print("LED encendido")
           time.sleep(1)
           GPIO.output(LED_PIN, GPIO.LOW)   # apagar
           print("LED apagado")
           time.sleep(1)
   except KeyboardInterrupt:
       pass
   finally:
       GPIO.cleanup()
   ```
2. Ejecútalo (sin `sudo`, si la Parte 2 quedó bien configurada):
   ```bash
   python3 led_blink.py
   ```
3. Confirma que el LED parpadea cada segundo, y detén el script con `Ctrl+C`.

### Parte 5: Extensión (control manual)

1. Modifica el script para que, en vez de parpadear solo, le pida al usuario qué hacer. Crea `led_manual.py`:
   ```python
   import Jetson.GPIO as GPIO

   LED_PIN = 12

   GPIO.setmode(GPIO.BOARD)
   GPIO.setup(LED_PIN, GPIO.OUT)

   try:
       while True:
           accion = input("Escribe 'on', 'off' o 'salir': ").strip().lower()
           if accion == "on":
               GPIO.output(LED_PIN, GPIO.HIGH)
           elif accion == "off":
               GPIO.output(LED_PIN, GPIO.LOW)
           elif accion == "salir":
               break
           else:
               print("Opción no válida")
   finally:
       GPIO.cleanup()
   ```
2. Ejecútalo y comprueba que puedes encender y apagar el LED a demanda, no solo en un ciclo automático.

## Entregable

Un reporte breve (PDF o Markdown) que incluya:

1. Una foto o video corto del LED parpadeando con `led_blink.py`.
2. El código de `led_manual.py` y evidencia (captura de terminal) de haberlo usado para encender y apagar el LED manualmente.
3. Respuesta a las preguntas de reflexión.

## Preguntas de reflexión

1. ¿Por qué hubiera sido riesgoso cambiar el comando `python` del sistema para que apuntara a Python 3, en vez de solo usar `python3` explícitamente?
2. ¿Qué le pasaría al LED (o al pin GPIO) si se conecta el circuito sin la resistencia limitadora de corriente?

## Rúbrica sugerida

| Criterio                                                      | Puntos |
|-----------------------------------------------------------------|--------|
| Python 3 y `pip3` verificados/instalados correctamente            | 15     |
| `Jetson.GPIO` instalado y permisos configurados (sin depender de `sudo`) | 20     |
| Circuito armado correctamente (evidencia del LED parpadeando)     | 25     |
| Script de control manual funcionando                               | 20     |
| Calidad de las respuestas de reflexión                             | 20     |
| **Total**                                                        | **100**|
