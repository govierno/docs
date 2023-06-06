---
title: Carga de Datos a la plataforma de Datos Abiertos
description: 
published: true
date: 2023-06-05T16:04:57.752Z
tags: 
editor: markdown
dateCreated: 2023-06-02T16:15:16.418Z
---

# Resumen

> Actualmente la carga de datos a la base de datos del sitio de datos abiertos se realiza de forma manual. 
{.is-info}

> 💡 Está pendiente implementar una solución automática para la carga de información.
{.is-warning}


# Carga de datos manual

## Creación de tabla en base MySQL

> La tabla se creó con el siguiente esquema: (Como hay un variedad de valores distintos en cada parámetros, la mayoría se creó como VARCHAR. Los que interesan son principalmente `precioNeto` y `cantidad`, que se crearon como DECIMAL)
{.is-warning}


```sql
-- compraspublicas.ordenesdecompra definition

CREATE TABLE `OC2022` (
  `ID` varchar(255) DEFAULT NULL,
  `Codigo` varchar(255) NOT NULL,
  `Link` text DEFAULT NULL,
  `Nombre` text DEFAULT NULL,
  `Descripcion/Obervaciones` text DEFAULT NULL,
  `Tipo` text DEFAULT NULL,
  `ProcedenciaOC` varchar(255) DEFAULT NULL,
  `EsTratoDirecto` varchar(255) DEFAULT NULL,
  `EsCompraAgil` varchar(255) DEFAULT NULL,
  `CodigoTipo` varchar(255) DEFAULT NULL,
  `CodigoAbreviadoTipoOC` text DEFAULT NULL,
  `DescripcionTipoOC` text DEFAULT NULL,
  `codigoEstado` int(11) DEFAULT NULL,
  `Estado` text DEFAULT NULL,
  `codigoEstadoProveedor` int(11) DEFAULT NULL,
  `EstadoProveedor` varchar(255) DEFAULT NULL,
  `FechaCreacion` date DEFAULT NULL,
  `FechaEnvio` date DEFAULT NULL,
  `FechaSolicitudCancelacion` varchar(255) DEFAULT NULL,
  `fechaUltimaModificacion` varchar(255) DEFAULT NULL,
  `FechaAceptacion` varchar(255) DEFAULT NULL,
  `FechaCancelacion` varchar(255) DEFAULT NULL,
  `tieneItems` int(11) DEFAULT NULL,
  `PromedioCalificacion` varchar(255) DEFAULT NULL,
  `CantidadEvaluacion` varchar(255) DEFAULT NULL,
  `MontoTotalOC` varchar(255) DEFAULT NULL,
  `TipoMonedaOC` varchar(255) DEFAULT NULL,
  `MontoTotalOC_PesosChilenos` varchar(255) DEFAULT NULL,
  `Impuestos` varchar(255) DEFAULT NULL,
  `TipoImpuesto` varchar(255) DEFAULT NULL,
  `Descuentos` varchar(255) DEFAULT NULL,
  `Cargos` varchar(255) DEFAULT NULL,
  `TotalNetoOC` varchar(255) DEFAULT NULL,
  `CodigoUnidadCompra` int(11) DEFAULT NULL,
  `RutUnidadCompra` varchar(255) DEFAULT NULL,
  `UnidadCompra` varchar(255) DEFAULT NULL,
  `CodigoOrganismoPublico` int(11) DEFAULT NULL,
  `OrganismoPublico` varchar(255) DEFAULT NULL,
  `sector` varchar(255) DEFAULT NULL,
  `ActividadComprador` varchar(255) DEFAULT NULL,
  `CiudadUnidadCompra` varchar(255) DEFAULT NULL,
  `RegionUnidadCompra` varchar(255) DEFAULT NULL,
  `PaisUnidadCompra` varchar(255) DEFAULT NULL,
  `CodigoSucursal` varchar(255) DEFAULT NULL,
  `RutSucursal` varchar(255) DEFAULT NULL,
  `Sucursal` varchar(255) DEFAULT NULL,
  `CodigoProveedor` varchar(255) DEFAULT NULL,
  `NombreProveedor` varchar(255) NOT NULL,
  `ActividadProveedor` varchar(255) DEFAULT NULL,
  `ComunaProveedor` varchar(255) DEFAULT NULL,
  `RegionProveedor` varchar(255) DEFAULT NULL,
  `PaisProveedor` varchar(255) DEFAULT NULL,
  `Financiamiento` varchar(255) DEFAULT NULL,
  `PorcentajeIva` varchar(255) DEFAULT NULL,
  `Pais` varchar(255) DEFAULT NULL,
  `TipoDespacho` int(11) DEFAULT NULL,
  `FormaPago` varchar(255) DEFAULT NULL,
  `CodigoLicitacion` varchar(255) DEFAULT NULL,
  `Codigo_ConvenioMarco` varchar(255) DEFAULT NULL,
  `IDItem` varchar(255) NOT NULL,
  `codigoCategoria` varchar(255) DEFAULT NULL,
  `Categoria` text DEFAULT NULL,
  `codigoProductoONU` varchar(255) DEFAULT NULL,
  `NombreroductoGenerico` text DEFAULT NULL,
  `RubroN1` text DEFAULT NULL,
  `RubroN2` text DEFAULT NULL,
  `RubroN3` text DEFAULT NULL,
  `EspecificacionComprador` text DEFAULT NULL,
  `EspecificacionProveedor` text DEFAULT NULL,
  `cantidad` decimal(65,16) DEFAULT NULL,
  `UnidadMedida` varchar(255) DEFAULT NULL,
  `monedaItem` varchar(255) DEFAULT NULL,
  `precioNeto` decimal(65,16) DEFAULT NULL,
  `totalCargos` varchar(255) DEFAULT NULL,
  `totalDescuentos` varchar(255) DEFAULT NULL,
  `totalImpuestos` varchar(255) DEFAULT NULL,
  `totalLineaNeto` varchar(255) DEFAULT NULL,
  `Forma de Pago` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`NombreProveedor`,`Codigo`,`IDItem`),
  UNIQUE KEY `Comprador` (`OrganismoPublico`,`Codigo`,`IDItem`),
  UNIQUE KEY `Rubros` (`RubroN1`,`RubroN2`,`RubroN3`,`OrganismoPublico`,`Codigo`,`IDItem`) USING HASH,
  FULLTEXT KEY `NombreProveedor` (`NombreProveedor`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci;
```

