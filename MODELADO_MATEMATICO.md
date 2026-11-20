# 3.5 Modelado Matemático de Requerimientos Nutricionales

## Descripción del Sistema

El módulo de cálculo nutricional implementa un modelo matemático basado en fórmulas clínicas validadas internacionalmente (Harris-Benedict, IPAQ, ASPEN/ESPEN, protocolo EsSalud) para determinar automáticamente los requerimientos energéticos y de macronutrientes individualizados de cada paciente con cáncer de próstata.

El modelo integra tres componentes principales que se calculan secuencialmente:

1. **Requerimiento energético total** (kcal/día)
2. **Requerimientos de macronutrientes** (proteína, carbohidratos, grasas en gramos/día)
3. **Requerimientos de micronutrientes** (RDA según edad)

---

## Ecuaciones del Modelo

### A. Cálculo de Energía Total

La energía diaria requerida se determina mediante la ecuación:

```
Energía (kcal/día) = TMB × FA × FE
```

Donde:
- **TMB**: Tasa Metabólica Basal (kcal/día)
- **FA**: Factor de Actividad (adimensional)
- **FE**: Factor de Estrés (adimensional)

---

#### A.1 Tasa Metabólica Basal (Harris-Benedict para hombres)

```
TMB = 66.47 + (13.75 × P) + (5.0 × T) - (6.76 × E)
```

Donde:
- **P**: Peso corporal (kg)
- **T**: Talla (cm)
- **E**: Edad (años)

**Ajuste por fiebre:**

```
Si temperatura > 37°C:
TMB_ajustada = TMB × [1 + 0.10 × (temperatura - 37)]
```

Ejemplo:
- Paciente con 39°C: TMB_ajustada = TMB × 1.20 (+20%)
- Paciente con 40°C: TMB_ajustada = TMB × 1.30 (+30%)

---

#### A.2 Factor de Actividad (FA)

El Factor de Actividad se determina mediante dos métodos según el estado del paciente:

**Método 1: Clasificación directa (pacientes con movilidad limitada)**

| Estado de movilidad | FA |
|---------------------|-----|
| Encamado | 1.2 |
| Ambulatorio/Sedentario | 1.3 |

**Método 2: Cuestionario IPAQ (pacientes activos)**

Se calcula el total de MET-minutos/semana mediante:

```
MET_total = (8.0 × min_intensa × días_intensa) + 
            (4.0 × min_moderada × días_moderada) + 
            (3.3 × min_caminata × días_caminata)
```

Clasificación según MET_total:

| Nivel de actividad | Criterio | FA |
|-------------------|----------|-----|
| Ligera | MET_total < 600 | 1.41 |
| Moderada | 600 ≤ MET_total < 3000 | 1.56 |
| Vigorosa | MET_total ≥ 3000 | 2.01 |

---

#### A.3 Factor de Estrés (FE)

El Factor de Estrés se asigna según la condición médica y su severidad:

| Condición | Severidad | FE |
|-----------|-----------|-----|
| Estable/Mantenimiento | - | 1.0 |
| Cirugía | Leve | 1.10 |
| Cirugía | Moderada | 1.15 |
| Cirugía | Severa | 1.20 |
| Infección | Leve | 1.20 |
| Infección | Moderada | 1.40 |
| Infección | Severa | 1.60 |
| Trauma | Leve | 1.10 |
| Trauma | Moderado | 1.45 |
| Trauma | Severo | 1.80 |
| Quemaduras | Leve | 1.50 |
| Quemaduras | Moderada | 1.70 |
| Quemaduras | Severa | 1.90 |

---

### B. Cálculo de Proteínas

El requerimiento proteico se calcula según el estado nutricional del paciente:

**Caso 1: Mantenimiento (Perfiles 1 y 2)**

```
Proteína (g/día) = P × 1.2
```

**Caso 2: Depleción nutricional (Perfil 3)**

```
Proteína (g/día) = P × 1.8  [Dieta Hiperproteica]
```

**Ajustes por comorbilidades:**

```
Si enfermedad renal SIN diálisis:
    Proteína (g/día) = P × 0.7

Si enfermedad renal CON diálisis:
    Proteína (g/día) = P × 1.3

Si enfermedad hepática:
    Proteína (g/día) = P × 1.1
```

**Relación Calorías No Proteicas : Nitrógeno**

Para optimizar la utilización de proteína, se verifica:

```
Nitrógeno (g) = Proteína (g) / 6.25

Calorías_No_Proteicas = Energía - (Proteína × 4)

Relación = Calorías_No_Proteicas / Nitrógeno

Objetivo: Relación = 150:1
```

---

### C. Cálculo de Carbohidratos

