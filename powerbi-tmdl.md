---
name: Power BI TMDL Manager
# prettier-ignore
description: This skill should be used when the user asks to "modify TMDL file", "edit DAX measure", "update Power BI model", "add column to table", "create new measure", "change table definition", "analyze TMDL", "review DAX code", "modify .pbip project", "document Power BI model", "scan semantic model", "generate model documentation", "create multiple measures", "bulk create tables", or provides instructions to work with Power BI semantic model files. Manages TMDL (Tabular Model Definition Language) files for Power BI projects with full read, write, scan, document, and bulk operation capabilities. Works with any Power BI project automatically.
version: 2.0.0
allowed-tools: Read, Edit, Write, Grep, Glob, AskUserQuestion
---

# Power BI TMDL Manager

Skill universal para trabajar con archivos TMDL (Tabular Model Definition Language) de **cualquier proyecto Power BI (.pbip)**. Detecta automáticamente la estructura del proyecto, escanea el modelo completo, genera documentación de referencia en JSON y Markdown, y ejecuta operaciones masivas sin intervención manual.

## 🎯 Qué hace este skill

### Capacidades básicas:
1. **Auto-detección de proyectos** - Encuentra automáticamente archivos .pbip en el directorio
2. **Leer y analizar** archivos TMDL del modelo semántico
3. **Modificar medidas DAX** existentes o crear nuevas
4. **Editar definiciones de tablas** (columnas, propiedades, particiones)
5. **Modificar relaciones** entre tablas
6. **Actualizar configuraciones del modelo** (culture, anotaciones, etc.)
7. **Crear nuevas tablas de medidas**
8. Validar sintaxis TMDL y DAX

### Capacidades avanzadas:
9. **Escanear el modelo completo** de cualquier proyecto Power BI automáticamente
10. **Generar documentación de referencia** en JSON y Markdown
11. **Documentar todas las tablas, medidas y relaciones** del modelo
12. **Analizar y sugerir mejoras** de presentación de datos
13. **Preguntas aclaratorias** sobre contexto de negocio
14. **Configurar límites de tokens** para modelos grandes (<50k)
15. **Excluir tablas específicas** de la documentación
16. **Creación masiva de medidas** basada en patrones
17. **Creación masiva de tablas** con estructura definida
18. **Aplicar cambios en lote** sin intervención manual
19. **Usar documentación como contexto** para tareas complejas

## 📁 Estructura de un proyecto Power BI (.pbip)

```
cualquier-proyecto.pbip                 # Archivo principal del proyecto
├── cualquier-proyecto.Report/          # Carpeta del reporte visual
│   └── definition.pbir                 # Definición del reporte
└── cualquier-proyecto.SemanticModel/   # Modelo semántico (DATOS Y MEDIDAS)
    └── definition/
        ├── model.tmdl                  # Configuración del modelo
        ├── database.tmdl               # Configuración de la base de datos
        ├── relationships.tmdl          # Relaciones entre tablas
        ├── cultures/
        │   └── en-US.tmdl              # Configuración regional
        └── tables/                     # Definiciones de tablas
            ├── DimCustomers.tmdl
            ├── FactSales.tmdl
            ├── Measures.tmdl           # Tablas de medidas DAX
            └── DateTableTemplate_*.tmdl # Tablas autogeneradas
```

## 🚀 Auto-detección de proyectos

Cuando el usuario pide trabajar con Power BI, el skill:

### Paso 1: Buscar proyecto automáticamente
```
1. Ejecutar: Glob pattern="**/*.pbip" desde directorio actual
2. Si encuentra 1 proyecto: Usarlo automáticamente
3. Si encuentra múltiples: Preguntar al usuario cuál usar
4. Si no encuentra: Pedir la ruta manualmente
```

### Paso 2: Identificar estructura
```
Una vez localizado proyecto.pbip:
1. Identificar carpeta SemanticModel: [nombre].SemanticModel/
2. Ubicar definition/: [nombre].SemanticModel/definition/
3. Identificar rutas clave:
   - Tablas: [nombre].SemanticModel/definition/tables/
   - Model: [nombre].SemanticModel/definition/model.tmdl
   - Relations: [nombre].SemanticModel/definition/relationships.tmdl
```

