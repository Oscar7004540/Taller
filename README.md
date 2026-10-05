# 🤖 Taller de Simulación y Control Robótico

Repositorio correspondiente al taller de simulación, control y comunicación de sistemas robóticos mediante **Python, PyBullet, ESP32 y comunicación TCP/IP**.

El proyecto se divide en tres partes principales:

- 🛩️ **Inciso A — Control de drones**
- 🤖 **Inciso B — Baxter mediante cinemática inversa**
- 🦾 **Inciso C — Control del robot Atlas mediante ESP32**

---

## 📁 Estructura del repositorio

```text
Taller-PyBullet-ESP32/
│
├── DRONES/
│   └── gym-pybullet-drones/
│       ├── examples/
│       ├── tests/
│       ├── gym_pybullet_drones/
│       ├── README.md
│       ├── pyproject.toml
│       └── ...
│
├── ATLAS/
│   ├── baxter_ik_demo.py
│   ├── ...
│   └── archivos del proyecto
│
├── ESP32/
│   ├── codigo_esp32.ino
│   └── ...
│
├── test_pybullet.py
│
└── .gitignore
```

---

# 🛩️ INCISO A — CONTROL DE DRONES

<details>
<summary><strong>▶ Ver explicación del inciso A</strong></summary>

## Objetivo

Desarrollar una simulación de varios drones utilizando **PyBullet** y la biblioteca `gym-pybullet-drones`.

La simulación permite observar el comportamiento de varios drones y controlar sus movimientos mediante controladores PID.

## Tecnologías utilizadas

- Python
- PyBullet
- NumPy
- Matplotlib
- gym-pybullet-drones
- Control PID

## Funcionamiento

El entorno utiliza `CtrlAviary` como entorno de simulación y `DSLPIDControl` para controlar los drones.

La configuración utilizada trabaja con:

```python
DEFAULT_NUM_DRONES = 3
DEFAULT_SIMULATION_FREQ_HZ = 240
DEFAULT_CONTROL_FREQ_HZ = 48
DEFAULT_DURATION_SEC = 12
```

Los tres drones se inicializan con diferentes posiciones y orientaciones.

El programa genera posiciones objetivo y posteriormente el controlador PID calcula la acción necesaria para llevar cada dron hacia el punto correspondiente.

### Control de los drones

El principio general es:

```text
Estado actual del dron
        ↓
Posición objetivo
        ↓
Controlador PID
        ↓
Acción de control
        ↓
PyBullet
        ↓
Nuevo estado del dron
        ↓
Repetir
```

## Código principal

El código utiliza:

```python
from gym_pybullet_drones.utils.enums import DroneModel, Physics
from gym_pybullet_drones.envs.CtrlAviary import CtrlAviary
from gym_pybullet_drones.control.DSLPIDControl import DSLPIDControl
from gym_pybullet_drones.utils.Logger import Logger
```

Se crean los controladores para cada dron:

```python
ctrl = [
    DSLPIDControl(drone_model=drone)
    for i in range(num_drones)
]
```

Durante la simulación se calcula la acción:

```python
action[j, :], _, _ = ctrl[j].computeControlFromState(
    control_timestep=env.CTRL_TIMESTEP,
    state=obs[j],
    target_pos=np.hstack([
        TARGET_POS[wp_counters[j], 0:2],
        INIT_XYZS[j, 2]
    ]),
    target_rpy=INIT_RPYS[j, :]
)
```

Posteriormente se actualiza el punto de navegación:

```python
for j in range(num_drones):
    wp_counters[j] = (
        wp_counters[j] + 1
        if wp_counters[j] < (NUM_WP - 1)
        else 0
    )
```

Esto permite que los drones continúen recorriendo la trayectoria.

## Ejecución

Abrir CMD:

```cmd
cd C:\RealTimeRobot
venv\Scripts\activate
cd ENTREGA\DRONES\gym-pybullet-drones
```

Ejecutar:

```cmd
python examples\drone_esp32.py
```

Si se desea ejecutar el ejemplo PID:

```cmd
python examples\pid.py
```

## Resultados

