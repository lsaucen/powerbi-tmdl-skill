---
name: Power BI TMDL Manager
# prettier-ignore
description: This skill should be used when the user asks to "modify TMDL file", "edit DAX measure", "update Power BI model", "add column to table", "create new measure", "change table definition", "analyze TMDL", "review DAX code", "modify .pbip project", "document Power BI model", "scan semantic model", "generate model documentation", "create multiple measures", "bulk create tables", "create calculated column", "add calculated columns", "create relationship", "modify relationship", "add relationship between tables", or provides instructions to work with Power BI semantic model files. Manages TMDL (Tabular Model Definition Language) files for Power BI projects with full read, write, scan, document, and bulk operation capabilities. Works with any Power BI project automatically.
version: 2.1.0
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
5. **Crear y modificar columnas calculadas** con DAX
6. **Crear, modificar y eliminar relaciones** entre tablas
7. **Actualizar configuraciones del modelo** (culture, anotaciones, etc.)
8. **Crear nuevas tablas de medidas**
9. Validar sintaxis TMDL y DAX

### Capacidades avanzadas:
9. **Escanear el modelo completo** de cualquier proyecto Power BI automáticamente
10. **Generar documentación de referencia** en JSON y Markdown
11. **Documentar todas las tablas, medidas y relaciones** del modelo
12. **Analizar y sugerir mejoras** de presentación de datos
13. **Preguntas aclaratorias** sobre contexto de negocio
14. **Configurar límites de tokens** para modelos grandes (<50k)
15. **Excluir tablas específicas** de la documentación
16. **Creación masiva de medidas** basada en patrones
17. **Creación masiva de columnas calculadas** en múltiples tablas
18. **Creación masiva de relaciones** basadas en convenciones
19. **Creación masiva de tablas** con estructura definida
20. **Aplicar cambios en lote** sin intervención manual
21. **Usar documentación como contexto** para tareas complejas

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

## 🔢 COLUMNAS CALCULADAS

### ¿Qué es una columna calculada?

Una columna calculada es una columna que se agrega a una tabla existente usando una fórmula DAX. A diferencia de las medidas, las columnas calculadas:
- Se calculan **fila por fila** durante la actualización de datos
- Se almacenan en el modelo (consumen memoria)
- Pueden usarse en filtros, segmentadores y ejes de visualización
- Son ideales para clasificaciones, categorías y transformaciones de datos

### Sintaxis de columna calculada en TMDL

```
column 'Nombre de Columna' =
    -- Fórmula DAX que se evalúa fila por fila
    IF([Columna1] > 100, "Alto", "Bajo")
    dataType: string
    formatString: (opcional)
    displayFolder: \"Carpeta\" (opcional)
    description: \"Descripción\" (opcional)
    lineageTag: [GUID único]
```

### Workflow: Crear columna calculada

**Comando**: "Crea una columna calculada llamada 'Categoría' en la tabla FactSales"

**Proceso**:
```
1. Auto-detectar proyecto .pbip
2. Identificar tabla destino (ej: FactSales)
3. Leer archivo .tmdl de la tabla
4. Localizar sección de columnas (después de las columnas normales, antes de measures)
5. Insertar nueva columna calculada con:
   - Fórmula DAX
   - Tipo de dato apropiado
   - lineageTag único (generar GUID)
6. Mantener indentación correcta (TABS)
7. Informar al usuario
```

### Ejemplo completo de columna calculada

```tmdl
table FactSales
    lineageTag: abc-123-def-456

    column SaleID
        dataType: int64
        sourceColumn: SaleID
        lineageTag: xyz-789-abc-012

    column Amount
        dataType: double
        formatString: $#,##0.00
        sourceColumn: Amount
        lineageTag: def-345-ghi-678

    /// COLUMNA CALCULADA - Categoría de Venta
    column 'Sales Category' =
            VAR SaleAmount = FactSales[Amount]
            RETURN
                SWITCH(
                    TRUE(),
                    SaleAmount >= 10000, "Premium",
                    SaleAmount >= 1000, "Standard",
                    "Basic"
                )
        dataType: string
        displayFolder: \"Classifications\"
        description: \"Categoría de venta basada en el monto\"
        lineageTag: new-guid-generated-here

    /// COLUMNA CALCULADA - Es Venta Grande
    column 'Is Large Sale' = FactSales[Amount] > 5000
        dataType: boolean
        displayFolder: \"Flags\"
        lineageTag: another-guid-here

    measure 'Total Sales' = SUM(FactSales[Amount])
        formatString: $#,##0.00

    partition FactSales = m
        mode: import
        source =
            let
                Source = ...
            in
                Source
```

### Casos de uso comunes

#### 1. Clasificaciones y categorías
```dax
column 'Age Group' =
    SWITCH(
        TRUE(),
        Customer[Age] < 18, "Menor",
        Customer[Age] < 35, "Joven",
        Customer[Age] < 55, "Adulto",
        "Senior"
    )
    dataType: string
```