### Paso 3: Informar al usuario
```
✅ Proyecto detectado: [nombre-proyecto].pbip
📂 Modelo semántico: [nombre-proyecto].SemanticModel
📊 Ruta de tablas: definition/tables/
```

## 📊 ESCANEO Y DOCUMENTACIÓN AUTOMÁTICA

### Workflow completo de documentación

**Comando**: "Documenta el modelo Power BI" o "Escanea el modelo completo"

#### Paso 1: Auto-detección
```
1. Buscar .pbip en directorio actual
2. Identificar estructura del proyecto
3. Confirmar con usuario si es correcto
```

#### Paso 2: Preguntas de contexto (AskUserQuestion)
```
Pregunta 1: "¿Cuál es el dominio de este modelo?"
Opciones:
- Ventas y comercio
- Finanzas y contabilidad
- Recursos humanos
- Operaciones y logística
- Marketing y CRM
- Otro (especificar)

Pregunta 2: "¿Qué nivel de documentación necesitas?"
Opciones:
- Completa (todas las tablas y medidas)
- Resumida (solo tablas principales y medidas clave)
- Técnica (para desarrolladores)
- Ejecutiva (para stakeholders)

Pregunta 3: "¿Excluir tablas autogeneradas?"
Opciones:
- Sí, excluir LocalDateTable y DateTableTemplate
- Sí, excluir solo LocalDateTable
- No, documentar todo
```

#### Paso 3: Configuración automática
Crear `.powerbi-docs-config.json`:
```json
{
  "projectName": "[auto-detectado]",
  "modelPath": "[auto-detectado]",
  "tokenLimit": 50000,
  "excludeTables": [
    "LocalDateTable_*",
    "DateTableTemplate_*"
  ],
  "excludeColumns": [
    "lineageTag",
    "annotation PBI_*"
  ],
  "includeMeasures": true,
  "includeRelationships": true,
  "includePartitions": false,
  "businessContext": {
    "domain": "[de respuesta usuario]",
    "documentationLevel": "[de respuesta usuario]"
  },
  "generatedAt": "[timestamp]"
}
```

#### Paso 4: Escaneo del modelo
```
1. Leer model.tmdl → extraer lista de tablas referenciadas
2. Leer relationships.tmdl → extraer todas las relaciones
3. Para cada tabla en tables/:
   a. Leer archivo .tmdl
   b. Clasificar: ¿Es tabla de datos o de medidas?
   c. Extraer metadata:
      - Columnas y propiedades
      - Medidas DAX (si tiene)
      - Partition (fuente de datos)
   d. Contar tokens acumulados
   e. Si alcanza límite: priorizar tablas importantes
```

#### Paso 5: Análisis inteligente
```
Para cada medida DAX encontrada:
- Detectar complejidad: simple/media/compleja
- Identificar dependencias (tablas y columnas usadas)
- Detectar patrones anti-performance (FILTER innecesario)
- Verificar organización (displayFolder)

Para el modelo completo:
- Identificar tablas de hechos vs dimensiones
- Mapear relaciones y direcciones de filtro
- Detectar relaciones inactivas o redundantes
- Sugerir mejoras de estructura
```

#### Paso 6: Generar documentación JSON