Durante la ejecución se abre la ventana de PyBullet y se pueden observar los tres drones.

También se generan resultados de la simulación mediante el sistema de logging.

</details>

---

# 🤖 INCISO B — BAXTER

<details>
<summary><strong>▶ Ver explicación del inciso B</strong></summary>

## Objetivo

Simular el robot **Baxter** utilizando PyBullet y controlar el movimiento de sus articulaciones mediante **cinemática inversa**.

La finalidad es conseguir que el brazo del robot alcance diferentes posiciones objetivo dentro del espacio de trabajo.

## Concepto utilizado

La cinemática inversa permite determinar los valores de las articulaciones necesarios para llevar el efector final a una posición determinada.

```text
Posición objetivo
       ↓
Cinemática inversa
       ↓
Ángulos articulares
       ↓
Articulaciones de Baxter
       ↓
Movimiento del brazo
```

## Archivo principal

El archivo principal del inciso es:

```text
ATLAS/baxter_ik_demo.py
```

## Ejecución

Desde CMD:

```cmd
cd C:\RealTimeRobot
venv\Scripts\activate
cd pybullet_robots
python baxter_ik_demo.py
```

Si el archivo se encuentra dentro de otra carpeta:

```cmd
dir /s /b baxter_ik_demo.py
```

Esto permite localizarlo.

## Funcionamiento

El programa:

1. Inicializa PyBullet.
2. Carga el modelo de Baxter.
3. Identifica las articulaciones.
4. Define una posición objetivo.
5. Calcula la cinemática inversa.
6. Envía los valores articulares al robot.
7. Actualiza la simulación.

## Control mediante cinemática inversa

Conceptualmente:

```python
joint_positions = p.calculateInverseKinematics(
    robot_id,
    end_effector_index,
    target_position
)
```

Posteriormente los valores calculados se aplican a las articulaciones del robot.

## Resultado

La ventana de PyBullet muestra el robot Baxter y permite observar el movimiento de su brazo hacia las posiciones objetivo.

</details>

---

# 🦾 INCISO C — CONTROL DEL ROBOT MEDIANTE ESP32

<details>
<summary><strong>▶ Ver explicación del inciso C</strong></summary>

## Objetivo

Desarrollar una consola de comandos mediante una **ESP32** para controlar el movimiento del robot simulado.

La comunicación se realiza mediante una conexión Wi-Fi y TCP/IP.

## Arquitectura

```text
             ┌──────────────┐
             │    ESP32     │
             │              │
             │ Wi-Fi / TCP  │
             └──────┬───────┘
                    │
                    │ Wi-Fi
                    │
                    ▼
             ┌──────────────┐
             │    Python    │
             │   Servidor   │
             │   TCP :5000  │
             └──────┬───────┘
                    │
                    ▼
             ┌──────────────┐
             │    PyBullet  │
             │     Robot    │
             └──────────────┘
```

## Comandos

La consola permite enviar comandos desde la ESP32.

Ejemplo:

```text
A = Punto A
B = Punto B
C = Punto C
S = STOP
1 = Secuencia A → B → C
```

### Punto A

El robot se desplaza hacia la posición definida como punto A.

### Punto B

El robot se desplaza hacia la posición definida como punto B.

### Punto C

El robot se desplaza hacia la posición definida como punto C.

### STOP

Detiene el movimiento del robot.

### Secuencia 1

Ejecuta automáticamente:

```text
A → B → C
```

## Comunicación TCP

El servidor Python utiliza el puerto:

```text
5000
```

La ESP32 se conecta mediante la dirección IP del computador que ejecuta el servidor.

Ejemplo:

```text
192.168.1.108
```

La dirección IP puede cambiar dependiendo de la red Wi-Fi.

## Flujo de comunicación

```text
Usuario
  ↓
ESP32
  ↓
Wi-Fi
  ↓
Servidor Python
  ↓
Comando recibido
  ↓
Control del robot
  ↓
PyBullet
```

## Ejemplo

Si desde la ESP32 se envía:

```text
A
```

el servidor recibe:

