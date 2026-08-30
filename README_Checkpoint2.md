# Checkpoint 2 – Modelo de datos y medidas DAX en Power BI

## Descripción

Este proyecto corresponde al segundo checkpoint del módulo de Data Analytics. El objetivo fue transformar el dataset previamente limpio en un modelo analítico funcional en Power BI, incorporando relaciones entre tablas, una dimensión calendario y medidas DAX para análisis temporal y comparativo.

El archivo final del proyecto es:

`Juarez_Juan_Checkpoint2.pbix`

## Modelo de datos

Se trabajó con un esquema compuesto por las siguientes tablas:

- `Fact_Ventas`: tabla de hechos con las transacciones de ventas.
- `Dim_Clientes`: dimensión de clientes.
- `Dim_Productos`: dimensión de productos.
- `Dim_Categorias`: dimensión de categorías.
- `Dim_Fechas`: tabla calendario utilizada para inteligencia temporal.
- `_Medidas`: tabla destinada exclusivamente a las medidas DAX.

Las relaciones requeridas en el modelo son:

| Tabla origen | Campo | Tabla destino | Campo | Cardinalidad |
|---|---|---|---|---|
| Dim_Clientes | id_cliente | Fact_Ventas | id_cliente | 1:N |
| Dim_Productos | id_producto | Fact_Ventas | id_producto | 1:N |
| Dim_Categorias | id_categoria | Dim_Productos | id_categoria | 1:N |
| Dim_Fechas | Date | Fact_Ventas | fecha_venta | 1:N |

Todas las relaciones se configuraron como activas y con dirección de filtro única desde la dimensión hacia la tabla relacionada.

## Ajuste realizado sobre categorías

Durante la construcción del modelo se detectó que la tabla `Dim_Productos` contenía el nombre de la categoría, pero no su identificador.

Para mantener una estructura relacional correcta, se incorporó `id_categoria` a `Dim_Productos`, utilizando la correspondencia entre el nombre de categoría de productos y la tabla `Dim_Categorias`.

De esta forma, la relación quedó definida mediante claves:

`Dim_Categorias[id_categoria]` → `Dim_Productos[id_categoria]`

Este ajuste evita relacionar tablas mediante campos de texto y mejora la consistencia del modelo, ya que el identificador funciona como clave estable mientras que el nombre de una categoría puede modificarse.

## Tabla calendario

Se creó la tabla `Dim_Fechas` mediante DAX utilizando el rango mínimo y máximo de fechas registrado en `Fact_Ventas`:

```DAX
Dim_Fechas =
CALENDAR(
    MIN(Fact_Ventas[fecha_venta]),
    MAX(Fact_Ventas[fecha_venta])
)
```

Luego se agregaron las columnas necesarias para el análisis temporal:

```DAX
Año = YEAR(Dim_Fechas[Date])
Mes Número = MONTH(Dim_Fechas[Date])
Mes Nombre = FORMAT(Dim_Fechas[Date], "MMMM")
Trimestre = "T" & QUARTER(Dim_Fechas[Date])
Semana = WEEKNUM(Dim_Fechas[Date])
```

Finalmente, `Dim_Fechas` fue marcada como tabla de fechas utilizando la columna `Date`.

## Medidas DAX

Se creó una tabla exclusiva para almacenar las medidas del modelo.

### Total Ventas

```DAX
Total Ventas =
SUM(Fact_Ventas[total_venta])
```

### Ventas Online

```DAX
Ventas Online =
CALCULATE(
    [Total Ventas],
    Fact_Ventas[canal] = "Online"
)
```

### Ventas YTD

```DAX
Ventas YTD =
TOTALYTD(
    [Total Ventas],
    Dim_Fechas[Date]
)
```

### Ventas LY

```DAX
Ventas LY =
CALCULATE(
    [Total Ventas],
    SAMEPERIODLASTYEAR(Dim_Fechas[Date])
)
```

### % Crecimiento Anual

```DAX
% Crecimiento Anual =
VAR VentasActual = [Total Ventas]
VAR VentasAnterior = [Ventas LY]
RETURN
    DIVIDE(
        VentasActual - VentasAnterior,
        VentasAnterior
    )
```

La medida `% Crecimiento Anual` fue configurada con formato porcentaje.

## Validación

Se creó una página denominada `Validación` con una matriz configurada de la siguiente manera:

- Filas: `Dim_Fechas[Mes Nombre]`
- Columnas: `Dim_Fechas[Año]`
- Valores:
  - `Total Ventas`
  - `Ventas YTD`
  - `Ventas LY`
  - `% Crecimiento Anual`

Esta matriz permite comprobar:

- El acumulado anual de `Ventas YTD`.
- La comparación de ventas contra el mismo período del año anterior mediante `Ventas LY`.
- El comportamiento de `BLANK` cuando no existe un período comparable.
- El crecimiento o decrecimiento porcentual entre años.

## Herramientas utilizadas

- Microsoft Excel
- Power Query
- Power BI Desktop
- DAX
- GitHub

## Autor

**Juan Juarez**  
Proyecto realizado como parte del curso de Data Analytics.