Crear `model-documentation.json`:
```json
{
  "metadata": {
    "projectName": "Sales Dashboard",
    "modelPath": "C:/Projects/YourProject/sales-dashboard.SemanticModel",
    "generatedAt": "2026-01-22T21:00:00Z",
    "totalTables": 12,
    "totalMeasures": 45,
    "totalRelationships": 8,
    "excludedTables": ["LocalDateTable_*"],
    "businessDomain": "Ventas y comercio"
  },
  "tables": [
    {
      "name": "FactSales",
      "type": "fact",
      "fileName": "FactSales.tmdl",
      "columns": [
        {
          "name": "SaleID",
          "dataType": "int64",
          "summarizeBy": "none",
          "sourceColumn": "SaleID"
        },
        {
          "name": "Amount",
          "dataType": "double",
          "formatString": "$#,##0.00",
          "summarizeBy": "sum"
        }
      ],
      "measures": [],
      "partition": {
        "mode": "import",
        "sourceType": "SQL",
        "summary": "SQL Server - SalesDB"
      },
      "relationships": {
        "outgoing": [
          {"to": "DimCustomers", "column": "CustomerID"},
          {"to": "DimProducts", "column": "ProductID"}
        ],
        "incoming": []
      }
    },
    {
      "name": "Measures",
      "type": "measures",
      "fileName": "Measures.tmdl",
      "measures": [
        {
          "name": "Total Sales",
          "dax": "SUM(FactSales[Amount])",
          "formatString": "$#,##0.00",
          "displayFolder": "Sales",
          "complexity": "simple",
          "dependencies": ["FactSales[Amount]"]
        },
        {
          "name": "YoY Growth %",
          "dax": "VAR CurrentYear = [Total Sales]\\nVAR PreviousYear = CALCULATE([Total Sales], SAMEPERIODLASTYEAR('Date'[Date]))\\nRETURN DIVIDE(CurrentYear - PreviousYear, PreviousYear, 0)",
          "formatString": "0.0%",
          "displayFolder": "Sales/Time Intelligence",
          "complexity": "medium",
          "dependencies": ["Total Sales", "Date[Date]"],
          "usesTimeIntelligence": true
        }
      ]
    }
  ],
  "relationships": [
    {
      "id": "abc123",
      "from": "FactSales.CustomerID",
      "to": "DimCustomers.CustomerID",
      "cardinality": "many-to-one",
      "crossFiltering": "single",
      "isActive": true
    }
  ],
  "analysis": {
    "factTables": ["FactSales", "FactOrders"],
    "dimensionTables": ["DimCustomers", "DimProducts", "DimDate"],
    "measureTables": ["Measures", "KPIs"],
    "starSchema": true
  },
  "suggestions": {
    "performance": [
      "Medida 'Complex Calculation' usa FILTER, considerar usar CALCULATE",
      "Tabla 'DimCustomers' tiene columnas sin usar en medidas"
    ],
    "organization": [
      "15 medidas sin displayFolder definido",
      "Considerar agrupar medidas temporales en carpeta 'Time Intelligence'"
    ],
    "presentation": [
      "Agregar descripciones a medidas complejas",
      "Estandarizar formatos de moneda ($#,##0.00)"
    ]
  }
}
```

#### Paso 7: Generar documentación Markdown

Crear `MODEL-REFERENCE.md`:
```markdown
# Documentación del Modelo: [Nombre Proyecto]

**Generado**: 2026-01-22 21:00:00
**Dominio**: Ventas y comercio
**Tablas**: 12 (8 datos, 4 medidas)
**Medidas DAX**: 45
**Relaciones**: 8

---

## 📋 Resumen Ejecutivo

Este modelo de Power BI contiene datos de [dominio] con un esquema [estrella/copo de nieve]. Las entidades principales son [detectadas automáticamente], con [X] tablas de hechos y [Y] dimensiones.

---

## 📊 Tablas de Datos

### FactSales (Tabla de Hechos)
**Archivo**: `tables/FactSales.tmdl`
**Fuente**: SQL Server - SalesDB
**Tipo**: Tabla de hechos (transaccional)

**Columnas principales**:
| Columna | Tipo | Formato | Uso |
|---------|------|---------|-----|
| SaleID | int64 | - | Clave primaria |
| Amount | double | $#,##0.00 | Monto de venta |
| CustomerID | int64 | - | FK a DimCustomers |
| Date | dateTime | General Date | Fecha de venta |

**Relaciones**:
- → `DimCustomers` (Many-to-One) via CustomerID
- → `DimProducts` (Many-to-One) via ProductID
- → `DimDate` (Many-to-One) via Date

---

### DimCustomers (Dimensión)
**Archivo**: `tables/DimCustomers.tmdl`
**Fuente**: SQL Server - SalesDB
**Tipo**: Dimensión

**Columnas principales**:
| Columna | Tipo | Descripción |
|---------|------|-------------|
| CustomerID | int64 | Identificador único |
| CustomerName | string | Nombre del cliente |
| Segment | string | Segmento (Corporate/Consumer/Home Office) |
| Country | string | País |

---

## 📐 Tablas de Medidas

### Measures
**Archivo**: `tables/Measures.tmdl`
**Total de medidas**: 18

#### Total Sales
```dax
SUM(FactSales[Amount])
```
- **Formato**: $#,##0.00
- **Carpeta**: Sales
- **Complejidad**: Simple
- **Dependencias**: FactSales[Amount]

---

#### YoY Growth %
```dax
VAR CurrentYear = [Total Sales]
VAR PreviousYear =
    CALCULATE(
        [Total Sales],
        SAMEPERIODLASTYEAR('Date'[Date])
    )