## Descarga de datos

La descarga de la información se realiza directamente desde el sitio de [datos abiertos](https://datos-abiertos.chilecompra.cl/):

![2023-06-02_12-12.png](/images/2023-06-02_12-12.png)

Luego, desde la consola descargamos y descomprimimos el archivo:

```bash
wget https://transparenciachc.blob.core.windows.net/oc-da/2023-5.zip
unzip 2023-5.zip
```

## Corrección de datos

> Los datos no vienen bien estructurados. Falta corregirlos (ajustarlos) un poco.
{.is-warning}

### Remover saltos de línea, nulos, reemplazar comma por punto

```bash
find . -type f -name '*.csv' -print -exec sed -i ':a;N;$!ba;s/\r\r\n/ /g' {} \;
find . -type f -name '*.csv' -print -exec iconv -f ISO-8859-14 -t UTF-8 {} -o {}.utf8 \;
find . -type f -name '*.utf8' -print -exec sed -i -E 's/\;([0-9]+)\,([0-9]+)\;/\;\1\.\2\;/g' {} \;
find . -type f -name '*.utf8' -print -exec sed -i -E 's/\;NA\;/\;NULL\;/g' {} \;
```

## Modificar números

> Muchas cifras o números de las ordenes de compra tenían la nomenclatura "1,9e+08" para representar el número "190000000", asi que tuvimos que reemplazar las distintas ocurrencias utilizando el compando `sed`.
{.is-warning}

```bash
find . -type f -name '*.utf8' -print -exec sed -i -E 's/\;([0-9])\,([0-9])e\+10\;/\;\1\2000000000\;/g; s/\;([0-9])\,([0-9])([0-9])e\+09\;/\;\1\2\30000000\;/g; s/\;([0-9])\,([0-9])e\+07\;/\;\1\2000000\;/g; s/\;([0-9])\,([0-9])([0-9])e\+08\;/\;\1\2\3000000\;/g; s/\;([0-9])\,([0-9])([0-9])([0-9])e\+09\;/\;\1\2\3\4000000\;/g; s/\;([0-9])e\+08\;/\;\100000000\;/g; s/\;([0-9])\,([0-9])e\+08\;/\;\1\20000000\;/g; s/\;([0-9])\,([0-9])e\+09;/\;\1\200000000\;/g; s/\;([0-9])e\+06\;/\;\1000000\;/g; s/\;([0-9])e\+05\;/\;\100000\;/g; s/\;([0-9])e\+07\;/\;\10000000\;/g; s/\;([0-9])e\-04\;/\;0\.000\1\;/g; s/\;([0-9])\,([0-9])([0-9])([0-9])([0-9])e\+10\;/\;\1\2\3\4\5000000\;/g; s/\;([0-9])e\-05\;/\;0\.0000\1\;/g; s/\;([0-9])e\+09\;/\;\1000000000\;/g' {} \;
```

## Copia del archivo al servidor de base de datos

Ya podemos copiar los archivos al contenedor de la base de datos para su importación:

```bash
find . -type f -name '*.utf8' -print -exec kubectl cp {} databases/mariadb-10-0:/bitnami/mariadb/data/{} \;
```

## Importación de datos a la tabla

> Luego, desde el servidor de MySQL, ejecutamos la importación de los datos:
{.is-success}


```sql
LOAD DATA INFILE "/bitnami/mariadb/data/2023-5.csv.utf8" 
INTO TABLE OC2022
CHARACTER SET utf8
COLUMNS TERMINATED BY ';'
OPTIONALLY ENCLOSED BY '"'
ESCAPED BY '"'
LINES TERMINATED BY '\r\n'
IGNORE 1 LINES;
```

> A veces hay que eliminar la línea `ESCAPED BY '"'` , de lo contrario la importación no funciona.
{.is-warning}

> Lo mismo con los saltos de línea... a veces hay que poner `LOAD DATA INFILE "/bitnami/mariadb/data/2023-5.csv" IGNORE` para que pueda cargar.
{.is-warning}


## Ronombrar moneda UF

> Por alguna razón, el tipo de moneda de la UF está como "CLP", luego debemos cambiarlo a "UF"
{.is-info}

```sql
UPDATE OC2022
SET monedaItem = REPLACE(monedaItem, 'CLF', 'UF')
```

# Validación

> Verificamos que los datos ya están cargados en la tabla OC2022:
{.is-info}

# Troubleshooting

> Durante la importación a la tabla, puede aparecer el error:
>
> **SQL Error [1265] [01000]: (conn=2864163) Data truncated for column 'precioNeto' at row 407010**
{.is-danger}

> A veces saler errores relacionados con el valor del `precioNeto`en cuyo caso hay que buscar con `a`.




