# Seguidor de Línia - UdA

<p align="center">
  <img src="https://github.com/user-attachments/assets/bffecef4-5e87-4c50-b168-953269a5a0ee" alt="Seguidor de Línia" width="100%" />
</p>

Robot seguidor de línia autònom controlat per algoritme PID, desenvolupat com a projecte a la **Universitat d'Andorra (UdA)**. El sistema utilitza una targeta **Arduino**, un matriu de sensors de reflectància **Pololu QTR-6A** i un controlador de motors **DFRobot MD1.3 2A Dual Motor Controller** per executar un seguiment fluid de la trajectòria i senyalització mitjançant LEDs intermitents.

---

## Especificacions de Hardware i Connexions

| Component | Descripció | Connexió de Pins (Arduino) |
| :--- | :--- | :--- |
| **Microcontrolador** | Arduino (Uno / Nano / Mega) | — |
| **Array de Sensors** | Pololu QTR-6A Reflectance Array | Pins Analògics `A0` a `A5` |
| **Controlador Motors** | DFRobot MD1.3 2A Dual Motor Controller (`DRI0002`) | — |
| **Motor Esquerre (PWM)**| Control de Velocitat Enable (`E1`) | `D6` |
| **Motor Esquerre (Dir)**| Control de Direcció (`M1`) | `D7` |
| **Motor Dret (PWM)**   | Control de Velocitat Enable (`E2`) | `D5` |
| **Motor Dret (Dir)**   | Control de Direcció (`M2`) | `D4` |
| **LEDs Intermitents**   | Senyal Esquerra (`L_ESQUERRA`) | `D9` |
| **LEDs Intermitents**   | Senyal Dreta (`L_DRETA`) | `D8` |

---

## Implementació del Control PID

El robot utilitza un algoritme Proporcional-Integral-Derivatiu (PID) per calcular els ajustos de direcció i velocitat basant-se en la posició de la línia proporcionada per la llibreria `QTRSensors`:

$$\text{Error} = \text{qtra.readLine}(\text{sensors}) - 2500$$

* **Valor Central d'Objectiu:** `2500` (dada la lectura de 6 sensors en un rang de `0` a `5000`).
* **Fórmula de Correcció:**

$$\text{Ajust} = (K_P \times e) + (K_D \times \Delta e) + (K_I \times \sum e)$$

### Constants d'Ajust (Tuning)

```cpp
#define KP 0.04  // Pes Proporcional
#define KI 0.1   // Pes Integral
#define KD 0.2   // Pes Derivatiu
```

 Compensació de Desviament de Motors

Per corregir les diferències físiques de potència entre els dos motors DC, es defineixen velocitats de base independents:

* **`VEL_MIN`** (Velocitat Base Motor Esquerre): `180`
* **`VEL_MIN_E`** (Velocitat Base Motor Dret - Correcció): `197`
* **`VEL_MAX`** (Límit Màxim PWM): `255`

---

##  Estructura del Codi

```plaintext
.
├── seguidor_linea.ino    # Sketch principal d'Arduino amb setup, loop, PID i control de motors
└── README.md             # Documentació del projecte
```

---

## Flux de Funcionament

1. **Rutina de Calibració (`setup`):** Executa un cicle de calibració de sensors de ~7.65 segons (`qtra.calibrate(QTR_EMITTERS_ON)`). Cal moure el robot manualment d'un costat a l'altre sobre la línia negra durant l'inici.
2. **Lectura i PID (`loop`):** Normalitza l'error de lectura analògica i calcula les velocitats PWM ajustades enviades a `ajustarVel()`.
3. **Indicadors Intermitents (`intermitents`):**
   * **Línia Centrada** (`-1000 <= error <= 1000`): Tots dos LEDs ON.
   * **Línia a la Dreta** (`error > 1000`): LED Esquerre ON, LED Dret OFF.
   * **Línia a l'Esquerra** (`error < -1000`): LED Esquerre OFF, LED Dret ON.

---

## Com Començar

### Requisits Previs
* **Arduino IDE** (v1.8+ o v2.0+)
* **Llibreria Pololu QTRSensors** (Instal·lable des del Cercador de Llibreries de l'Arduino IDE o des de GitHub)

### Instal·lació i Càrrega

1. **Clona aquest repositori:**
   ```bash
   git clone https://github.com/el-teu-usuari/uda-seguidor-linea.git
   ```

2. **Obre el fitxer** `seguidor_linea.ino` a l'Arduino IDE.
3. **Connecta la placa Arduino** mitjançant USB, selecciona el port i el tipus de placa corresponent a **Eines**.
4. **Prem Pujar** (`Ctrl + U` / `Cmd + U`).
5. En reiniciar la placa, passa immediatament la barra de sensors d'un costat a l'altre per sobre de la línia negra durant els 7.6 segons de calibració inicial.