RETURN
    DIVIDE(CurrentYear - PreviousYear, PreviousYear, 0)
```
- **Formato**: 0.0%
- **Carpeta**: Sales/Time Intelligence
- **Complejidad**: Media
- **Dependencias**: [Total Sales], Date[Date]
- **Usa**: Time Intelligence

---

## 🔗 Mapa de Relaciones

```
FactSales
├── → DimCustomers (CustomerID)
├── → DimProducts (ProductID)
└── → DimDate (Date)

FactOrders
├── → DimCustomers (CustomerID)
└── → DimDate (OrderDate)
```

**Esquema detectado**: Estrella ⭐

---

## 💡 Sugerencias de Mejora

### ⚡ Rendimiento
- ⚠️ Medida 'Complex Calculation' usa FILTER, considerar CALCULATE para mejor rendimiento
- 💡 Tabla 'DimCustomers' tiene 5 columnas sin usar en medidas

### 📁 Organización
- 📌 15 medidas sin displayFolder definido
- 📌 Considerar agrupar medidas temporales en carpeta "Time Intelligence"
- 📌 Estandarizar nomenclatura (usar PascalCase o camelCase consistentemente)

### 🎨 Presentación
- ✨ Agregar descripciones a 8 medidas complejas
- ✨ Estandarizar formatos de moneda: usar $#,##0.00
- ✨ Agregar displayFolder a todas las medidas

---

## 📖 Glosario de Términos

| Término | Definición |
|---------|------------|
| Total Sales | Suma total de ventas en el período seleccionado |
| YoY Growth % | Crecimiento año a año en porcentaje |
| [Auto-generado basado en medidas encontradas] |
```

Crear `QUICK-REFERENCE.md`:
```markdown
# Referencia Rápida: [Nombre Proyecto]

## Métricas Principales
- **Total Sales**: $X.XX millones
- **Customer Count**: X,XXX clientes
- **Average Order Value**: $X.XX

## Tablas Clave
- FactSales (Transacciones)
- DimCustomers (Clientes)
- DimProducts (Productos)

## Medidas Más Usadas
1. Total Sales
2. YoY Growth %
3. Customer Count
4. Average Order Value

[Generado automáticamente basado en análisis del modelo]
```

## 🔧 OPERACIONES MASIVAS

### 1. Creación Masiva de Medidas Temporales

**Comando**: "Crea versiones MoM y YoY para todas las medidas base"

**Proceso automático**:
```
1. Detectar proyecto .pbip
2. Escanear todas las tablas de medidas
3. Identificar medidas "base" (sin sufijos MoM, YoY, %, etc.)
4. Para cada medida base:
   - Generar [Nombre] MoM
   - Generar [Nombre] MoM %
   - Generar [Nombre] YoY
   - Generar [Nombre] YoY %
5. Aplicar todo en un solo Edit por tabla
6. Actualizar documentación si existe
```