```text
COMANDO ESP32: A
```

y posteriormente ejecuta:

```text
>>> COMANDO A
>>> Robot dirigido al punto A
```

## Código de la ESP32

El código de la ESP32 utiliza la biblioteca:

```cpp
#include <WiFi.h>
```

La estructura general es:

```cpp
#include <WiFi.h>

const char* ssid = "NOMBRE_WIFI";
const char* password = "CONTRASEÑA_WIFI";

const char* serverIP = "192.168.1.108";
const int serverPort = 5000;

WiFiClient client;

void setup() {

    Serial.begin(115200);

    WiFi.begin(ssid, password);

    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }

    Serial.println();
    Serial.println("WiFi conectado");
    Serial.print("IP ESP32: ");
    Serial.println(WiFi.localIP());
}

void loop() {

    if (!client.connected()) {

        Serial.println("Conectando con Python...");

        if (client.connect(serverIP, serverPort)) {
            Serial.println("Servidor conectado");
        }
        else {
            Serial.println("No se pudo conectar");
            delay(2000);
            return;
        }
    }

    // Aquí se envían los comandos
    // A, B, C, S o 1
}
```

> **Nota:** los valores de `ssid`, `password` y `serverIP` deben cambiarse según la red utilizada.

</details>

---

# 🖥️ SERVIDOR PYTHON

<details>
<summary><strong>▶ Ver funcionamiento del servidor</strong></summary>

El servidor Python espera una conexión TCP de la ESP32.

El puerto utilizado es:

```text
5000
```

La estructura de funcionamiento es:

```text
Iniciar servidor
      ↓
Esperar ESP32
      ↓
ESP32 se conecta
      ↓
Esperar comando
      ↓
Interpretar comando
      ↓
Ejecutar movimiento
      ↓
Actualizar PyBullet
```

Ejemplo de servidor TCP:

```python
import socket

HOST = "0.0.0.0"
PORT = 5000

server = socket.socket(
    socket.AF_INET,
    socket.SOCK_STREAM
)

server.bind((HOST, PORT))
server.listen(1)

print("SERVIDOR DE ROBOT")
print("Puerto:", PORT)
print("Esperando conexión de la ESP32...")

conn, addr = server.accept()

print("ESP32 CONECTADA:", addr)

while True:

    data = conn.recv(1024)

    if not data:
        break

    command = data.decode().strip().upper()

    print("COMANDO ESP32:", command)

    if command == "A":
        print("Robot dirigido al punto A")

    elif command == "B":
        print("Robot dirigido al punto B")

    elif command == "C":
        print("Robot dirigido al punto C")

    elif command == "S":
        print("STOP")

    elif command == "1":
        print("Secuencia A -> B -> C")

    else:
        print("Comando no válido")
```

</details>

---

# 🧰 INSTALACIÓN

<details>
<summary><strong>▶ Instalación del entorno</strong></summary>

## Python

Se utiliza Python junto con un entorno virtual.

Crear el entorno:

```cmd
python -m venv venv
```

Activarlo:

```cmd
venv\Scripts\activate
```

## PyBullet

Instalar:

```cmd
pip install pybullet
```

## NumPy

```cmd
pip install numpy
```

## Matplotlib

```cmd
pip install matplotlib
```

## gym-pybullet-drones

Entrar a la carpeta:

```cmd
cd DRONES\gym-pybullet-drones
```

Instalar:

```cmd
pip install -e . --no-deps
```

Comprobar:

```cmd
python -c "import gym_pybullet_drones; print('GYM-PYBULLET-DRONES OK')"
```

</details>

---

# ▶️ EJECUCIÓN DEL PROYECTO

<details>
<summary><strong>▶ Abrir los proyectos</strong></summary>

## Drones

```cmd
cd C:\RealTimeRobot
venv\Scripts\activate
cd ENTREGA\DRONES\gym-pybullet-drones
python examples\drone_esp32.py
```

---

## Baxter

```cmd
cd C:\RealTimeRobot
venv\Scripts\activate
cd ENTREGA\ATLAS
python baxter_ik_demo.py
```

---

