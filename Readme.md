Herramienta para la Creación de Equipos en el Aula

Este repositorio contiene una aplicación web de página única (Single Page Application) diseñada para ayudar a los docentes a distribuir al alumnado en grupos cooperativos (heterogéneos u homogéneos) de manera visual y sencilla.

📋 Descripción

La herramienta permite:

Cargar datos: Importar listas de alumnos desde Excel/CSV o pegar texto.

Configurar: Clasificar al alumnado según su rol (Ayuda, Autónomo, Necesita ayuda).

Distribuir: Generar un mapa visual del aula con la distribución automática.

Exportar: Generar un informe profesional en PDF.

🚀 Cómo subir este proyecto a GitHub

Sigue estos pasos para subir tu código y compartirlo con el mundo.

Paso 1: Preparar los archivos

Crea una carpeta nueva en tu ordenador (ej: creador-equipos-aula).

Guarda el código HTML que has generado dentro de esa carpeta.

Importante: Cambia el nombre del archivo de distribuidora_aula.html a index.html. Esto permitirá que GitHub Pages lo reconozca automáticamente como la página principal.

Paso 2: Crear el repositorio en GitHub

Entra en GitHub e inicia sesión.

Haz clic en el botón + (arriba a la derecha) y selecciona "New repository".

Escribe un nombre para el repositorio (ej: creador-equipos).

Asegúrate de que esté marcado como Public.

No marques ninguna otra casilla (ni README, ni .gitignore) por ahora.

Haz clic en "Create repository".

Paso 3: Subir los archivos (Opción Fácil: Web)

Si no quieres usar la terminal (consola de comandos), sigue estos pasos:

En la pantalla de tu nuevo repositorio, busca el enlace que dice "uploading an existing file".

Arrastra tu archivo index.html a la zona de carga.

En "Commit changes", escribe: "Subida inicial de la aplicación".

Haz clic en el botón verde "Commit changes".

Paso 3: Subir los archivos (Opción Profesional: Terminal/Git)

Si tienes Git instalado en tu ordenador:

cd ruta/a/tu/carpeta
git init
git add .
git commit -m "Versión inicial de la herramienta"
git branch -M main
git remote add origin [https://github.com/TU_USUARIO/NOMBRE_REPOSITORIO.git](https://github.com/TU_USUARIO/NOMBRE_REPOSITORIO.git)
git push -u origin main


(Recuerda cambiar TU_USUARIO y NOMBRE_REPOSITORIO por los tuyos).

🌐 Cómo publicar la web (GitHub Pages)

Una vez subido el archivo, sigue estos pasos para que la aplicación funcione en internet:

Ve a la pestaña Settings (Configuración) de tu repositorio.

En el menú de la izquierda, haz clic en Pages.

En la sección "Build and deployment", bajo Branch, selecciona main (o master) y asegúrate de que la carpeta sea /(root).

Haz clic en Save.

Espera unos minutos (1-2 min). Refresca la página y verás un mensaje arriba que dice: "Your site is live at...".

¡Ese enlace es el que puedes compartir con otros profesores para que usen la herramienta!

🛠️ Tecnologías usadas

HTML5

Tailwind CSS (CDN)

SheetJS (Librería para Excel)

Phosphor Icons

JavaScript Vanilla

Autor: El loco de la mochila