**Patrón generado** (ejemplo genérico):
```dax
// Medida original
measure 'Total Sales' =
    SUM(Sales[Amount])
    formatString: $#,##0.00
    displayFolder: "Base Metrics"

// Auto-generadas:
measure 'Total Sales MoM' =
    VAR CurrentMonth = [Total Sales]
    VAR PreviousMonth =
        CALCULATE(
            [Total Sales],
            DATEADD('Calendar'[Date], -1, MONTH)
        )
    RETURN
        CurrentMonth - PreviousMonth
    formatString: $#,##0.00
    displayFolder: "Time Intelligence/MoM"

measure 'Total Sales MoM %' =
    VAR CurrentMonth = [Total Sales]
    VAR PreviousMonth =
        CALCULATE(
            [Total Sales],
            DATEADD('Calendar'[Date], -1, MONTH)
        )
    RETURN
        DIVIDE(CurrentMonth - PreviousMonth, PreviousMonth, 0)
    formatString: 0.0%
    displayFolder: "Time Intelligence/MoM"

measure 'Total Sales YoY' =
    VAR CurrentYear = [Total Sales]
    VAR PreviousYear =
        CALCULATE(
            [Total Sales],
            SAMEPERIODLASTYEAR('Calendar'[Date])
        )
    RETURN
        CurrentYear - PreviousYear
    formatString: $#,##0.00
    displayFolder: "Time Intelligence/YoY"

measure 'Total Sales YoY %' =
    VAR CurrentYear = [Total Sales]
    VAR PreviousYear =
        CALCULATE(
            [Total Sales],
            SAMEPERIODLASTYEAR('Calendar'[Date])
        )
    RETURN
        DIVIDE(CurrentYear - PreviousYear, PreviousYear, 0)
    formatString: 0.0%
    displayFolder: "Time Intelligence/YoY"
```

### 2. Reorganización Automática de Medidas

**Comando**: "Reorganiza medidas por categorías"

**Proceso**:
```
1. Leer todas las medidas del modelo
2. Clasificar automáticamente por:
   - KPIs (medidas principales simples)
   - Time Intelligence (MoM, YoY, etc.)
   - Calculations (ratios, promedios)
   - Aggregations (sumas, conteos)
3. Preguntar confirmación al usuario
4. Crear nuevas tablas de medidas categorizadas
5. Mover medidas a tabla apropiada
6. Actualizar model.tmdl
```

### 3. Optimización Masiva de DAX

**Comando**: "Optimiza todas las medidas DAX"

**Detecciones automáticas**:
```
Anti-patrón 1: FILTER innecesario
❌ COUNTROWS(FILTER(Table, Table[Column] = Value))
✅ CALCULATE(COUNTROWS(Table), Table[Column] = Value)

Anti-patrón 2: División sin protección
❌ [Numerator] / [Denominator]
✅ DIVIDE([Numerator], [Denominator], 0)

Anti-patrón 3: Múltiples CALCULATE anidados
❌ CALCULATE(CALCULATE(SUM(...), Filter1), Filter2)
✅ CALCULATE(SUM(...), Filter1, Filter2)

Anti-patrón 4: VALUES vs DISTINCT innecesario
[Detectar contextos donde uno es más eficiente]
```

### 4. Uso de Documentación como Contexto

**Comando**: "Usando la documentación, crea un dashboard de KPIs"

**Proceso inteligente**:
```
1. Leer model-documentation.json
2. Identificar tablas de hechos (Facts)
3. Identificar medidas numéricas principales
4. Identificar campos de fecha
5. Generar automáticamente:
   - Medidas totales
   - Medidas de crecimiento temporal
   - Medidas de promedios
   - Medidas de conteos únicos
6. Aplicar todo sin pedir confirmación item por item
```

**Ejemplo de generación inteligente**:
```
Detectado: FactSales con columnas [Amount, Quantity, CustomerID, Date]

Auto-generar:
measure 'Total Sales Amount' = SUM(FactSales[Amount])
measure 'Total Quantity' = SUM(FactSales[Quantity])
measure 'Unique Customers' = DISTINCTCOUNT(FactSales[CustomerID])
measure 'Average Order Value' = DIVIDE([Total Sales Amount], DISTINCTCOUNT(FactSales[OrderID]))
measure 'Sales MoM %' = [lógica MoM basada en Date]
measure 'Sales YoY %' = [lógica YoY basada en Date]
```

