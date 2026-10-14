# test_buenas_practica_IA

Repositorio plantilla para experimentar con buenas prácticas de creación de agentes de **GitHub Copilot en VS Code**. Reúne ejemplos de configuración, instrucciones compartidas, una skill reutilizable y pruebas de invocación entre agentes.

El objetivo es disponer de una base sencilla para aprender, probar y adaptar agentes: definir una tarea concreta, limitar las herramientas, describir el procedimiento y verificar el resultado antes de darlo por terminado.

## Estructura del repositorio

```text
test_buenas_practica_IA/
├── .github/
│   ├── agents/
│   │   ├── agente-maestro.agent.md
│   │   ├── agente-maestro-skill.agent.md
│   │   ├── agente-maestro-test.agent.md
│   │   └── agente-tests-esclavo.agent.md
│   ├── instructions/
│   │   └── general.instructions.md
│   ├── skills/
│   │   └── escribir-txt/
│   │       └── SKILL.md
│   └── copilot-instructions.md
├── .vscode/
│   └── mcp.json
├── estructura-agente-ampliada.excalidraw
├── estructura-agente-ampliada.png
├── LICENSE
└── README.md
```

Este árbol combina los archivos aportados con las ubicaciones indicadas por el autor. Las capturas muestran las carpetas principales, el documento Excalidraw, su representación gráfica, la licencia y el README; no muestran el contenido de todas las subcarpetas. El nombre `.png` corresponde al diagrama adjunto.

Las pruebas pueden generar además `ejemplos/<nombre>.txt` y `logEsclavo1.txt` en la raíz. Son salidas de los agentes, no archivos cuya existencia inicial esté acreditada por las capturas.

## Los cuatro agentes

El nombre del archivo y el campo YAML `name` no siempre coinciden. Para identificar el agente en el selector de Copilot, conviene consultar su `name`.

| Archivo en `.github/agents/` | `name` declarado | Función |
| --- | --- | --- |
| `agente-maestro.agent.md` | `Agente maestro` | Probar la invocación de subagentes. |
| `agente-maestro-test.agent.md` | `crear-txt` | Ejemplo principal de buenas prácticas: crear y verificar un archivo de texto. |
| `agente-maestro-skill.agent.md` | `crear-txt-skill` | Variante que utiliza la skill `escribir-txt`. |
| `agente-tests-esclavo.agent.md` | `agente-qa-esclavo` | Dejar una señal escrita para probar que se ha invocado. |

### 1. `agente-maestro.agent.md`: invocación de subagentes

**Propósito explicado por el autor:** ejemplo centrado en las diferencias de la cabecera, especialmente en el campo `agents`.

**Configuración observada:**

```yaml
name: Agente maestro
description: llama agentes esclavos
tools: ['read', 'search', 'edit', 'agent']
agents: ['agente-qa-esclavo']
user-invocable: true
disable-model-invocation: false
```

Además de la cabecera, contiene una instrucción breve que pide invocar tanto a `agente-qa-esclavo` como a `agente-qa-esclavo-2`.

**Diferencia que debe tenerse en cuenta:** la lista `agents` solo incluye `agente-qa-esclavo`, y entre los cuatro archivos aportados no existe una definición del segundo subagente. El ejemplo no permite dar por válida la ejecución de ambas llamadas. Antes de reutilizarlo, hay que alinear el cuerpo con la lista y con los agentes disponibles.

### 2. `agente-maestro-test.agent.md`: ejemplo principal

**Propósito explicado por el autor:** es el agente principal sobre el que se han realizado las pruebas y donde se concentran las buenas prácticas.

**Configuración observada:** su nombre es `crear-txt`; declara `tools: ['read', 'edit']`, `agents: []`, `user-invocable: true` y `disable-model-invocation: false`.

Su tarea es crear **un único archivo `.txt` dentro de `ejemplos/`**, sin sobrescribir archivos existentes. El procedimiento indicado es:

1. Leer `general.instructions.md` y realizar las acciones que indique.
2. Obtener el nombre y el contenido, utilizando los valores por defecto cuando falten.
3. Validar el nombre y comprobar si el destino existe, incluidos archivos ocultos o ignorados.
4. Si existe, detenerse e informar. Si no existe, crear la carpeta cuando sea necesario y escribir el archivo en UTF-8.
5. Leer el archivo creado, comprobar su contenido e informar de la ruta.

Valores por defecto:

