# ProstaCare - Conjunto de Datos Nutricional

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Dataset Version](https://img.shields.io/badge/version-1.0-blue.svg)](https://github.com/tu-usuario/prostacare-dataset)

## Descripción

Este conjunto de datos contiene información nutricional completa de **235 alimentos** de la gastronomía peruana, diseñado específicamente para el cálculo automatizado de requerimientos nutricionales en pacientes con cáncer de próstata mediante la aplicación móvil **ProstaCare**.

El dataset incluye macronutrientes (proteína, carbohidratos, grasas) y micronutrientes críticos para la salud prostática (Vitaminas A, E, B6, B12 y Zinc) según las recomendaciones de ASPEN/ESPEN y el Institute of Medicine (IOM).

## Estructura del Dataset

### Archivo Principal
- **Archivo**: `food_catalog_nutritional_dataset.csv`
- **Registros**: 235 alimentos
- **Columnas**: 16 variables

### Variables

| Columna | Tipo | Descripción | Unidad |
|---------|------|-------------|--------|
| `N` | Integer | Identificador único del alimento | - |
| `Nombre_del_alimento` | String | Denominación común en español | - |
| `Tipo` | Categorical | Clasificación: "Simple" o "Compuesto" | - |
| `Componente_principal` | String | Alimento base en preparaciones compuestas | - |
| `Componente_acompañamiento` | String | Alimento secundario en preparaciones compuestas | - |
| `Gramos_unidad_principal` | Float | Peso estándar de una unidad del componente principal | gramos |
| `Gramos_unidad_acomp` | Float | Peso estándar de una unidad del acompañamiento | gramos |
| `Porcion_sugerida_g` | Float | Cantidad recomendada para consumo | gramos |
| `Proteina_g_100g` | Float | Contenido proteico por 100g de alimento | gramos |
| `Carbohidratos_g_100g` | Float | Contenido de carbohidratos por 100g | gramos |
| `Grasas_g_100g` | Float | Contenido de grasas totales por 100g | gramos |
| `Vit_A_mcg_100g` | Float | Vitamina A (Retinol) por 100g | microgramos |
| `Vit_E_mg_100g` | Float | Vitamina E (Alfa-tocoferol) por 100g | miligramos |
| `Vit_B6_mg_100g` | Float | Vitamina B6 (Piridoxina) por 100g | miligramos |
| `Vit_B12_mcg_100g` | Float | Vitamina B12 (Cobalamina) por 100g | microgramos |
| `Zinc_mg_100g` | Float | Zinc por 100g | miligramos |

### Clasificación de Alimentos

#### Alimentos Simples
Alimentos individuales sin preparación compuesta. El cálculo nutricional se realiza directamente multiplicando los nutrientes por 100g × (porción_consumida / 100).

**Ejemplos**: Huevo cocido, Pechuga de pollo, Arroz integral, Leche, Frutas, Verduras

#### Alimentos Compuestos
Preparaciones que combinan dos componentes principales de la gastronomía peruana. El cálculo nutricional considera las porciones de cada componente por separado.

**Ejemplos**: 
- Pan con tamal (60g pan + 150g tamal)
- Pan con chicharrón (60g pan + 80g chicharrón)
- Pan con huevo (60g pan + 60g huevo frito)
- Arroz con pollo (200g arroz + 120g pollo)

## Fuentes de Datos

El conjunto de datos fue construido mediante la integración de tres fuentes principales:

1. **Tablas de Composición Nutricional de Alimentos Peruanos**
   - Centro Nacional de Alimentación y Nutrición (CENAN)
   - Instituto Nacional de Salud del Perú (INS)

2. **USDA FoodData Central**
   - Base de datos del Departamento de Agricultura de EE.UU.
   - Utilizada para alimentos no incluidos en tablas locales

3. **Registro Fotográfico y Pesaje Directo**
   - Mediciones en consulta nutricional ambulatoria
   - Porciones estándar de preparaciones peruanas compuestas

## Validación

- ✅ Valores nutricionales verificados con fuentes oficiales
- ✅ Porciones estándar validadas por nutricionistas clínicos
- ✅ Preparaciones compuestas con pesaje directo de componentes
- ✅ Micronutrientes críticos contrastados con tablas IOM/USDA

## Uso en ProstaCare

Este dataset es utilizado por el módulo de cálculo nutricional de ProstaCare para:

1. **Reconocimiento de alimentos**: Matching entre texto ingresado por el usuario y catálogo
2. **Cálculo automático de nutrientes**: Conversión de porciones a valores nutricionales totales
3. **Detección de déficits**: Comparación de ingesta vs. RDA (Recommended Dietary Allowance)
4. **Generación de alertas**: Notificaciones cuando déficit < 80% del requerimiento
5. **Recomendaciones personalizadas**: Sugerencias de alimentos para corregir déficits específicos

### Ejemplo de Cálculo

**Usuario registra**: "2 huevos cocidos"

```dart
// Búsqueda en dataset
FoodItem huevo = findFood("Huevo cocido");

// Cálculo nutricional
double gramos_consumidos = 2 × 50g = 100g;
double proteina = (huevo.proteina_g_100g × gramos_consumidos) / 100;
                = (12.6 × 100) / 100 = 12.6g

// Comparación con meta del paciente
if (proteina_total_dia < meta_proteina × 0.80) {
  showAlert("Déficit proteico detectado");
}
```

## Estadísticas del Dataset

- **Alimentos simples**: 210 (89.4%)
- **Alimentos compuestos**: 25 (10.6%)
- **Alimentos de origen animal**: 45 (19.1%)
- **Alimentos de origen vegetal**: 165 (70.2%)
- **Preparaciones peruanas típicas**: 25 (10.6%)

### Distribución de Macronutrientes (Promedio)

| Nutriente | Promedio | Mínimo | Máximo |
|-----------|----------|--------|--------|
| Proteína (g/100g) | 8.9 | 0.3 | 31.0 |
| Carbohidratos (g/100g) | 18.4 | 0 | 81.5 |
| Grasas (g/100g) | 7.2 | 0 | 100.0 |

### Cobertura de Micronutrientes

| Micronutriente | Alimentos con aporte significativo (>10% RDA por 100g) |
|----------------|--------------------------------------------------------|
| Vitamina A | 42 alimentos (17.9%) |
| Vitamina E | 38 alimentos (16.2%) |
| Vitamina B6 | 51 alimentos (21.7%) |
| Vitamina B12 | 28 alimentos (11.9%) |
| Zinc | 64 alimentos (27.2%) |

## Formato de Archivo

- **Formato**: CSV (Comma-Separated Values)
- **Encoding**: UTF-8
- **Separador**: Coma (`,`)
- **Delimitador de texto**: Ninguno
- **Valores nulos**: Representados como `-` (guión)

## Cómo Usar el Dataset

### Python (Pandas)

```python
import pandas as pd

# Cargar dataset
df = pd.read_csv('food_catalog_nutritional_dataset.csv')

# Buscar alimento
huevo = df[df['Nombre_del_alimento'] == 'Huevo cocido']

# Calcular nutrientes para 150g
porcion = 150
proteina = (huevo['Proteina_g_100g'].values[0] * porcion) / 100
print(f"Proteína en 150g de huevo: {proteina}g")
```

### R

```r
# Cargar dataset
df <- read.csv('food_catalog_nutritional_dataset.csv')

# Filtrar alimentos compuestos
compuestos <- subset(df, Tipo == 'Compuesto')

# Estadísticas descriptivas
summary(df$Proteina_g_100g)
```

### JavaScript/TypeScript (Node.js)

```javascript
const fs = require('fs');
const csv = require('csv-parser');

const foods = [];

fs.createReadStream('food_catalog_nutritional_dataset.csv')
  .pipe(csv())
  .on('data', (row) => {
    foods.push(row);
  })
  .on('end', () => {
    const pollo = foods.find(f => f.Nombre_del_alimento === 'Pechuga de pollo');
    console.log(pollo);
  });
```

## Limitaciones

- ⚠️ Los valores nutricionales pueden variar según la variedad del alimento, método de cocción y origen geográfico
- ⚠️ Preparaciones compuestas asumen proporciones estándar que pueden variar según el cocinero
- ⚠️ Micronutrientes sensibles al calor (Vit C, algunas vitaminas B) pueden degradarse con cocción prolongada
- ⚠️ Dataset enfocado en gastronomía peruana, puede requerir adaptación para otras regiones

## Actualizaciones Futuras

- [ ] Expansión a 500 alimentos
- [ ] Inclusión de fibra dietética
- [ ] Adición de ácidos grasos omega-3 y omega-6
- [ ] Incorporación de índice glucémico
- [ ] Valores de sodio y potasio para pacientes con hipertensión
- [ ] Preparaciones de otras cocinas latinoamericanas

## Licencia

Este dataset está disponible bajo la licencia **MIT License**. Puedes usar, modificar y distribuir libremente siempre que incluyas la atribución correspondiente.

## Citación

Si utilizas este dataset en tu investigación o aplicación, por favor cita:

```bibtex
@dataset{prostacare_nutritional_dataset_2024,
  author = {Tu Nombre},
  title = {ProstaCare Nutritional Food Catalog Dataset},
  year = {2024},
  publisher = {GitHub},
  url = {https://github.com/tu-usuario/prostacare-dataset}
}
```

## Contacto

Para preguntas, sugerencias o reporte de errores:
- **Email**: tu-email@ejemplo.com
- **GitHub Issues**: [https://github.com/tu-usuario/prostacare-dataset/issues](https://github.com/tu-usuario/prostacare-dataset/issues)

## Agradecimientos

- Centro Nacional de Alimentación y Nutrición (CENAN) - INS Perú
- USDA FoodData Central
- Equipo de nutricionistas clínicos del proyecto ProstaCare
- Pacientes que participaron en la validación de porciones estándar

---

**Última actualización**: Enero 2025  
**Versión del dataset**: 1.0
