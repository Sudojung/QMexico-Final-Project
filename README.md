# Matching 4×4 como QUBO y QAOA local

Proyecto final para QMexico Summer School 2026.

El objetivo del proyecto es formular un problema de emparejamiento bipartito de tamaño 4×4 como un QUBO, validarlo con un método clásico exacto y ejecutar una versión local con QAOA.

## Estructura del repositorio

```text
.
├── data/
│   └── dataset_real_4x4.csv
├── proyecto_qubo_qaoa.ipynb
└── README.md
```

## Dataset usado

El dataset es semi-real y educativo. Modela una asignación académica entre cuatro perfiles sintéticos de estudiantes/equipos y cuatro proyectos posibles relacionados con computación cuántica.

- Lado A: perfiles de estudiantes o equipos.
- Lado B: proyectos finales posibles.
- Tamaño: 4×4.
- Archivo: `data/dataset_real_4x4.csv`.

No se usan nombres reales ni datos personales. Los perfiles son sintéticos y solo representan combinaciones plausibles de habilidades en una escuela de verano de computación cuántica.

## Lado A: estudiantes/equipos

| ID | Perfil |
|---|---|
| A1 | Estudiante_Quantum_Software |
| A2 | Estudiante_Data_Optimization |
| A3 | Estudiante_Cybersecurity_PQC |
| A4 | Estudiante_Research_Communication |

## Lado B: proyectos

| ID | Proyecto |
|---|---|
| B1 | QAOA_para_matching |
| B2 | QML_para_datos_pequenos |
| B3 | Criptografia_postcuantica_intro |
| B4 | Divulgacion_quantum_labs |

## Construcción del score

Cada estudiante y cada proyecto se describen con seis dimensiones en escala de 1 a 5:

- Python
- Álgebra lineal
- Optimización
- Análisis de datos
- Ciberseguridad
- Comunicación

El score `S_ij` mide qué tan compatible es el estudiante `A_i` con el proyecto `B_j`. La fórmula usada es:

```text
S_ij = sum_k w_k * (10 - 2 * abs(a_ik - b_jk))
```

Los pesos usados son:

| Dimensión | Peso |
|---|---:|
| Python | 0.22 |
| Álgebra lineal | 0.18 |
| Optimización | 0.22 |
| Análisis de datos | 0.16 |
| Ciberseguridad | 0.12 |
| Comunicación | 0.10 |

Una mayor coincidencia entre habilidades del estudiante y requerimientos del proyecto produce un score más alto.

## Matriz de compatibilidad

| A \ B | B1 | B2 | B3 | B4 |
|---|---:|---:|---:|---:|
| A1 | 8.48 | 8.12 | 7.28 | 7.68 |
| A2 | 8.36 | 8.40 | 6.12 | 6.52 |
| A3 | 5.76 | 6.28 | 10.00 | 8.32 |
| A4 | 6.72 | 7.24 | 7.76 | 8.96 |

## Formulación del problema

La variable binaria es:

```text
x_ij = 1 si el estudiante/equipo A_i se asigna al proyecto B_j
x_ij = 0 en caso contrario
```

El objetivo original es maximizar el score total:

```text
max sum_i sum_j S_ij x_ij
```

Sujeto a restricciones de asignación uno-a-uno:

```text
sum_j x_ij = 1 para cada estudiante A_i
sum_i x_ij = 1 para cada proyecto B_j
```

## Formulación QUBO

Para usar QUBO, el problema se transforma en una minimización:

```text
E(x) = - sum_i sum_j S_ij x_ij
       + lambda_A * sum_i (sum_j x_ij - 1)^2
       + lambda_B * sum_j (sum_i x_ij - 1)^2
```

El primer término premia asignaciones con score alto. Los términos cuadráticos penalizan violaciones de las restricciones de fila y columna.

## Validación clásica

El notebook valida el QUBO revisando las 2^16 configuraciones binarias posibles y también comparando contra las 24 permutaciones factibles del problema de asignación.

La mejor asignación esperada por el score es:

| Estudiante/equipo | Proyecto | Score |
|---|---|---:|
| A1 | B1 | 8.48 |
| A2 | B2 | 8.40 |
| A3 | B3 | 10.00 |
| A4 | B4 | 8.96 |

Score total clásico óptimo:

```text
35.84
```

## QAOA local

El notebook ejecuta una simulación local de QAOA con `numpy` y `scipy`. Se usa un Hamiltoniano de costo derivado del QUBO y un mixer estándar.

Como el mixer estándar no preserva automáticamente las restricciones de matching, el notebook mide:

- mejor energía observada,
- factibilidad de las muestras,
- probabilidad del óptimo clásico,
- comparación contra la solución clásica exacta.

La ejecución en hardware real de IBM Quantum se deja desactivada porque es opcional para la entrega base.

## Interpretación

La solución clásica óptima asigna cada perfil al proyecto más coherente con sus habilidades principales:

- A1 se asigna a QAOA para matching porque combina Python, álgebra lineal y programación cuántica.
- A2 se asigna a QML para datos pequeños porque tiene mejor ajuste con análisis de datos y optimización.
- A3 se asigna a criptografía postcuántica porque prioriza ciberseguridad.
- A4 se asigna a divulgación y laboratorios porque tiene mayor peso en comunicación e investigación.

El resultado debe interpretarse solo como una demostración educativa de modelado QUBO y QAOA local, no como una herramienta real de asignación académica.

## Limitaciones y ética

- El dataset es pequeño y semi-real.
- Los perfiles son sintéticos.
- La escala de habilidades es simplificada.
- La matriz 4×4 sirve para validar el pipeline, no para tomar decisiones reales.
- No se usan datos personales ni sensibles.
- En un caso real se necesitarían más variables, validación institucional y revisión ética.

## Archivos principales

- `proyecto_qubo_qaoa.ipynb`: notebook final con QUBO, validación clásica y QAOA local.
- `data/dataset_real_4x4.csv`: dataset semi-real usado por el notebook.
- `README.md`: explicación del dataset, formulación e interpretación.