| Entrada | Valor |
| --- | --- |
| Nombre | `ejemplo.txt` |
| Contenido | `Hola, este es mi primer archivo creado con un agente.` |

El nombre debe terminar en `.txt` y no puede contener rutas ni separadores. El contenido se trata como texto literal. El agente no debe modificar ni borrar archivos existentes, usar terminal o red, ni instalar dependencias.

Debe detenerse si el nombre es inválido, falta el workspace, hay varias raíces sin un destino claro o no puede comprobar la existencia del archivo. Si una herramienta falla, debe informar del error sin afirmar que ha terminado. La confirmación de éxito exige verificar el resultado con `#tool:read`.

Salida prevista:

```text
Archivo creado: ejemplos/<nombre>
```

La referencia a `general.instructions.md` aparece solo por su nombre; la ubicación indicada por el autor es `.github/instructions/general.instructions.md`. Para adaptar la plantilla conviene hacer esa referencia inequívoca.

### 3. `agente-maestro-skill.agent.md`: reutilización mediante una skill

**Propósito explicado por el autor:** variante del agente principal que llama a una skill para realizar la creación del archivo.

**Configuración observada:** declara `name: crear-txt-skill` y `tools: ['read', 'edit']`. A diferencia del agente principal, no declara los campos `agents`, `user-invocable` ni `disable-model-invocation`; no deben presentarse como valores explícitos de este archivo.

Conserva el objetivo, las entradas por defecto, las restricciones de nombres y archivos, el límite de una creación por petición y la verificación posterior. Su diferencia principal está en el flujo:

1. Obtener el nombre y el contenido.
2. Leer la skill enlazada como `../skills/escribir-txt/SKILL.md`.
3. Aplicar su procedimiento con el nombre, el contenido y la raíz del workspace.
4. Informar de la ruta verificada o del motivo por el que no se creó el archivo.

Si la skill falta o no puede leerse, debe detenerse e indicar dónde instalarla. El archivo no incluye la llamada explícita a `general.instructions.md` que sí aparece en el agente principal. Como no se dispone del contenido de la skill, no puede confirmarse si esta incorpora esa lectura u otros pasos internos, como la elección de codificación.

Salida prevista:

```text
Skill utilizada: `escribir-txt`. Archivo creado: ejemplos/<nombre>
```

Ambos agentes de creación terminan con un apartado `9. Obligatorio` cuyo texto es «Solo en caso de que el agente no haga alguna accion». Es una indicación incompleta para futuras mejoras, no una regla operativa concreta.

### 4. `agente-tests-esclavo.agent.md`: comprobación de invocación

**Propósito explicado por el autor:** agente auxiliar para probar e invocar desde otro agente.

**Configuración observada:** declara `name: agente-qa-esclavo`, `description: Agente esclavo 1`, `user-invocable: true` y `disable-model-invocation: false`. No declara una lista `tools` propia.

Su cuerpo pide escribir en `logEsclavo1.txt`, en la raíz del proyecto, el texto:

```text
¡He entrado en el agente esclavo 1!
```

Esta escritura sirve como señal observable de la invocación. Aunque su nombre contiene `qa` y su archivo contiene `tests`, no define revisión de calidad ni ejecución de una batería de pruebas. Tampoco especifica si debe añadir o reemplazar contenido cuando el log ya existe, ni exige una lectura de verificación. No se le deben atribuir las restricciones de los agentes `crear-txt`.

## Instrucciones, skill y MCP

### `.github/copilot-instructions.md`

