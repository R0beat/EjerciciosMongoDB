# Ejercicios

## Comando para importar datos

```js
mongoimport --jsonArray --file ruta/archivo --db EJERCICIOS --collection declaraciones
```

## Ejercicios a realizar

### 1- Filtrar datos con __$match__

Filtrar datos
    * Tengan como instituci&#243;n __"Instituto Electoral del Estado de M&#233;xico"__
      Donde el campo __declaracion.situacionPatrimonial.ingresos.totalIngresosConclusionNetos.valor__ sea > 30000.

```js
db.declaraciones.aggregate(
    [
        {
            $match: {
                "declaracion.situacionPatrimonial.datosEmpleoCargoComision.nombreEntePublico": "Instituto Electoral del Estado de México",
                "declaracion.situacionPatrimonial.ingresos.totalIngresosConclusionNetos.valor": { $gt: 30000 }
            }
        },
    ]
)
```

### 2- Combinar filtros y proyecci&#243;n.

Usa __$match__ y __$project__ para mostrar únicamente:
  * Los nombres completos de los declarantes que tienen nacionalidad "MX".
  * Su correo electrónico personal y su nivel de empleo/cargo.

```js
db.declaraciones.aggregate(
    [
        {
            $match: {
                "declaracion.situacionPatrimonial.datosGenerales.nacionalidad": "MX"
            }
        },
        {
            $project:{
                "datosGenerales": "$declaracion.situacionPatrimonial.datosGenerales",
                "cargo":"$declaracion.situacionPatrimonial.datosEmpleoCargoComision.empleoCargoComision"
            }
        },
        {
            $project:{
                "nombreCompleto":{
                    $concat: ["$datosGenerales.nombre", " ","$datosGenerales.primerApellido"," ","$datosGenerales.segundoApellido"]
                },
                "correoElectronico":"$datosGenerales.correoElectronico.personal",
                "cargo":"$cargo"
            } 
        }
    ]
)
```

### 3- Desanidar un arreglo.

Usa __$unwind__ para mostrar cada registro de escolaridad (declaracion.situacionPatrimonial.datosCurricularesDeclarante.escolaridad) como un documento individual, mostrando solo: _nivel.valor_

```js

db.declaraciones.aggregate(
    [
        {
            $project: {
                "datos": "$declaracion.situacionPatrimonial.datosCurricularesDeclarante"
            }
        },
        {
            $unwind: "$datos.escolaridad",
            /*$unwind: {
                path: "$datos.escolaridad",
                preserveNullAndEmptyArrays: true
            }
            */
        },
        {
            $project: {
                "nivel": "$datos.escolaridad.nivel.valor",
                "institucion": "$datos.escolaridad.institucionEducativa.nombre",
                "estatus": "$datos.escolaridad.estatus",
            }
        }
    ]
)
```

### 4- Calcular promedio de valores.
Agrupa __($group)__ todos los documentos por _"metadata.institucion"_ y calcula:
1. El promedio de ingresos totales netos.
2. La cantidad de declaraciones por institución.

```js
db.declaraciones.aggregate(
    [
        {
            $group: {
                _id: "$metadata.institucion",
                "totalDeclaraciones": {$sum:1},
                "totalNeto":{
                    $avg: "$declaracion.situacionPatrimonial.ingresos.totalIngresosAnualesNetos.valor"
                },
            }
        }
    ]
)
```

### 5- Agrega un nuevo campo "totalIngresosGlobales"
Que sea la suma de:
1. ingresoConclusionNetoDeclarante.valor
2. ingresoConclusionNetoParejaDependiente.valor
3. Luego, muestra solo el nombre y este nuevo valor.

```js
db.declaraciones.aggregate(
    [
        {
            $addFields: {
                "totalIngresosGlobales": {
                    "declarante": {$sum: "$declaracion.situacionPatrimonial.ingresos.ingresoConclusionNetoDeclarante.valor"},
                    "parejaDeclarante":{$sum: "$declaracion.situacionPatrimonial.ingresos.ingresoConclusionNetoParejaDependiente.valor"}
                }
            }
        },
        {
            $project: {
                "totalIngresosGlobales": 1
            }
        }
    ]
)
```