**Rango estándar (50-60% del VCT):**

```
CHO_mín (g/día) = (Energía × 0.50) / 4

CHO_máx (g/día) = (Energía × 0.60) / 4
```

**Restricción pulmonar (máximo 40% del VCT):**

```
Si enfermedad pulmonar:
    CHO_máx (g/día) = (Energía × 0.40) / 4
```

Donde:
- 4 kcal/g = valor energético de los carbohidratos

---

### D. Cálculo de Grasas

**Rango estándar (25-30% del VCT):**

```
Grasa_mín (g/día) = (Energía × 0.25) / 9

Grasa_máx (g/día) = (Energía × 0.30) / 9
```

**Dieta hipograsa (20-25% del VCT):**

```
Si malabsorción o esteatorrea:
    Grasa_mín (g/día) = (Energía × 0.20) / 9
    Grasa_máx (g/día) = (Energía × 0.25) / 9
```

**Restricción de grasas saturadas (hiperlipidemia):**

```
Si hiperlipidemia:
    Grasa_saturada (g/día) < (Energía × 0.07) / 9
    Grasa_monoinsaturada (g/día) ≤ (Energía × 0.08) / 9
    Colesterol (mg/día) < 300
```

Donde:
- 9 kcal/g = valor energético de las grasas

---

### E. Requerimientos de Micronutrientes (RDA)

Los requerimientos se establecen según la edad del paciente:

| Micronutriente | RDA (51-70 años) | RDA (>70 años) | Unidad |
|----------------|------------------|----------------|--------|
| Vitamina A | 900 | 900 | mcg/día |
| Vitamina E | 15 | 15 | mg/día |
| Vitamina B6 | 1.7 | 1.7 | mg/día |
| Vitamina B12 | 2.4 | 2.4 | mcg/día |
| Zinc | 11 | 11 | mg/día |

**Criterio de alerta:**

```
Si Ingesta < (RDA × 0.80):
    → Generar alerta de déficit
```

---

## Diagrama de Flujo del Modelo

```
┌─────────────────────────────────────────────────────────────────┐
│                   ENTRADA DE DATOS                              │
├─────────────────────────────────────────────────────────────────┤
│  • Peso (kg)                    • Tratamiento oncológico        │
│  • Talla (cm)                   • Estado nutricional            │
│  • Edad (años)                  • Comorbilidades                │
│  • Temperatura (°C)             • Intolerancia alimentarias     │
│  • Estado de salud actual       • Nivel de actividad física     │
└───────────────────────┬─────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────────┐
│              CÁLCULO DE TASA METABÓLICA BASAL (TMB)             │
├─────────────────────────────────────────────────────────────────┤
│  TMB = 66.47 + (13.75×P) + (5.0×T) - (6.76×E)                  │
│  Si fiebre: TMB_ajustada = TMB × [1 + 0.10×(temp - 37)]        │
└───────────────────────┬─────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────────┐
│              DETERMINACIÓN DE FACTOR DE ACTIVIDAD (FA)          │
├─────────────────────────────────────────────────────────────────┤
│  Ruta 1: Encamado → FA = 1.2                                   │
│  Ruta 2: Sedentario → FA = 1.3                                 │
│  Ruta 3: Activo → IPAQ → MET-min/sem → FA (1.41/1.56/2.01)    │
└───────────────────────┬─────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────────┐
│              DETERMINACIÓN DE FACTOR DE ESTRÉS (FE)             │
├─────────────────────────────────────────────────────────────────┤
│  Estable → FE = 1.0                                             │
│  Cirugía/Infección/Trauma/Quemadura →                           │
│  Severidad (Leve/Moderada/Severa) → FE (1.10 a 1.90)           │
└───────────────────────┬─────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────────┐
│              CÁLCULO DE ENERGÍA TOTAL                           │
├─────────────────────────────────────────────────────────────────┤
│  Energía (kcal/día) = TMB × FA × FE                            │
└───────────────────────┬─────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────────┐
│              CLASIFICACIÓN EN RANGO ENERGÉTICO                  │
├─────────────────────────────────────────────────────────────────┤
│  Energía/Peso < 24 → Rango BAJO (20-24 kcal/kg)                │
│  24 ≤ Energía/Peso ≤ 30 → Rango MEDIO (25-30 kcal/kg)          │
│  Energía/Peso > 30 → Rango ALTO (30-35 kcal/kg)                │
└───────────────────────┬─────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────────┐
│              ASIGNACIÓN DE PERFIL METABÓLICO                    │
├─────────────────────────────────────────────────────────────────┤
│  Perfil 1: Vigilancia Activa/Post-cirugía + Sobrepeso          │
│  Perfil 2: Estable en Radioterapia/Post-cirugía + Peso estable │
│  Perfil 3: Pérdida de peso / Efectos secundarios tratamiento   │
└───────────────────────┬─────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────────┐
│         CÁLCULO DE REQUERIMIENTOS DE MACRONUTRIENTES            │
├─────────────────────────────────────────────────────────────────┤
│  PROTEÍNA:                                                       │
│    • Mantenimiento (P1, P2): Peso × 1.2 g/kg                   │
│    • Depleción (P3): Peso × 1.8 g/kg                           │
│    • Ajustes por comorbilidad: ×0.7 / ×1.1 / ×1.3             │
│                                                                  │
│  CARBOHIDRATOS:                                                  │
│    • Estándar: 50-60% VCT → (Energía × 0.50-0.60) / 4         │
│    • Restricción pulmonar: máx 40% VCT                         │
│                                                                  │
│  GRASAS:                                                         │
│    • Estándar: 25-30% VCT → (Energía × 0.25-0.30) / 9         │
│    • Hipograsa: 20-25% VCT (malabsorción)                      │
│    • Hiperlipidemia: Grasa_sat < 7%, Colest < 300mg           │
└───────────────────────┬─────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────────┐
│         ESTABLECIMIENTO DE REQUERIMIENTOS DE MICRONUTRIENTES    │
├─────────────────────────────────────────────────────────────────┤
│  RDA según edad (51-70 años / >70 años):                        │
│    • Vitamina A: 900 mcg/día                                    │
│    • Vitamina E: 15 mg/día                                      │
│    • Vitamina B6: 1.7 mg/día                                    │
│    • Vitamina B12: 2.4 mcg/día                                  │
│    • Zinc: 11 mg/día                                            │
└───────────────────────┬─────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────────────┐
│                   SALIDA DEL MODELO                             │
├─────────────────────────────────────────────────────────────────┤
│  • Energía total (kcal/día)                                     │
│  • Proteína (g/día) con rango                                   │
│  • Carbohidratos (g/día) con rango mín-máx                      │
│  • Grasas (g/día) con rango mín-máx                             │
│  • RDA de 5 micronutrientes clave                               │
│  • Perfil metabólico asignado (1, 2 o 3)                        │
│  • Plan de comidas distribuido por tiempo (Desayuno, Colación,  │
│    Almuerzo, Merienda, Cena) con cantidades exactas             │
└─────────────────────────────────────────────────────────────────┘
```

