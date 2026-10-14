---
name: escribir-txt
description: Crea y verifica un archivo .txt en ejemplos/ sin sobrescribir archivos existentes. Úsala cuando se pida guardar texto en esa carpeta.
---

# Escribir un archivo de texto

Recibe del agente un nombre .txt, el contenido literal y la raíz del workspace. El agente resuelve los valores por defecto antes de usar esta skill.

## Procedimiento
1. Comprueba que el nombre sea válido para el sistema y termine en .txt. Rechaza rutas, separadores y nombres vacíos.
2. Comprueba directamente la carpeta ejemplos/ y el destino ejemplos/<nombre>, incluyendo entradas ocultas o ignoradas. No deduzcas que está libre solo por una búsqueda sin resultados.
3. Si el destino existe, detente sin modificarlo y solicita otro nombre. Si no puedes comprobarlo, informa del impedimento.
4. Crea ejemplos/ si falta. Guarda un único archivo nuevo en UTF-8 con el contenido literal, sin añadir títulos, comillas ni explicaciones. Conserva los saltos de línea. No sobrescribas archivos.
5. Lee el archivo creado y compara su contenido. Comunica al agente la ruta y si la verificación fue correcta. Si algo falla, explica el error sin afirmar que terminaste.

Usa las herramientas de lectura y edición disponibles. No uses terminal ni red. Mantén el destino dentro del workspace; detente si ejemplos/ es un enlace que conduce fuera de él.

## Ejemplo
Entrada: nombre saludo.txt; contenido Hola, mundo.
Resultado: ejemplos/saludo.txt contiene únicamente Hola, mundo.
Si ya existe, no se modifica y se solicita otro nombre.