## 🛠️ MODIFICACIONES BÁSICAS

### Modificar medida existente

```
1. Auto-detectar proyecto
2. Buscar medida: Grep pattern="measure 'Nombre'"
3. Leer archivo que la contiene
4. Edit: cambiar DAX
5. Verificar sintaxis
6. Informar cambios
```

### Crear nueva medida

```
1. Identificar o preguntar: ¿En qué tabla?
2. Leer tabla de medidas
3. Insertar nueva medida antes de partition
4. Mantener indentación (TABS)
5. Verificar sintaxis
```

### Crear nueva tabla de medidas

```
1. Write: nuevo archivo .tmdl en tables/
2. Estructura estándar:
   table NombreTabla
   measure 'Medida1' = ...
   partition NombreTabla = m
       mode: import
       source = let Source = #table({}, {}) in Source
3. Edit model.tmdl: agregar ref table NombreTabla
```

## ⚙️ SINTAXIS TMDL

### Estructura de medida
```
measure 'Nombre de Medida' =
    VAR Variable1 = CALCULATE(...)
    VAR Variable2 = FILTER(...)
    RETURN
        DIVIDE(Variable1, Variable2, 0)
    formatString: $#,##0.00
    displayFolder: "Carpeta/Subcarpeta"
    description: "Descripción opcional"
```

### Tipos de datos
- `string` - Texto
- `int64` - Entero
- `double` - Decimal
- `dateTime` - Fecha/hora
- `boolean` - Verdadero/falso

### Formatos comunes
- `#,##0` - Entero con comas
- `$#,##0.00` - Moneda
- `0.0%` - Porcentaje 1 decimal
- `#,##0.00` - Decimal 2 lugares

## ⚠️ PRECAUCIONES CRÍTICAS

### 1. TABS no espacios
TMDL usa **TABS** para indentación. Siempre copiar exactamente la indentación original.

### 2. Cerrar Power BI antes de modificar
Los cambios solo funcionan si Power BI Desktop está cerrado. Siempre verificar primero.

### 3. Partition al final
En toda tabla TMDL, el bloque `partition` SIEMPRE va al final del archivo.

### 4. LineageTags
No modificar lineageTags existentes. Para nuevos elementos, generar GUIDs únicos.

### 5. Sintaxis DAX válida
Validar antes de escribir que el DAX sea sintácticamente correcto.

## ✅ VALIDACIÓN FINAL

Antes de completar cualquier operación:
- ✅ Verificar sintaxis TMDL correcta
- ✅ Validar indentación (tabs, no espacios)
- ✅ Confirmar que DAX es sintácticamente válido
- ✅ Verificar que partition esté al final
- ✅ Comprobar que tablas/columnas referenciadas existan
- ✅ Releer archivo modificado para confirmar cambios
- ✅ Actualizar documentación si existe
- ✅ Informar al usuario sobre próximos pasos (cerrar/reabrir Power BI)

## 📚 REFERENCIA RÁPIDA DAX

### Funciones comunes
- `SUM()`, `AVERAGE()`, `MIN()`, `MAX()` - Agregaciones
- `COUNT()`, `COUNTROWS()`, `DISTINCTCOUNT()` - Conteos
- `CALCULATE()` - Modificar contexto de filtro
- `FILTER()` - Filtrar tabla
- `DIVIDE()` - División segura (evita /0)
- `VALUES()`, `DISTINCT()` - Valores únicos
- `RELATED()`, `RELATEDTABLE()` - Navegación de relaciones
- `SAMEPERIODLASTYEAR()`, `DATEADD()` - Time intelligence

### Carpetas estándar (displayFolder)
- "Base Metrics" - Métricas fundamentales
- "KPIs" - Indicadores clave
- "Time Intelligence" - Medidas temporales
- "Time Intelligence/MoM" - Month over Month
- "Time Intelligence/YoY" - Year over Year
- "Calculations" - Cálculos derivados
- "Aggregations" - Sumas y conteos
