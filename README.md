# Electronic Wheel Alignment &mdash; Baja SAE USB

An electronic wheel&#8209;alignment system for the **Baja SAE USB** off&#8209;road vehicle
prototypes (Universidad Simón Bolívar). It measures each wheel's **camber** with an
accelerometer and the **toe** (distances between the wheels) with ultrasonic sensors,
and shows the driver / mechanic what to adjust on a small handheld display.

*Leonardo Ward &mdash; Equipo Baja SAE USB (Divisiones de Electrónica y de Suspensión),
Sartenejas, August 2013.*

## Table of Contents

1. [Why](#why)
2. [What It Measures](#what-it-measures)
3. [System Architecture](#system-architecture)
4. [Camber &mdash; the Accelerometer](#camber--the-accelerometer)
5. [Toe &mdash; the Ultrasonic Sensors](#toe--the-ultrasonic-sensors)
6. [Mechanical Coupling](#mechanical-coupling)
7. [Tests](#tests)
8. [Repository Layout](#repository-layout)
9. [License](#license)

## Why

The Baja team aligned its cars **by hand** &mdash; parallel lines chalked on the floor
(or cones joined with string), then a tape measure to set each wheel parallel to them.
There was no way to read the actual **camber** angle or to check that the four wheels
were symmetric. Commercial electronic aligners (IR heads bolted to a bench, a PC doing
the maths) are far too big and expensive for a student team.

The design targets: cost under 3000&nbsp;Bsf, everything inside 50&times;50&times;50&nbsp;cm,
a simple user interface, and a written guide.

## What It Measures

* **Camber** &mdash; the angle of the wheel plane away from vertical.
* **Toe** &mdash; via the horizontal distances between reference points on the four
  wheels. For a symmetric car the paired distances must match: `D1 = D2`, `D3 = D4`,
  `D5 = D6`.

The theory section of the report also covers kingpin inclination, caster, the Ackermann
condition and the Jeantaud steering trapezoid.

## System Architecture

Five modules:

| Module | Radio | Bus | Role |
| ------ | ----- | --- | ---- |
| **User interface** | XBee (master) | &mdash; | 16&times;2 LCD, push&#8209;buttons, µC. Shows the angles and a simple wheel diagram telling you which way to adjust. |
| **Wheel 1** | XBee (slave) | I²C (master) | accelerometer + ultrasonic sensors + µC; bridges the wheel bus to the UI over XBee |
| **Wheel 2 / 3 / 4** | &mdash; | I²C (slave) | accelerometer + ultrasonic sensors + µC |

The four wheel modules sit on a single 3&#8209;wire bus on the car &mdash; **Vdd, SDA, SCL**
(I²C, with pull&#8209;ups). Wheel&nbsp;1 is the I²C master and also carries the XBee that
talks wirelessly to the handheld UI module. (A CAN&#8209;bus version was sketched first;
the built system uses XBee&nbsp;+&nbsp;I²C.)

Each wheel module has its own **5&nbsp;V and 3.3&nbsp;V regulators**, and every module has
an ICSP header for programming.

## Camber &mdash; the Accelerometer

An **ADXL335** (±3&nbsp;g, 3&#8209;axis) is mounted parallel to the wheel. With the car
stationary the only acceleration is gravity, so the sensor's X/Y/Z outputs give the
wheel plane's tilt relative to vertical.

At 3.3&nbsp;V supply the ADXL335 sits at 0&nbsp;g = 1.65&nbsp;V with a 330&nbsp;mV/g scale.
Read through a 10&#8209;bit ADC:

```
a [m/s²] = ( reading_mV − 1650 ) / 330 × 9.80665
```

## Toe &mdash; the Ultrasonic Sensors

40&nbsp;kHz ultrasonic emitter/detector pairs on the **inner** face of each tyre (the
wave can't pass through the wheel). Time&#8209;of&#8209;flight &times; the speed of sound gives
the distance to the reference point on another wheel; comparing the symmetric pairs
tells you the toe error. Ultrasonic was chosen over laser: cheaper, smaller, and no
separate receiver structure to mount.

## Mechanical Coupling

SolidWorks parts and assemblies to attach the electronics to the car:

* a clamp (*pinza*) with a torsion spring that grips each tyre;
* an articulated **link&#8209;chain arm** that references a longitudinal axis of the car;
* tube&#8209;axle and end&#8209;cap parts, and the base plates.

## Tests

* **ADXL335** &mdash; bench&#8209;tested in the datasheet's three reference orientations. The
  raw sensor showed a noticeable zero offset and scale error (e.g. ~1.3&nbsp;g read
  where 1&nbsp;g was expected on one axis), so a per&#8209;axis calibration is needed
  before the angle is trustworthy. Results are in
  `Resultados Prueba de Acelerómetros.xlsx`.
* Ultrasonic driver circuits were prototyped in **Multisim** (`*.ms12`).

## Repository Layout

```
README.md
LICENSE                                        MIT
Sistema de Alineación/
  Informe Sistema de Alineación.docx           main report (theory, design, tests)
  Acople mecánico… / Explicación ultrasonido…  supporting docs
  Árbol de Funciones y Tormenta de Ideas CAN…  function tree + early CAN concept
  Pruebas del Acelerómetro… .docx / .xlsx      camber sensor tests
  Componentes… / Primeras Compras… .xlsx       BOM and purchasing
  Receptor.ms12 / Sistema de alineación alterno*.ms12   Multisim schematics
  Acople con eje longitudinal/ , Acople con las Ruedas/  SolidWorks CAD + tutorials
  *.jpg  (module diagrams, hardware photos, the car on the alignment jig)
  datasheets: adxl335, MB1010, SRF004, P113, TL082, UA741, LM137, MAX232, …
```

## License

MIT &mdash; see [`LICENSE`](LICENSE).