---

## Ejemplo Numérico Completo

### Datos del paciente:

- **Nombre**: Carlos, 68 años
- **Peso**: 72 kg
- **Talla**: 170 cm
- **Diagnóstico**: Cáncer de próstata post-prostatectomía radical
- **Estado nutricional**: Pérdida de peso reciente (5 kg en 2 meses)
- **Temperatura**: 37.5°C
- **Estado de salud**: Recuperándose de cirugía (severidad moderada)
- **Movilidad**: Ambulatorio/Sedentario
- **Comorbilidades**: Ninguna
- **Intolerancia**: Ninguna

### Paso 1: Cálculo de TMB

```
TMB = 66.47 + (13.75 × 72) + (5.0 × 170) - (6.76 × 68)
TMB = 66.47 + 990 + 850 - 459.68
TMB = 1,446.79 kcal/día
```

**Ajuste por fiebre:**

```
Temperatura = 37.5°C → 0.5°C sobre 37°C
TMB_ajustada = 1,446.79 × [1 + 0.10 × 0.5]
TMB_ajustada = 1,446.79 × 1.05
TMB_ajustada = 1,519.13 kcal/día
```

### Paso 2: Factor de Actividad

Carlos es ambulatorio/sedentario:

```
FA = 1.3
```

### Paso 3: Factor de Estrés

Recuperándose de cirugía moderada:

```
FE = 1.15
```

### Paso 4: Energía Total

```
Energía = TMB_ajustada × FA × FE
Energía = 1,519.13 × 1.3 × 1.15
Energía = 2,271.36 kcal/día ≈ 2,270 kcal/día
```

**Clasificación energética:**

```
Energía/Peso = 2,270 / 72 = 31.5 kcal/kg/día
→ RANGO ALTO (30-35 kcal/kg)
```

### Paso 5: Asignación de Perfil

- Post-cirugía + Pérdida de peso reciente → **Perfil 3** (Depleción nutricional)

### Paso 6: Cálculo de Macronutrientes

**Proteína (Perfil 3 - Hiperproteica):**

```
Proteína = 72 × 1.8 = 129.6 g/día ≈ 130 g/día
```

**Verificación relación 150:1:**