#### 2. Flags booleanos
```dax
column 'Is Active Customer' = Customer[LastPurchaseDate] >= DATE(2025, 1, 1)
    dataType: boolean
```

#### 3. Concatenaciones
```dax
column 'Full Name' = Customer[FirstName] & " " & Customer[LastName]
    dataType: string
```

#### 4. Cálculos con RELATED (navegación de relaciones)
```dax
column 'Customer Country' = RELATED(DimCustomer[Country])
    dataType: string
    displayFolder: \"Customer Info\"
```

#### 5. Extracciones de fecha
```dax
column 'Year' = YEAR(FactSales[Date])
    dataType: int64
    displayFolder: \"Time Attributes\"

column 'Month Name' = FORMAT(FactSales[Date], "MMMM")
    dataType: string
    displayFolder: \"Time Attributes\"

column 'Quarter' = "Q" & QUARTER(FactSales[Date])
    dataType: string
    displayFolder: \"Time Attributes\"
```

### Creación masiva de columnas calculadas

**Comando**: "Crea columnas de tiempo (Year, Quarter, Month) en todas las tablas de hechos"

**Proceso inteligente**:
```
1. Escanear modelo para identificar tablas de hechos
2. En cada tabla de hechos, detectar columnas de fecha
3. Para cada columna de fecha detectada:
   - Crear columna Year
   - Crear columna Quarter
   - Crear columna Month Name
   - Crear columna Month Number
   - Crear columna Weekday
4. Aplicar todos los cambios en un solo Edit por tabla
5. Actualizar documentación si existe
```

### Mejores prácticas para columnas calculadas

**✅ USAR columnas calculadas cuando:**
- Necesitas categorizar o clasificar datos
- Requieres el resultado en filtros o segmentadores
- La lógica es simple y se evalúa fila por fila
- Necesitas concatenar o transformar texto

**❌ NO USAR columnas calculadas cuando:**
- Puedes hacer el cálculo en Power Query (más eficiente)
- Necesitas agregaciones (usa medidas DAX en su lugar)
- El cálculo depende del contexto de filtro (usa medidas)
- Tienes millones de filas y memoria limitada

## 🔗 GESTIÓN DE RELACIONES

### ¿Qué son las relaciones en Power BI?

Las relaciones conectan tablas entre sí, permitiendo que los filtros se propaguen de una tabla a otra. Son fundamentales para crear modelos relacionales efectivos.

### Anatomía de una relación en TMDL

```tmdl
relationship [GUID-o-nombre-legible]
    fromColumn: TablaDe[ColumnaLlave]
    toColumn: TablaA[ColumnaLlave]
    fromCardinality: many | one
    toCardinality: one | many
    isActive: true | false
    crossFilteringBehavior: oneToMany | bothDirections | automatic
    securityFilteringBehavior: oneDirection | bothDirections
    relyOnReferentialIntegrity: false | true
```

### Tipos de cardinalidad

| Tipo | Descripción | Uso típico |
|------|-------------|------------|
| **Many-to-One** (N:1) | Múltiples filas en tabla origen → Una fila en tabla destino | Fact → Dimension |
| **One-to-Many** (1:N) | Una fila en tabla origen → Múltiples filas en tabla destino | Dimension → Fact |
| **One-to-One** (1:1) | Una fila → Una fila | Tablas complementarias |
| **Many-to-Many** (N:N) | Múltiples → Múltiples | Require tabla puente |

### Ubicación del archivo de relaciones

Las relaciones se almacenan en:
```
[proyecto].SemanticModel/definition/relationships.tmdl
```

### Estructura del archivo relationships.tmdl

```tmdl
model Model
    culture: en-US

relationship FactSales_DimCustomer
    fromColumn: FactSales[CustomerID]
    toColumn: DimCustomer[CustomerID]
    fromCardinality: many
    toCardinality: one
    isActive: true
    crossFilteringBehavior: oneToMany

relationship FactSales_DimProduct
    fromColumn: FactSales[ProductID]
    toColumn: DimProduct[ProductID]
    fromCardinality: many
    toCardinality: one
    isActive: true
    crossFilteringBehavior: oneToMany

relationship FactSales_DimDate
    fromColumn: FactSales[OrderDate]
    toColumn: DimDate[Date]
    fromCardinality: many
    toCardinality: one
    isActive: true
    crossFilteringBehavior: oneToMany

/// Relación inactiva (para fechas alternativas)
relationship FactSales_DimDate_ShipDate
    fromColumn: FactSales[ShipDate]
    toColumn: DimDate[Date]
    fromCardinality: many
    toCardinality: one
    isActive: false
    crossFilteringBehavior: oneToMany

/// Relación bidireccional (usar con precaución)
relationship FactSales_DimStore
    fromColumn: FactSales[StoreID]
    toColumn: DimStore[StoreID]
    fromCardinality: many
    toCardinality: one
    isActive: true
    crossFilteringBehavior: bothDirections
```

