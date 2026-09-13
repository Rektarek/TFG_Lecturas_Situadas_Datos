# Lecturas situadas y performativas

Repositorio asociado al TFG **Lecturas situadas y performativas: diseño e implementación de un instrumento interactivo**.

El prototipo trabaja separando dos elementos: el **corpus documental** y la **lectura situada** construida sobre él. Ambos se almacenan en archivos JSON independientes y se conectan mediante identificadores estables.

Este repositorio contiene los archivos utilizados en el caso desarrollado durante el TFG y documenta la estructura necesaria para construir otros corpus y lecturas compatibles con el instrumento.

La implementación completa de la aplicación se mantiene actualmente en un repositorio privado.

---

## Repository contents

```text
.
├── README.md
└── examples/
    ├── corpus_padron.json
    └── tarek_padron.json
```

`corpus_padron.json` contiene el corpus documental utilizado en el caso del padrón municipal.

`tarek_padron.json` contiene la lectura construida sobre ese corpus: posición de lectura, operaciones, asignaciones, residuos, estimaciones y relaciones entre fragmentos.

Los dos archivos corresponden al caso real utilizado durante el TFG y funcionan también como referencia para preparar nuevos datos.

---

## Data model

### Corpus

El corpus contiene el material documental disponible para una lectura.

Su estructura básica es:

```json
{
  "id": "padron-es-2015-2020",
  "version": "2026-09-11",
  "object": {
    "label": "Condiciones de acceso e inscripción al padrón municipal"
  },
  "scope": {},
  "fragments": []
}
```

Cada fragmento tiene un identificador único y conserva su procedencia documental:

```json
{
  "id": "pad2015_a01",
  "source": {
    "title": "...",
    "apparatus": "administracion",
    "producer": "...",
    "material_base": "...",
    "date": "2015-01-30",
    "format": "...",
    "url": "..."
  },
  "text": {
    "excerpt": "...",
    "location": "apartado 1, punto 1",
    "verbatim": true
  }
}
```

El `id` del fragmento es el vínculo utilizado posteriormente por la lectura. La lectura no duplica el contenido documental, sino que referencia los fragmentos existentes en el corpus.

La fragmentación depende del material trabajado. En el caso del padrón se conserva, siempre que es posible, la propia estructura de apartados y subapartados de las resoluciones.

---

### Reading

La lectura se almacena en un archivo separado.

Su estructura general es:

```json
{
  "id": "tarek-padron-reading-01",
  "corpus_id": "padron-es-2015-2020",
  "source_version": "2026-09-11",
  "version": "2026-09-11",
  "position": {},
  "grid": {},
  "fragments": {},
  "relations": []
}
```

`corpus_id` debe coincidir con el identificador del corpus utilizado.

`position` declara desde qué problema o posición se construye la lectura.

`grid` define las operaciones utilizadas en esa lectura.

En el caso desarrollado durante el TFG:

```json
"operations": [
  "fijacion",
  "omision",
  "banalizacion",
  "deriva",
  "desactivacion_extension"
]
```

Cada operación dispone también de una definición explícita dentro de `operation_definitions`.

Estas operaciones pertenecen a la lectura del padrón y no constituyen una taxonomía universal del instrumento. Otra lectura puede trabajar con otra grilla, siempre que mantenga la misma lógica de correspondencia entre las operaciones declaradas y las asignaciones realizadas sobre los fragmentos.

---

## Fragment assignments

Dentro de la lectura, cada fragmento puede recibir una o varias asignaciones.

Ejemplo:

```json
"pad2015_a01": {
  "status": "assigned",
  "analysis": {
    "thematic_category": "definicion",
    "structuring_terms": [
      "Padrón municipal",
      "registro administrativo"
    ]
  },
  "assignments": [
    {
      "operation": "fijacion",
      "epistemic_status": "localizable",
      "evidence": {
        "fragment_id": "pad2015_a01",
        "location": "apartado 1, punto 1"
      }
    }
  ],
  "estimates": {
    "formalization": {
      "value": 0.76,
      "status": "estimado"
    }
  }
}
```

