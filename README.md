

---

## Tabla de contenidos

1. [Descripción general](#1-descripción-general)
2. [Objetivos](#2-objetivos)
3. [Estructura del repositorio](#3-estructura-del-repositorio)
4. [Requisitos](#4-requisitos)
5. [Punto 1: Teclado I²C + brazo robótico en PyBullet dibujando](#5-punto-1-teclado-i²c--simulación-de-brazo-robótico-en-pybullet-dibujando)
6. [Punto 2: Reconocimiento de dígitos con OpenCV + OLED I²C + SPI](#6-punto-2-reconocimiento-de-dígitos-escritos-a-mano-con-opencv--pantalla-oled-i²c--comunicación-spi)
7. [Protocolos de comunicación](#7-protocolos-de-comunicación)
8. [Resultados](#8-resultados)
9. [Pruebas realizadas](#9-pruebas-realizadas)
10. [Problemas frecuentes y soluciones](#10-problemas-frecuentes-y-soluciones)
11. [Limitaciones y trabajo futuro](#11-limitaciones-y-trabajo-futuro)
12. [Conclusiones](#12-conclusiones)
13. [Referencias](#13-referencias)
14. [Créditos y licencia](#14-créditos-y-licencia)

---

## 1. Descripción general

Este proyecto integra **hardware embebido (ESP32)**, **simulación robótica** y **visión por computador con aprendizaje profundo**, dividido en dos puntos independientes:

| Punto | Tema | Tecnologías principales |
|-------|------|--------------------------|
| **1** | Teclado matricial 4x4 + pantalla LCD I²C + simulación de un brazo robótico que dibuja el dígito presionado | ESP32, Wokwi, I²C, LCD 16x2, Python, PyBullet |
| **2** | Reconocimiento de dígitos escritos a mano con cámara, enviados por serie a un ESP32 maestro SPI, que los transmite a un ESP32 esclavo que los muestra en una OLED I²C | OpenCV, TensorFlow/Keras (CNN), UART, SPI, OLED SSD1306 I²C, ESP32 |

---

## 2. Objetivos

### Objetivo general
Desarrollar un sistema que combine la lectura de periféricos por I²C, la simulación de un brazo robótico, la visión artificial y la comunicación entre microcontroladores por SPI.

### Objetivos específicos

**Punto 1**
- Leer un teclado matricial 4x4 con un ESP32 y mostrar la tecla presionada en una LCD 16x2 con interfaz I²C.
- Enviar la tecla al computador por puerto serie.
- Simular en PyBullet un brazo robótico que reciba la tecla y **dibuje el dígito correspondiente** sobre un lienzo.

**Punto 2**
- Capturar imágenes de dígitos escritos a mano con la cámara del PC.
- Preprocesar la imagen con OpenCV (escala de grises, filtrado, umbralización, recorte y normalización).
- Clasificar el dígito con una red neuronal convolucional (CNN).
- Enviar el resultado por puerto serie al **ESP-A (maestro SPI)**, que lo reenvía por SPI al **ESP-B (esclavo SPI)**, el cual lo muestra en una **pantalla OLED I²C**.

---

## 3. Estructura del repositorio

```
.
├── README.md
├── requirements.txt
├── docs/
│   ├── img/                     # Capturas de pantalla y diagramas
│   └── informe.pdf              # (opcional) informe de la práctica
├── punto1_teclado_pybullet/
│   ├── esp32_teclado/
│   │   ├── sketch.ino           # Código del ESP32 (teclado + LCD I²C + serial)
│   │   ├── diagram.json         # Circuito de Wokwi
│   │   └── libraries.txt        # Librerías usadas en Wokwi
│   └── pybullet_brazo/
│       ├── brazo_dibujo.py      # Simulación y control del brazo
│       ├── trayectorias.py      # Trazos de los dígitos 0-9
│       └── brazo.urdf           # Modelo del brazo (si aplica)
├── punto2_reconocimiento_digitos/
│   ├── python/
│   │   ├── entrenar_modelo.py   # Entrenamiento de la CNN
│   │   ├── reconocer_digitos.py # Cámara + OpenCV + CNN + envío serial
│   │   └── modelo_digitos.h5    # Modelo entrenado
│   ├── esp_a_maestro/
│   │   └── esp_a_maestro.ino    # ESP32 maestro SPI
│   └── esp_b_esclavo/
│       └── esp_b_esclavo.ino    # ESP32 esclavo SPI + OLED I²C
└── videos/
    └── demo.mp4                 # (opcional) video de demostración
```

> Ajusta los nombres de carpetas y archivos a los de tu entrega real.

---

## 4. Requisitos

### 4.1 Software

| Herramienta | Versión sugerida | Uso |
|-------------|------------------|-----|
| Python | 3.9 – 3.11 | Scripts de PyBullet y visión |
| PyBullet | ≥ 3.2 | Simulación del brazo robótico |
| OpenCV (`opencv-python`) | ≥ 4.8 | Captura y preprocesamiento |
| TensorFlow / Keras | ≥ 2.12 | CNN |
| NumPy | ≥ 1.23 | Cálculo numérico |
| pySerial | ≥ 3.5 | Comunicación serial PC ↔ ESP32 |
| Arduino IDE | ≥ 2.x | Programación de los ESP32 |
| Wokwi (simulador online o extensión VS Code) | Última | Simulación de circuitos |
| Paquete de placas ESP32 (Espressif) | ≥ 2.0 | Compilación |

### 4.2 Librerías de Arduino

- `Keypad` (Mark Stanley / Alexander Brevig)
- `LiquidCrystal_I2C`
- `Adafruit_GFX`
- `Adafruit_SSD1306`
- `ESP32DMASPISlave` (para el ESP32 en modo esclavo SPI)
- `SPI` (incluida en el core)

### 4.3 Hardware (o su equivalente simulado en Wokwi)

- 3 × ESP32 DevKit (1 en el punto 1; 2 en el punto 2: ESP-A y ESP-B)
- 1 × Teclado matricial 4x4
- 1 × LCD 16x2 con módulo I²C (PCF8574)
- 1 × Pantalla OLED SSD1306 128x64 I²C
- 1 × Cámara web (la del PC)
- Cables Dupont / cables USB

### 4.4 Instalación

```bash
# 1. Clonar el repositorio
git clone [URL_DEL_REPOSITORIO]
cd [NOMBRE_DEL_REPOSITORIO]

# 2. Crear entorno virtual
python -m venv venv
source venv/bin/activate        # Linux / macOS
venv\Scripts\activate           # Windows

# 3. Instalar dependencias
pip install -r requirements.txt
```

**`requirements.txt` sugerido:**

```
numpy
opencv-python
tensorflow
pybullet
pyserial
matplotlib
```

---

## 5. Punto 1: Teclado I²C + simulación de brazo robótico en PyBullet dibujando

### 5.1 Descripción

Un ESP32 lee un teclado matricial 4x4. Cada tecla presionada se muestra en una LCD 16x2 (bus I²C) y se envía por el puerto serie al computador. Un script en Python recibe el carácter y ordena a un brazo robótico simulado en PyBullet que **dibuje el dígito** sobre un lienzo blanco (el "1" de la figura es el resultado de presionar la tecla 1).

### 5.2 Diagrama de flujo

```
 Teclado 4x4 ──► ESP32 ──► LCD 16x2 (I²C)
                   │
                   └── UART (USB) ──► Python ──► PyBullet (brazo) ──► Lienzo (dígito dibujado)
```

### 5.3 Conexiones (referencia; ajústalas a tu `diagram.json`)

**Teclado 4x4 → ESP32**

| Pin del teclado | GPIO ESP32 |
|-----------------|-----------|
| R1 | 19 |
| R2 | 18 |
| R3 | 5 |
| R4 | 17 |
| C1 | 16 |
| C2 | 4 |
| C3 | 0 |
| C4 | 2 |

**LCD 16x2 I²C → ESP32**

| LCD (PCF8574) | ESP32 |
|---------------|-------|
| GND | GND |
| VCC | 5V / VIN |
| SDA | GPIO 21 |
| SCL | GPIO 22 |

> Dirección I²C típica de la LCD: `0x27` (o `0x3F`). Verifícala con un escáner I²C.

### 5.4 Firmware del ESP32 (fragmento de referencia)

```cpp
#include <Keypad.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

const byte FILAS = 4, COLUMNAS = 4;
char teclas[FILAS][COLUMNAS] = {
  {'1','2','3','A'},
  {'4','5','6','B'},
  {'7','8','9','C'},
  {'*','0','#','D'}
};
byte pinesFilas[FILAS]       = {19, 18, 5, 17};
byte pinesColumnas[COLUMNAS] = {16, 4, 0, 2};

Keypad teclado = Keypad(makeKeymap(teclas), pinesFilas, pinesColumnas, FILAS, COLUMNAS);
LiquidCrystal_I2C lcd(0x27, 16, 2);

void setup() {
  Serial.begin(115200);
  lcd.init();
  lcd.backlight();
  lcd.setCursor(0, 0);
  lcd.print("Tecla:");
}

void loop() {
  char tecla = teclado.getKey();
  if (tecla) {
    lcd.setCursor(0, 1);
    lcd.print("                ");   // limpiar línea
    lcd.setCursor(0, 1);
    lcd.print(tecla);
    Serial.println(tecla);           // envío al PC
  }
}
```

### 5.5 Simulación en PyBullet

**Modelo del brazo (según la simulación):** una base cilíndrica gris, un eslabón cian, un eslabón naranja y un efector final (pinza/lápiz) en la punta que se desplaza sobre el plano del lienzo.

**Lógica del script `brazo_dibujo.py`:**

1. Conectar a PyBullet (`p.connect(p.GUI)`), cargar plano y modelo del brazo.
2. Abrir el puerto serie y leer las teclas enviadas por el ESP32.
3. Para cada dígito (`0–9`), obtener su **trayectoria** (lista de puntos 2D normalizados).
4. Transformar los puntos 2D al plano de trabajo 3D del lienzo.
5. Calcular la **cinemática inversa** con `p.calculateInverseKinematics` y mover las articulaciones con `p.setJointMotorControlArray`.
6. Registrar cada posición del efector final y **dibujar el trazo** (con `p.addUserDebugLine` y/o sobre un lienzo 2D en OpenCV/Matplotlib).
7. Al terminar el trazo, levantar el efector y esperar la siguiente tecla.

**Esqueleto de referencia:**

```python
import time
import serial
import pybullet as p
import pybullet_data
from trayectorias import TRAYECTORIAS   # dict: '0'..'9' -> lista de (x, y)

PUERTO = "COM3"        # Linux: /dev/ttyUSB0
BAUDIOS = 115200

p.connect(p.GUI)
p.setAdditionalSearchPath(pybullet_data.getDataPath())
p.setGravity(0, 0, -9.81)
p.loadURDF("plane.urdf")
brazo = p.loadURDF("brazo.urdf", useFixedBase=True)
EFECTOR = p.getNumJoints(brazo) - 1

def mover_a(x, y, z):
    joints = p.calculateInverseKinematics(brazo, EFECTOR, [x, y, z])
    for i, q in enumerate(joints):
        p.setJointMotorControl2(brazo, i, p.POSITION_CONTROL, q)
    for _ in range(20):
        p.stepSimulation()
        time.sleep(1/240)

def dibujar_digito(d, z_lienzo=0.05, escala=0.1):
    puntos = TRAYECTORIAS[d]
    prev = None
    for (u, v) in puntos:
        x, y = u * escala, v * escala
        mover_a(x, y, z_lienzo)
        pos = p.getLinkState(brazo, EFECTOR)[0]
        if prev:
            p.addUserDebugLine(prev, pos, [0, 0, 0], 2, 0)
        prev = pos
    mover_a(*prev[:2], z_lienzo + 0.05)  # levantar lápiz

with serial.Serial(PUERTO, BAUDIOS, timeout=1) as ser:
    while True:
        linea = ser.readline().decode(errors="ignore").strip()
        if linea in TRAYECTORIAS:
            dibujar_digito(linea)
```

### 5.6 Ejecución

1. Abrir el proyecto en Wokwi y arrancar la simulación (o cargar el firmware al ESP32 físico).
2. Conectar el puerto serie al PC (en Wokwi puede usarse el puente RFC2217, `wokwi-cli` o un puerto virtual).
3. Ejecutar:
   ```bash
   cd punto1_teclado_pybullet/pybullet_brazo
   python brazo_dibujo.py
   ```
4. Presionar una tecla numérica en el teclado: la LCD la muestra y el brazo dibuja el dígito.

### 5.7 Resultado esperado

- La LCD muestra la tecla presionada.
- El brazo se mueve en PyBullet y traza el dígito (por ejemplo, el **1**) sobre el lienzo blanco.

---

## 6. Punto 2: Reconocimiento de dígitos escritos a mano con OpenCV + pantalla OLED I²C + comunicación SPI

### 6.1 Descripción

Se captura con la cámara del PC un dígito escrito a mano, se preprocesa con OpenCV, se clasifica con una CNN y el resultado se envía por puerto serie a un ESP32 **maestro SPI (ESP-A)**. Este lo transmite por **SPI** a un segundo ESP32 **esclavo (ESP-B)**, que lo muestra en una **OLED I²C**.

### 6.2 Esquema de desarrollo

```
Cámara PC ─► Preprocesamiento OpenCV ─► Reconocimiento CNN ─► Envío por puerto serie
                                                                      │
                                                                      ▼
                                                        ESP-A (MAESTRO SPI)
                                                                      │  SPI
                                                                      ▼
                                                        ESP-B (ESCLAVO SPI)
                                                                      │
                                                                      ▼
                                                             Mostrar en OLED I²C
```

### 6.3 Preprocesamiento con OpenCV

Pasos aplicados a cada fotograma:

1. **Captura** con `cv2.VideoCapture(0)`.
2. **ROI (región de interés):** recuadro verde centrado en pantalla donde se coloca el dígito.
3. **Escala de grises:** `cv2.cvtColor(..., cv2.COLOR_BGR2GRAY)`.
4. **Filtro gaussiano:** `cv2.GaussianBlur` para reducir ruido.
5. **Umbralización** (adaptativa u Otsu) e **inversión** para obtener trazo blanco sobre fondo negro (formato MNIST).
6. **Operaciones morfológicas** (apertura/cierre) para limpiar el trazo.
7. **Recorte al contorno** del dígito y **relleno cuadrado** manteniendo la proporción.
8. **Redimensionado a 28×28** y **normalización** (`/255.0`).
9. Se muestra la ventana "Digito" con la imagen procesada (la miniatura 28×28 de la figura).

```python
import cv2
import numpy as np

def preprocesar(roi_bgr):
    gris = cv2.cvtColor(roi_bgr, cv2.COLOR_BGR2GRAY)
    gris = cv2.GaussianBlur(gris, (5, 5), 0)
    _, bin_inv = cv2.threshold(gris, 0, 255, cv2.THRESH_BINARY_INV + cv2.THRESH_OTSU)
    kernel = np.ones((3, 3), np.uint8)
    bin_inv = cv2.morphologyEx(bin_inv, cv2.MORPH_OPEN, kernel)

    contornos, _ = cv2.findContours(bin_inv, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)
    if not contornos:
        return None
    x, y, w, h = cv2.boundingRect(max(contornos, key=cv2.contourArea))
    digito = bin_inv[y:y+h, x:x+w]

    lado = max(w, h) + 20
    lienzo = np.zeros((lado, lado), np.uint8)
    ox, oy = (lado - w) // 2, (lado - h) // 2
    lienzo[oy:oy+h, ox:ox+w] = digito

    img = cv2.resize(lienzo, (28, 28), interpolation=cv2.INTER_AREA)
    return img.astype("float32").reshape(1, 28, 28, 1) / 255.0
```

### 6.4 Red neuronal convolucional (CNN)

**Dataset:** MNIST (60 000 imágenes de entrenamiento y 10 000 de prueba), con posible aumento de datos (rotación, desplazamiento, zoom) para mejorar la robustez ante escritura real.

**Arquitectura de referencia:**

| Capa | Configuración |
|------|---------------|
| Entrada | 28×28×1 |
| Conv2D | 32 filtros, 3×3, ReLU |
| MaxPooling2D | 2×2 |
| Conv2D | 64 filtros, 3×3, ReLU |
| MaxPooling2D | 2×2 |
| Flatten | |
| Dense | 128 (ReLU) |
| Dropout | 0.3 – 0.5 |
| Dense | 10 (Softmax) |

**Compilación:** optimizador `adam`, pérdida `sparse_categorical_crossentropy`, métrica `accuracy`.

```python
import tensorflow as tf
from tensorflow.keras import layers, models

(x_tr, y_tr), (x_te, y_te) = tf.keras.datasets.mnist.load_data()
x_tr = x_tr[..., None] / 255.0
x_te = x_te[..., None] / 255.0

modelo = models.Sequential([
    layers.Input((28, 28, 1)),
    layers.Conv2D(32, 3, activation="relu"),
    layers.MaxPooling2D(),
    layers.Conv2D(64, 3, activation="relu"),
    layers.MaxPooling2D(),
    layers.Flatten(),
    layers.Dense(128, activation="relu"),
    layers.Dropout(0.4),
    layers.Dense(10, activation="softmax"),
])
modelo.compile(optimizer="adam",
               loss="sparse_categorical_crossentropy",
               metrics=["accuracy"])
modelo.fit(x_tr, y_tr, epochs=10, validation_data=(x_te, y_te))
modelo.save("modelo_digitos.h5")
```

### 6.5 Programa principal en Python (`reconocer_digitos.py`)

```python
import cv2
import numpy as np
import serial
import tensorflow as tf
from preprocesamiento import preprocesar   # función de la sección 6.3

PUERTO = "COM4"
modelo = tf.keras.models.load_model("modelo_digitos.h5")
ser = serial.Serial(PUERTO, 115200, timeout=1)
cap = cv2.VideoCapture(0)
ultimo = None

while True:
    ok, frame = cap.read()
    if not ok:
        break
    h, w = frame.shape[:2]
    x1, y1, x2, y2 = w//2 - 100, h//2 - 100, w//2 + 100, h//2 + 100
    roi = frame[y1:y2, x1:x2]

    entrada = preprocesar(roi)
    if entrada is not None:
        pred = modelo.predict(entrada, verbose=0)[0]
        digito = int(np.argmax(pred))
        confianza = float(pred[digito]) * 100

        cv2.putText(frame, f"Numero: {digito} ({confianza:.1f}%)", (x1, y1 - 10),
                    cv2.FONT_HERSHEY_SIMPLEX, 0.7, (0, 255, 0), 2)
        cv2.imshow("Digito", (entrada[0, :, :, 0] * 255).astype("uint8"))

        if confianza > 90 and digito != ultimo:      # evita reenvíos repetidos
            trama = bytes([0xAA, digito, 0xAA ^ digito])
            ser.write(trama)
            ultimo = digito

    cv2.rectangle(frame, (x1, y1), (x2, y2), (0, 255, 0), 2)
    cv2.imshow("Reconocimiento de digitos", frame)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

cap.release()
ser.close()
cv2.destroyAllWindows()
```

### 6.6 ESP-A (Maestro SPI)

**Rol:** recibe la trama por UART desde el PC y la envía por SPI al esclavo.

**Conexión SPI (VSPI):**

| Señal | ESP-A (maestro) | ESP-B (esclavo) |
|-------|-----------------|------------------|
| SCK | GPIO 18 | GPIO 18 |
| MISO | GPIO 19 | GPIO 19 |
| MOSI | GPIO 23 | GPIO 23 |
| SS/CS | GPIO 5 | GPIO 5 |
| GND | GND | GND (común) |

```cpp
#include <SPI.h>

const int PIN_SS = 5;

void setup() {
  Serial.begin(115200);
  pinMode(PIN_SS, OUTPUT);
  digitalWrite(PIN_SS, HIGH);
  SPI.begin(18, 19, 23, PIN_SS);
}

void loop() {
  if (Serial.available() >= 3) {
    uint8_t cabecera = Serial.read();
    uint8_t digito   = Serial.read();
    uint8_t crc      = Serial.read();

    if (cabecera == 0xAA && crc == (cabecera ^ digito)) {
      SPI.beginTransaction(SPISettings(1000000, MSBFIRST, SPI_MODE0));
      digitalWrite(PIN_SS, LOW);
      SPI.transfer(cabecera);
      SPI.transfer(digito);
      SPI.transfer(crc);
      digitalWrite(PIN_SS, HIGH);
      SPI.endTransaction();
    }
  }
}
```

### 6.7 ESP-B (Esclavo SPI + OLED I²C)

**Rol:** recibe los bytes por SPI, valida la trama y dibuja el dígito en grande en la OLED.

**OLED SSD1306 → ESP-B**

| OLED | ESP-B |
|------|-------|
| GND | GND |
| VCC | 3V3 |
| SDA | GPIO 21 |
| SCL | GPIO 22 |

> Dirección I²C típica: `0x3C`.

```cpp
#include <Wire.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>
#include <ESP32DMASPISlave.h>

Adafruit_SSD1306 oled(128, 64, &Wire, -1);
ESP32DMASPI::Slave slave;

uint8_t* rx_buf;
uint8_t* tx_buf;

void mostrarDigito(int d) {
  oled.clearDisplay();
  oled.setTextSize(6);
  oled.setTextColor(SSD1306_WHITE);
  oled.setCursor(45, 8);
  oled.print(d);
  oled.display();
}

void setup() {
  Serial.begin(115200);
  Wire.begin(21, 22);
  oled.begin(SSD1306_SWITCHCAPVCC, 0x3C);
  oled.clearDisplay();
  oled.display();

  rx_buf = slave.allocDMABuffer(4);
  tx_buf = slave.allocDMABuffer(4);
  slave.setDataMode(SPI_MODE0);
  slave.setMaxTransferSize(4);
  slave.setQueueSize(1);
  slave.begin(VSPI, 18, 19, 23, 5);   // SCK, MISO, MOSI, SS
}

void loop() {
  slave.queue(NULL, rx_buf, 3);
  slave.trigger();
  if (slave.wait(rx_buf, 3)) {
    if (rx_buf[0] == 0xAA && rx_buf[2] == (rx_buf[0] ^ rx_buf[1])) {
      mostrarDigito(rx_buf[1]);
    }
  }
}
```

### 6.8 Ejecución

1. Cargar `esp_a_maestro.ino` en el ESP-A y `esp_b_esclavo.ino` en el ESP-B.
2. Conectar el ESP-A al PC por USB y anotar el puerto (`COMx` o `/dev/ttyUSBx`).
3. Entrenar el modelo (solo la primera vez):
   ```bash
   python punto2_reconocimiento_digitos/python/entrenar_modelo.py
   ```
4. Ejecutar el reconocimiento:
   ```bash
   python punto2_reconocimiento_digitos/python/reconocer_digitos.py
   ```
5. Colocar un papel con un dígito dentro del recuadro verde. El PC muestra `Numero: X (YY.Y%)` y la OLED del ESP-B muestra el mismo dígito.
6. Presionar **`q`** para salir.

---

## 7. Protocolos de comunicación

| Enlace | Protocolo | Velocidad / Modo | Descripción |
|--------|-----------|------------------|-------------|
| Teclado → ESP32 (Punto 1) | Matriz GPIO | n/a | Escaneo de filas y columnas |
| ESP32 → LCD (Punto 1) | I²C | 100 kHz | Dirección `0x27` |
| ESP32 → PC (Punto 1) | UART / USB | 115200 baudios, 8N1 | Envía la tecla + `\n` |
| PC → ESP-A (Punto 2) | UART / USB | 115200 baudios, 8N1 | Trama de 3 bytes |
| ESP-A → ESP-B (Punto 2) | SPI | 1 MHz, modo 0, MSB first | Trama de 3 bytes |
| ESP-B → OLED (Punto 2) | I²C | 100–400 kHz | Dirección `0x3C` |

**Formato de trama (Punto 2):**

| Byte | Contenido | Descripción |
|------|-----------|-------------|
| 0 | `0xAA` | Cabecera |
| 1 | `0–9` | Dígito reconocido |
| 2 | `0xAA ^ dígito` | Checksum (XOR) |

---

## 8. Resultados

> Inserta aquí tus capturas reales. Ejemplos de referencia:

### Punto 1
| Evidencia | Imagen |
|-----------|--------|
| Circuito en Wokwi (teclado + ESP32 + LCD I²C) | `docs/img/p1_circuito.png` |
| Brazo en PyBullet dibujando el "1" | `docs/img/p1_pybullet.png` |
| Lienzo con el dígito dibujado | `docs/img/p1_lienzo.png` |

```markdown
![Circuito Punto 1](docs/img/p1_circuito.png)
![Brazo dibujando](docs/img/p1_pybullet.png)
```

### Punto 2
| Evidencia | Imagen |
|-----------|--------|
| Cámara + preprocesamiento + predicción ("Numero: 0 (100.0%)") | `docs/img/p2_reconocimiento.png` |
| ESP-A maestro y ESP-B esclavo con OLED mostrando "0" | `docs/img/p2_spi_oled.png` |

```markdown
![Reconocimiento](docs/img/p2_reconocimiento.png)
![SPI + OLED](docs/img/p2_spi_oled.png)
```

### Métricas del modelo (completa con tus valores)

| Métrica | Valor |
|---------|-------|
| Precisión en entrenamiento | [XX %] |
| Precisión en validación (MNIST) | [XX %] |
| Precisión en pruebas reales con cámara | [XX %] |
| Épocas | [N] |
| Tiempo de inferencia por fotograma | [XX ms] |

---

## 9. Pruebas realizadas

### Punto 1
| # | Prueba | Resultado esperado | Resultado |
|---|--------|--------------------|-----------|
| 1 | Presionar cada tecla `0–9` | La LCD muestra la tecla | ✅ |
| 2 | Enviar tecla por serie | El PC recibe el carácter | ✅ |
| 3 | Brazo dibuja cada dígito | Trazo reconocible en el lienzo | ✅ |
| 4 | Teclas no numéricas (`A–D`, `*`, `#`) | Se ignoran para el dibujo | ✅ |

### Punto 2
| # | Prueba | Resultado esperado | Resultado |
|---|--------|--------------------|-----------|
| 1 | Dígitos 0–9 escritos a mano | Clasificación correcta con alta confianza | ✅ |
| 2 | Cambio de iluminación | Reconocimiento estable | [ ] |
| 3 | Trama serie PC → ESP-A | ESP-A recibe 3 bytes válidos | ✅ |
| 4 | Transferencia SPI ESP-A → ESP-B | ESP-B recibe la misma trama | ✅ |
| 5 | Visualización OLED | Dígito correcto en pantalla | ✅ |
| 6 | Trama con checksum inválido | Se descarta | ✅ |

> Marca con ✅/❌ según tus resultados reales.

---

## 10. Problemas frecuentes y soluciones

| Problema | Causa probable | Solución |
|----------|----------------|----------|
| La LCD no muestra nada | Dirección I²C incorrecta o contraste | Probar `0x27`/`0x3F`, ajustar el potenciómetro |
| `Access denied` / `could not open port` | Puerto ocupado por otro programa | Cerrar el monitor serie de Arduino o Wokwi |
| La cámara no abre | Índice incorrecto | Probar `VideoCapture(1)` o `VideoCapture(0, cv2.CAP_DSHOW)` |
| Predicciones erróneas | Iluminación o trazo delgado | Escribir con marcador grueso, fondo blanco, buena luz |
| Predice siempre el mismo número | Imagen no invertida (fondo blanco) | Verificar que el trazo quede blanco sobre fondo negro |
| El esclavo SPI no recibe | Pines cruzados o GND no común | Revisar SCK/MOSI/MISO/SS y unir GND |
| OLED en blanco | Dirección `0x3C` errónea o SDA/SCL invertidos | Escáner I²C y revisar cableado |
| El brazo no alcanza el punto | Fuera del espacio de trabajo | Reducir la escala de la trayectoria o acercar el lienzo |
| Errores de TensorFlow con GPU | Versiones incompatibles | Usar `pip install tensorflow-cpu` |

---

## 11. Limitaciones y trabajo futuro

**Limitaciones**
- El modelo entrenado solo con MNIST puede fallar con estilos de escritura muy distintos.
- El reconocimiento es sensible a la iluminación y al contraste del papel.
- La simulación del brazo no modela fricción real del lápiz ni el contacto con el papel.

**Mejoras propuestas**
- Reentrenar con un dataset propio de dígitos capturados con la cámara.
- Detección de múltiples dígitos en una sola imagen (segmentación).
- Ampliar el reconocimiento a letras (EMNIST).
- Implementar un brazo físico con servomotores controlado por el ESP32.
- Agregar comunicación bidireccional SPI (confirmación ACK del esclavo).
- Interfaz gráfica para seleccionar puertos y visualizar métricas.

---

## 12. Conclusiones

- Se integró correctamente el manejo de periféricos por **I²C** (LCD y OLED) con microcontroladores ESP32.
- La combinación de **PyBullet** con datos provenientes de hardware demuestra un flujo de trabajo tipo *hardware-in-the-loop*.
- El pipeline **OpenCV + CNN** permite reconocer dígitos manuscritos en tiempo real con alta confianza cuando el preprocesamiento es adecuado.
- La comunicación **maestro-esclavo por SPI** entre dos ESP32 permitió distribuir tareas (recepción de datos y visualización) de forma eficiente y confiable, apoyada en una trama con cabecera y checksum.
- El proyecto integra visión artificial, robótica y sistemas embebidos en una sola solución.

---

## 13. Referencias

- Documentación de PyBullet: https://pybullet.org
- Documentación de OpenCV: https://docs.opencv.org
- Documentación de TensorFlow/Keras: https://www.tensorflow.org
- Dataset MNIST: http://yann.lecun.com/exdb/mnist/
- Espressif ESP32: https://docs.espressif.com/projects/esp-idf
- Librería Keypad: https://github.com/Chris--A/Keypad
- Librería Adafruit SSD1306: https://github.com/adafruit/Adafruit_SSD1306
- Librería ESP32DMASPISlave: https://github.com/hideakitai/ESP32DMASPI
- Simulador Wokwi: https://wokwi.com

---

## 14. Créditos y licencia

Proyecto desarrollado por **[Tu nombre]** para la asignatura **[Nombre del curso]**.

Licencia: [MIT / Uso académico]
# tarea-6-micros