```
Nitrógeno = 130 / 6.25 = 20.8 g
Calorías_No_Proteicas = 2,270 - (130 × 4) = 2,270 - 520 = 1,750 kcal
Relación = 1,750 / 20.8 = 84.1:1

⚠️ Relación inferior al objetivo (150:1)
→ Aumentar carbohidratos y grasas para alcanzar 150:1
```

**Carbohidratos (50-60% VCT):**

```
CHO_mín = (2,270 × 0.50) / 4 = 283.8 g/día
CHO_máx = (2,270 × 0.60) / 4 = 340.5 g/día

Rango: 284 - 341 g/día
```

**Grasas (25-30% VCT):**

```
Grasa_mín = (2,270 × 0.25) / 9 = 63.1 g/día
Grasa_máx = (2,270 × 0.30) / 9 = 75.7 g/día

Rango: 63 - 76 g/día
```

### Paso 7: Micronutrientes (RDA para 68 años)

| Micronutriente | RDA | Umbral de alerta (80%) |
|----------------|-----|------------------------|
| Vitamina A | 900 mcg/día | < 720 mcg/día |
| Vitamina E | 15 mg/día | < 12 mg/día |
| Vitamina B6 | 1.7 mg/día | < 1.36 mg/día |
| Vitamina B12 | 2.4 mcg/día | < 1.92 mcg/día |
| Zinc | 11 mg/día | < 8.8 mg/día |

### Paso 8: Salida del Modelo

**Requerimientos nutricionales de Carlos:**

- **Energía**: 2,270 kcal/día (31.5 kcal/kg)
- **Perfil**: 3 (Depleción - Soporte intensivo)
- **Proteína**: 130 g/día (1.8 g/kg) - Dieta hiperproteica
- **Carbohidratos**: 284-341 g/día (50-60% VCT)
- **Grasas**: 63-76 g/día (25-30% VCT)
- **Micronutrientes**: RDA para 68 años

**Plan de comidas sugerido (Perfil 3, Rango Alto):**

| Tiempo | Ejemplo de alimentos | Calorías aprox. |
|--------|---------------------|----------------|
| Desayuno | 1 rebanada pan + ½ taza avena + 2 huevos revueltos + 1 taza yogur griego + 2 cdas mantequilla maní | 650 kcal |
| Media mañana | ¼ taza pasas + ¼ taza nueces | 280 kcal |
| Almuerzo | 1 taza arroz + 170g pescado al horno + ½ taza verduras + 1 cda aceite oliva | 700 kcal |
| Media tarde | 1 taza yogur + ½ taza durazno | 200 kcal |
| Cena | 1 taza fideos + 170g pollo desmenuzado + ½ taza verduras cocidas | 640 kcal |

**Total estimado**: ~2,470 kcal/día (cumple requerimiento)

---

## Validación del Modelo

El sistema compara continuamente la ingesta registrada vs. los requerimientos calculados:

```
Para cada nutriente n:
    Cumplimiento_n = (Ingesta_n / Requerimiento_n) × 100

    Si Cumplimiento_n < 80%:
        → Alerta de déficit
        → Sugerencias de alimentos fuente
        → Cálculo de cantidad necesaria para corregir
```

**Ejemplo de alerta generada:**

```
⚠️ Déficit Proteico Detectado

Meta diaria: 130g
Ingesta registrada: 42g
Déficit: 88g (68% por debajo del objetivo)

Cumplimiento: 32%

Sugerencia: Agregar 100g de pechuga de pollo (31g proteína) 
+ 2 huevos cocidos (25g proteína) en tu próxima comida.
```

---

## Referencias Bibliográficas del Modelo

1. Harris JA, Benedict FG. A Biometric Study of Human Basal Metabolism. Proc Natl Acad Sci. 1918;4(12):370-373.

2. Craig CL, et al. International Physical Activity Questionnaire: 12-Country Reliability and Validity. Med Sci Sports Exerc. 2003;35(8):1381-1395.

3. McClave SA, et al. Guidelines for the Provision and Assessment of Nutrition Support Therapy in the Adult Critically Ill Patient: ASPEN/SCCM. JPEN J Parenter Enteral Nutr. 2016;40(2):159-211.

4. Arends J, et al. ESPEN Guidelines on Nutrition in Cancer Patients. Clin Nutr. 2017;36(1):11-48.

5. Institute of Medicine. Dietary Reference Intakes for Energy, Carbohydrate, Fiber, Fat, Fatty Acids, Cholesterol, Protein, and Amino Acids. Washington, DC: The National Academies Press; 2005.

6. Ministerio de Salud del Perú - EsSalud. Protocolo de Evaluación y Soporte Nutricional en Pacientes Hospitalizados. Lima: MINSA; 2020.