## ESP32 + Robot

Primero iniciar el servidor Python:

```cmd
cd C:\RealTimeRobot
venv\Scripts\activate
cd ENTREGA\ATLAS
python servidor.py
```

Después:

1. Conectar la ESP32 al computador mediante Wi-Fi.
2. Abrir Arduino IDE.
3. Cargar el programa `.ino`.
4. Abrir el Monitor Serial.
5. Verificar la IP de la ESP32.
6. Comprobar la conexión con el servidor.
7. Enviar los comandos.

</details>

---

# 📡 COMUNICACIÓN ESP32

<details>
<summary><strong>▶ Configuración de red</strong></summary>

La ESP32 y el computador deben estar conectados a la misma red.

El computador debe ejecutar el servidor TCP.

Ejemplo:

```text
PC
IP: 192.168.1.108
Puerto: 5000
```

La ESP32 se conecta a:

```text
192.168.1.108:5000
```

Si la IP cambia, debe actualizarse en el programa de Arduino.

Para consultar la IP del computador en Windows:

```cmd
ipconfig
```

Buscar:

```text
Dirección IPv4
```

</details>

---

# 📊 RESULTADOS

Durante el desarrollo se obtuvieron los siguientes resultados:

### Inciso A

Se logró ejecutar una simulación con tres drones y visualizar su movimiento dentro del entorno PyBullet.

### Inciso B

Se logró cargar el robot Baxter y controlar sus articulaciones mediante cinemática inversa.

### Inciso C

Se estableció comunicación entre:

```text
ESP32 ↔ Wi-Fi ↔ Python ↔ PyBullet
```

permitiendo enviar comandos desde la ESP32 para controlar el robot simulado.

---

# ⚠️ SOLUCIÓN DE PROBLEMAS

<details>
<summary><strong>▶ Problemas frecuentes</strong></summary>

### `No module named pybullet`

Ejecutar:

```cmd
pip install pybullet
```

---

### `No module named gym_pybullet_drones`

Entrar a:

```cmd
cd DRONES\gym-pybullet-drones
```

y ejecutar:

```cmd
pip install -e . --no-deps
```

---

### La ESP32 no conecta

Comprobar:

```text
1. ESP32 conectada a Wi-Fi
2. PC y ESP32 en la misma red
3. IP correcta
4. Puerto 5000
5. Servidor Python ejecutándose
```

Consultar la IP del PC:

```cmd
ipconfig
```

---

### El servidor dice "servidor no disponible"

Primero ejecutar el servidor Python y posteriormente conectar la ESP32.

El orden recomendado es:

```text
1. Ejecutar PyBullet
2. Ejecutar servidor
3. Encender ESP32
4. Conectar ESP32
5. Enviar comandos
```

---

### El robot no se mueve

Comprobar:

```text
Servidor Python
        ↓
ESP32 conectada
        ↓
Comando recibido
        ↓
Función de movimiento
        ↓
PyBullet ejecutándose
```

</details>

---

# 📚 TECNOLOGÍAS

| Tecnología | Utilización |
|---|---|
| Python | Programación y control |
| PyBullet | Simulación física |
| NumPy | Cálculos numéricos |
| Matplotlib | Gráficas |
| gym-pybullet-drones | Simulación de drones |
| ESP32 | Interfaz física de control |
| Wi-Fi | Comunicación inalámbrica |
| TCP/IP | Comunicación ESP32-PC |
| Arduino IDE | Programación de ESP32 |

---

# 👨‍💻 PROYECTO

**Taller de Simulación y Control Robótico**

Proyecto desarrollado utilizando simulación física, control de robots, comunicación TCP/IP y microcontroladores ESP32.

---

## 📌 Nota

Los archivos ejecutables y códigos fuente se encuentran dentro de las carpetas correspondientes a cada inciso.

```text
DRONES/  → Inciso A
ATLAS/   → Incisos B y C
ESP32/   → Código del microcontrolador
```

Para ejecutar correctamente el proyecto se recomienda utilizar un entorno virtual de Python y realizar la instalación de las dependencias indicadas anteriormente.