Las asignaciones representan actos de lectura, no propiedades inherentes al documento.

Un mismo fragmento puede recibir más de una operación y debe conservar la evidencia que permita volver a su localización documental.

---

## Residue

No todos los fragmentos leídos deben ser asignados a una operación.

Cuando un fragmento ha sido leído pero no puede incorporarse a la grilla sin forzar la interpretación, puede conservarse como residuo:

```json
{
  "status": "residue",
  "assignments": [],
  "residue": {
    "reason": "Fragmento leído que no se asigna a ninguna operación de la grilla sin forzar la lectura."
  }
}
```

El residuo forma parte de la lectura. No se considera un error pendiente de clasificación.

---

## Epistemic status and estimates

La lectura del TFG utiliza tres estatutos epistémicos:

```text
documental
localizable
estimado
```

Estos estatutos indican bajo qué condiciones se sostiene una asignación o relación. No expresan un grado de verdad.

La lectura incluye además una dimensión estimada de formalización:

```json
"estimated_dimensions": {
  "formalization": {
    "range": [0, 1],
    "epistemic_status": "estimado",
    "description": "..."
  }
}
```

Los valores de esta dimensión se utilizan de forma comparativa dentro de la lectura y no deben interpretarse como una medida objetiva del documento.

---

## Relations

La lectura puede registrar relaciones entre fragmentos.

Ejemplo:

```json
{
  "from": "pad2015_a16",
  "to": "pad2020_a21",
  "type": "fragmentacion_reglamentaria",
  "epistemic_status": "localizable",
  "evidence": "..."
}
```

`from` y `to` deben corresponder a identificadores existentes en el corpus.

El campo `type` describe el tipo de relación inscrita entre ambos fragmentos.

En el caso del padrón aparecen, entre otros:

```text
misma_cadena_procedimental
categoria_complementaria
convergencia_operacion
desarrollo_tematico
asimetria_documental
convergencia_tematica
fragmentacion_reglamentaria
deriva_terminologica
deriva_normativa
```

Estos tipos corresponden a esta lectura concreta y pueden cambiar en otros casos.

---

## Building a new case

La forma más sencilla de preparar un nuevo caso es partir de los archivos incluidos en `examples/`.

1. Duplicar `corpus_padron.json`.
2. Crear un nuevo `id` y actualizar la versión.
3. Definir el objeto y el alcance documental.
4. Sustituir los fragmentos por los del nuevo corpus, manteniendo identificadores únicos.
5. Duplicar `tarek_padron.json`.
6. Hacer coincidir `corpus_id` con el nuevo corpus.
7. Definir la posición desde la que se realiza la lectura.
8. Definir la grilla de operaciones.
9. Añadir asignaciones, residuos y estimaciones cuando corresponda.
10. Añadir relaciones entre fragmentos.
11. Comprobar que todas las referencias utilizadas por la lectura existen en el corpus.

Una vez preparados respetando esta estructura, los archivos pueden importarse en el instrumento y utilizarse dentro del flujo de proyección, representación, trazabilidad y comparación.

---

## Example included in this repository

El ejemplo incluido corresponde al caso de estudio desarrollado durante el TFG sobre las condiciones de acceso e inscripción al padrón municipal.

El corpus reúne fragmentos procedentes de las resoluciones seleccionadas de 2015 y 2020.

La lectura incluida utiliza una posición concreta, una grilla de cinco operaciones, tres estatutos epistémicos y una dimensión estimada de formalización.

Los resultados presentados en la memoria corresponden específicamente a esta combinación de corpus y lectura y no deben entenderse como resultados generales del instrumento.

---

## Project status

La versión completa del prototipo utilizada durante el TFG se conserva en un repositorio privado.

Este repositorio público se centra en la estructura de datos que necesita preparar un usuario para trabajar con el instrumento: corpus, lectura y referencias entre ambos.

La publicación del código completo queda fuera del alcance de esta fase del proyecto.