### Workflow: Crear una relación

**Comando**: "Crea una relación entre FactSales[CustomerID] y DimCustomer[CustomerID]"

**Proceso**:
```
1. Auto-detectar proyecto .pbip
2. Leer archivo relationships.tmdl
3. Verificar que ambas tablas y columnas existan
4. Determinar cardinalidad (detectar automáticamente):
   - Si "ID" en tabla Fact → Many-to-One
   - Si "ID" en tabla Dim → One-to-Many
5. Agregar nueva relación al archivo
6. Usar convención de nombre: [TablaOrigen]_[TablaDestino]
7. Configurar:
   - isActive: true (por defecto)
   - crossFilteringBehavior: oneToMany (por defecto)
8. Informar al usuario
```

### Workflow: Modificar una relación

**Comando**: "Cambia la relación FactSales-DimCustomer a bidireccional"

**Proceso**:
```
1. Leer relationships.tmdl
2. Buscar relación por nombre o columnas
3. Edit: cambiar crossFilteringBehavior a bothDirections
4. Advertir sobre implicaciones de rendimiento
5. Confirmar cambios
```

### Workflow: Eliminar una relación

**Comando**: "Elimina la relación inactiva entre FactSales[ShipDate] y DimDate[Date]"

**Proceso**:
```
1. Leer relationships.tmdl
2. Identificar relación exacta
3. Confirmar con usuario antes de eliminar
4. Edit: eliminar bloque completo de la relación
5. Informar cambios
```

### Creación masiva de relaciones

**Comando**: "Crea relaciones automáticas entre todas las tablas de hechos y dimensiones"

**Proceso inteligente**:
```
1. Escanear modelo completo
2. Identificar tablas de hechos (Fact*) y dimensiones (Dim*)
3. Para cada tabla de hechos:
   a. Buscar columnas que terminen en "ID" o "Key"
   b. Buscar tabla dimensión correspondiente (ej: CustomerID → DimCustomer)
   c. Verificar que columna exista en dimensión
   d. Crear relación Many-to-One
4. Preguntar confirmación antes de aplicar
5. Aplicar todas las relaciones en un solo Edit
6. Generar reporte de relaciones creadas
```

### Detección automática de cardinalidad

```
Regla 1: Si columna origen está en tabla "Fact*" → Many
Regla 2: Si columna destino está en tabla "Dim*" → One
Regla 3: Si columna es "ID" o "Key" en Dim → One
Regla 4: Si hay duplicados en origen → Many
Regla 5: Si todos son únicos en destino → One
```

### Ejemplos de comandos de relaciones

#### Crear relación simple
```
Usuario: "Crea relación entre FactOrders[CustomerID] y DimCustomers[CustomerID]"

Resultado:
relationship FactOrders_DimCustomers
    fromColumn: FactOrders[CustomerID]
    toColumn: DimCustomers[CustomerID]
    fromCardinality: many
    toCardinality: one
    isActive: true
    crossFilteringBehavior: oneToMany
```

#### Crear relación bidireccional
```
Usuario: "Crea relación bidireccional entre FactSales[StoreID] y DimStore[StoreID]"

Resultado:
relationship FactSales_DimStore
    fromColumn: FactSales[StoreID]
    toColumn: DimStore[StoreID]
    fromCardinality: many
    toCardinality: one
    isActive: true
    crossFilteringBehavior: bothDirections
```

#### Crear relación inactiva
```
Usuario: "Crea relación inactiva entre FactOrders[ShipDate] y DimDate[Date]"

Resultado:
relationship FactOrders_DimDate_ShipDate
    fromColumn: FactOrders[ShipDate]
    toColumn: DimDate[Date]
    fromCardinality: many
    toCardinality: one
    isActive: false
    crossFilteringBehavior: oneToMany
```

### Convenciones de nombres para relaciones

```
Formato estándar: [TablaOrigen]_[TablaDestino]
Formato con campo alternativo: [TablaOrigen]_[TablaDestino]_[Campo]

Ejemplos:
- FactSales_DimCustomer
- FactSales_DimProduct
- FactOrders_DimDate_OrderDate
- FactOrders_DimDate_ShipDate (inactiva)
```

### Mejores prácticas para relaciones

**✅ HACER:**
- Usar relaciones Many-to-One de Fact a Dimension
- Mantener filtrado unidireccional (oneToMany) por defecto
- Crear relaciones inactivas para fechas alternativas
- Usar nombres descriptivos para relaciones
- Documentar relaciones bidireccionales