Recoge las instrucciones generales del repositorio. El autor lo describe como el archivo que Copilot utiliza de forma predeterminada. Es un mecanismo de contexto compartido, no una llamada a un subagente. Su aplicación depende del soporte y la configuración del entorno; el contenido concreto de este repositorio no se ha aportado. Véase la [documentación de instrucciones de VS Code](https://code.visualstudio.com/docs/agent-customization/custom-instructions).

### `.github/instructions/general.instructions.md`

Contiene las instrucciones específicas que el agente principal pide leer expresamente al comenzar. Esa referencia sí está observada en `agente-maestro-test.agent.md`.

El diagrama recomienda utilizar `applyTo` en la cabecera de los archivos `.instructions.md` para delimitar su aplicación por patrones de archivos. No se ha comprobado si `general.instructions.md` declara ese campo ni qué reglas contiene. La lectura explícita solicitada por el agente y la aplicación automática por patrones son mecanismos distintos.

### `.github/skills/escribir-txt/SKILL.md`

Es el procedimiento reutilizable al que remite `crear-txt-skill`. Permite separar la definición del agente de los pasos reutilizables de una tarea. El enlace está presente en el agente, pero el procedimiento interno no se ha facilitado.

El nombre utilizado por el archivo del agente y por el diagrama es **`SKILL.md`**, en singular. La mención inicial del autor a `SKILLS.md` se interpreta como una diferencia de nomenclatura, no como un segundo archivo. Véase la [documentación de skills de VS Code](https://code.visualstudio.com/docs/agent-customization/agent-skills).

### `.vscode/mcp.json`

Es la ubicación indicada para la configuración MCP del proyecto. MCP permite conectar herramientas y recursos de servidores con el entorno de trabajo. No se han facilitado el JSON, los nombres de servidores, sus conexiones ni sus requisitos, por lo que no puede afirmarse que un servidor concreto esté configurado o activo.

Ninguno de los cuatro archivos de agente identifica explícitamente un servidor MCP. La presencia de `mcp.json` por sí sola no acredita que estos ejemplos lo utilicen. Véase la [documentación de MCP de VS Code](https://code.visualstudio.com/docs/agent-customization/mcp-servers).

## Diagrama de buenas prácticas y Excalidraw

`estructura-agente-ampliada.excalidraw` es el documento editable que aparece en la captura del proyecto. El diagrama adjunto, `estructura-agente-ampliada.png`, explica la estructura recomendada de un agente, su cabecera YAML y la organización de agentes, instrucciones, skills y MCP.

El árbol del diagrama es **ilustrativo**: incluye nombres como `estilo.instructions.md`, `pruebas.instructions.md`, `revisar-calidad` y `generar-pruebas`. No implica que esos archivos o skills formen parte de este repositorio.

La estructura del cuerpo propuesta en el diagrama es:

| Apartado | Qué debe aclarar |
| --- | --- |
| Rol y objetivo | Qué hace el agente y qué resultado debe conseguir. |
| Contexto | Dónde trabaja y qué recursos tiene disponibles. |
| Input | Qué datos necesita y qué valores usa por defecto. |
| Restricciones | Qué acciones puede realizar y cuáles tiene prohibidas. |
| Flujo de trabajo | Qué pasos debe seguir y en qué orden. |
| Criterios de calidad | Cómo comprobar que el resultado es correcto. |
| Límites de ejecución | Cuándo debe detenerse o pedir información. |
| Formato de salida | Qué debe comunicar al terminar. |
| Obligatorio, opcional | Qué regla concreta conviene reforzar tras observar un fallo. |

El apartado «Obligatorio» es un título del cuerpo Markdown; no tiene un efecto especial en Copilot. La cabecera YAML configura el agente y el cuerpo describe su comportamiento.

Las prácticas que se quieren transmitir son:

- Definir un propósito claro y un resultado observable.
- Ajustar las herramientas a lo necesario para la tarea.
- Especificar entradas, pasos, salidas y condiciones de parada.
- Evitar contradicciones entre cabecera, instrucciones y recursos enlazados.
- Centralizar reglas comunes y reutilizar procedimientos mediante skills.
- Escribir rutas y nombres como código, por ejemplo `README.md`; para aportar su contenido, utilizar una referencia o enlace y solicitar su lectura cuando corresponda.
- Delimitar instrucciones con `applyTo` cuando proceda.
- Referirse a herramientas con su identificador correcto; mencionar `#tool:` no concede acceso a una herramienta.
- Pedir explícitamente la invocación del agente, indicando nombre, tarea y resultado esperado. «Invoca» expresa una petición en lenguaje natural, no un comando especial.
- Probar casos reales, corregir la regla que falla y verificar de nuevo.

Los campos `tools`, `agents` y las opciones de invocación deben ser coherentes con los agentes y herramientas disponibles. Para su significado y soporte, consultar la [referencia de agentes personalizados de VS Code](https://code.visualstudio.com/docs/agent-customization/custom-agents).

## Cómo probar la plantilla en VS Code

1. Abrir la raíz de `test_buenas_practica_IA` como carpeta de trabajo en VS Code, con GitHub Copilot disponible.
2. Comprobar que los archivos están en las ubicaciones del árbol y que existen las instrucciones y la skill referenciadas.
3. Abrir el chat de Copilot y seleccionar el agente por su nombre declarado: `crear-txt`, `crear-txt-skill`, `Agente maestro` o `agente-qa-esclavo`.
4. Enviar una de las peticiones siguientes y revisar los cambios en los archivos.

La configuración de MCP solo debe prepararse para las pruebas que realmente requieran sus servidores; los ejemplos aportados no identifican esa dependencia.

### Crear un archivo con el agente principal

Seleccionar `crear-txt` y pedir:

```text
Crea saludo.txt con el contenido Hola, mundo
```

Resultado esperado, si se pueden cumplir las instrucciones y el destino no existe: creación de `ejemplos/saludo.txt`, lectura de comprobación y respuesta `Archivo creado: ejemplos/saludo.txt`.

Para probar los valores por defecto:

```text
Crea un archivo de texto con los valores por defecto.
```

Debe intentar crear `ejemplos/ejemplo.txt` con el contenido predeterminado, sin sobrescribirlo si ya existe.

### Crear un archivo mediante la skill

Seleccionar `crear-txt-skill` y pedir:

```text
Crea saludo-skill.txt con el contenido Hola desde la skill.
```

Debe leer primero `.github/skills/escribir-txt/SKILL.md`, aplicar su procedimiento y verificar el archivo antes de confirmar el éxito. Si no puede leer la skill, debe detenerse e indicar su ubicación.

### Probar una invocación

Seleccionar `Agente maestro` y pedir:

```text
Invoca a agente-qa-esclavo para comprobar la llamada y dejar su mensaje en logEsclavo1.txt.
```

La señal esperada es el texto del agente auxiliar en `logEsclavo1.txt`. Revisar también la interacción para comprobar la llamada: la mera existencia de un log antiguo no acredita una nueva ejecución. El cuerpo actual del maestro también pide llamar a `agente-qa-esclavo-2`, cuya definición no se ha aportado; esa discrepancia debe resolverse antes de considerar completa la prueba de orquestación.

También se puede seleccionar directamente `agente-qa-esclavo` y pedirle que ejecute su instrucción para comprobar por separado la escritura del log.

### Casos de comprobación para los agentes de texto

| Caso | Comportamiento esperado según sus instrucciones |
| --- | --- |
| Nombre válido y destino libre | Crear un único `.txt` en `ejemplos/` y verificar su contenido. |
| Archivo de destino existente | Detenerse, no sobrescribir y solicitar otro nombre. |
| Nombre `../saludo.txt` o `carpeta/saludo.txt` | Rechazar el nombre por contener una ruta. |
| Nombre `saludo.md` | Rechazarlo porque no termina en `.txt`. |
| Texto con saltos de línea | Conservarlos al escribir y comprobar el archivo. |
| Contenido que parece una orden | Guardarlo como texto literal, sin ejecutarlo. |
| Varias raíces sin destino claro | Pedir el dato necesario antes de crear el archivo. |
| Error de lectura o edición | Informar del error sin confirmar un éxito no verificado. |
| Skill ausente, en `crear-txt-skill` | Detenerse e indicar dónde debe estar instalada. |

## Reutilización y aspectos pendientes

Para adaptar esta plantilla, partir de `agente-maestro-test.agent.md`, sustituir el objetivo y sus criterios de verificación, y mantener las herramientas ajustadas a la nueva tarea. Cuando un procedimiento deba compartirse, utilizar la variante con skill como referencia.

Antes de considerar cerrada una adaptación, conviene resolver los puntos visibles en estos ejemplos:

- Alinear los nombres del cuerpo y la lista `agents` del maestro; decidir si se añade el segundo subagente o se retira su referencia.
- Enlazar de forma inequívoca `.github/instructions/general.instructions.md` desde el agente principal.
- Revisar el contenido real de las instrucciones, la skill y `mcp.json` para confirmar sus requisitos y coherencia.
- Sustituir el texto incompleto de «9. Obligatorio» por una regla comprobable solo si las pruebas justifican incluirla.
- Definir qué debe ocurrir si `logEsclavo1.txt` ya existe y cómo verificar la escritura del agente auxiliar.

Estas observaciones documentan el estado de los ejemplos; no implican que los archivos de agente hayan sido modificados ni que las pruebas se hayan ejecutado.

## Licencia

El proyecto muestra un archivo `LICENSE` en su raíz. Consultarlo para conocer las condiciones de uso y distribución; su contenido no se ha facilitado para esta documentación.