**❌ EVITAR:**
- Relaciones bidireccionales innecesarias (impactan rendimiento)
- Relaciones Many-to-Many sin tabla puente
- Múltiples relaciones activas entre las mismas tablas
- Cadenas largas de relaciones
- Relaciones circulares

### Análisis de relaciones existentes

**Comando**: "Analiza todas las relaciones del modelo"

**Salida**:
```markdown
## Análisis de Relaciones

### Relaciones activas (8)
1. FactSales → DimCustomer (Many-to-One, unidireccional) ✅
2. FactSales → DimProduct (Many-to-One, unidireccional) ✅
3. FactSales → DimDate (Many-to-One, unidireccional) ✅
4. FactSales → DimStore (Many-to-One, **bidireccional**) ⚠️

### Relaciones inactivas (2)
1. FactOrders → DimDate (ShipDate) ✅ Uso: USERELATIONSHIP()
2. FactOrders → DimDate (DeliveryDate) ✅ Uso: USERELATIONSHIP()

### Advertencias
⚠️ Relación bidireccional detectada: FactSales → DimStore
   - Impacto: Puede afectar rendimiento
   - Sugerencia: Evaluar si es realmente necesaria

### Recomendaciones
💡 Todas las relaciones siguen patrón Fact → Dimension
💡 Esquema detectado: Estrella ⭐
✅ No se detectaron relaciones circulares
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

### Crear columna calculada

```
1. Auto-detectar proyecto
2. Identificar tabla destino
3. Leer archivo .tmdl de la tabla
4. Localizar posición (después de columnas normales, antes de measures)
5. Insertar columna calculada con:
   - Fórmula DAX apropiada
   - dataType correcto
   - lineageTag único (generar GUID)
6. Mantener indentación (TABS)
7. Verificar sintaxis DAX
8. Informar al usuario
```

### Crear relación

```
1. Auto-detectar proyecto
2. Leer relationships.tmdl
3. Verificar existencia de tablas y columnas
4. Determinar cardinalidad automáticamente
5. Insertar nueva relación con:
   - Nombre descriptivo
   - Configuración apropiada (isActive, crossFiltering)
6. Informar al usuario
```

### Modificar relación existente

```
1. Leer relationships.tmdl
2. Buscar relación por nombre o columnas
3. Edit: modificar propiedades solicitadas
4. Advertir sobre impactos si es bidireccional
5. Confirmar cambios
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

### Estructura de columna calculada
```
column 'Nombre de Columna' =
    -- DAX que se evalúa fila por fila
    IF([Columna1] > 100, "Alto", "Bajo")
    dataType: string
    formatString: (opcional)
    displayFolder: "Carpeta" (opcional)
    description: "Descripción" (opcional)
    lineageTag: [GUID único]
```

### Estructura de relación
```
relationship [NombreRelacion]
    fromColumn: TablaOrigen[Columna]
    toColumn: TablaDestino[Columna]
    fromCardinality: many
    toCardinality: one
    isActive: true
    crossFilteringBehavior: oneToMany
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

### Funciones para medidas
- `SUM()`, `AVERAGE()`, `MIN()`, `MAX()` - Agregaciones
- `COUNT()`, `COUNTROWS()`, `DISTINCTCOUNT()` - Conteos
- `CALCULATE()` - Modificar contexto de filtro
- `FILTER()` - Filtrar tabla
- `DIVIDE()` - División segura (evita /0)
- `VALUES()`, `DISTINCT()` - Valores únicos
- `RELATED()`, `RELATEDTABLE()` - Navegación de relaciones
- `SAMEPERIODLASTYEAR()`, `DATEADD()` - Time intelligence

### Funciones para columnas calculadas
- `RELATED()` - Obtener valor de tabla relacionada (lado "uno")
- `RELATEDTABLE()` - Obtener tabla relacionada (lado "muchos")
- `EARLIER()` - Referenciar fila en contexto externo
- `IF()`, `SWITCH()` - Lógica condicional
- `FORMAT()` - Formatear valores como texto
- `YEAR()`, `MONTH()`, `DAY()` - Extraer partes de fecha
- `CONCATENATE()` o `&` - Unir textos
- `LEFT()`, `RIGHT()`, `MID()` - Manipulación de texto
- `LOOKUPVALUE()` - Búsqueda personalizada sin relación

### Carpetas estándar (displayFolder)
- "Base Metrics" - Métricas fundamentales
- "KPIs" - Indicadores clave
- "Time Intelligence" - Medidas temporales
- "Time Intelligence/MoM" - Month over Month
- "Time Intelligence/YoY" - Year over Year
- "Calculations" - Cálculos derivados
- "Aggregations" - Sumas y conteos
- "Classifications" - Categorías y clasificaciones (columnas calculadas)
- "Flags" - Indicadores booleanos (columnas calculadas)
- "Time Attributes" - Atributos de tiempo (columnas calculadas)
