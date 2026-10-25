# Guía Técnica de Despliegue, Resolución de Errores y Endurecimiento de Seguridad - PokéDex

Este documento detalla de forma exhaustiva el proceso técnico completo para desplegar la aplicación PokéDex en la nube pública de Microsoft Azure, incluyendo la resolución de incidencias en rutas de compilación y la implementación de cabeceras HTTP de seguridad para alcanzar la máxima calificación en **Security Headers**.

---

## 1. Despliegue de la Aplicación en Azure Static Web Apps

1. **Creación del Recurso en el Portal de Azure:**
   - En el panel de Azure, hacer clic en **Crear un recurso** y buscar el servicio **Aplicación web estática** (*Static Web Apps*).
   - Configurar los parámetros básicos solicitados por el asistente de creación:
     - **Suscripción:** `Azure for Students Starter`.
     - **Grupo de recursos:** Crear un grupo nuevo denominado `rg-pokedex-prod` para aislar los recursos del proyecto.
     - **Nombre de la aplicación web estática:** `swa-pokedex-portal-prod`.
     - **Plan de hospedaje / SKU:** Seleccionar la opción **Gratis (Free)** para proyectos personales y de prueba.

2. **Gestión y Resolución de Restricción de Región:**
   - **Incidencia:** Al intentar realizar la validación y creación final del recurso seleccionando inicialmente la región `centralus`, la plataforma arrojó un error de validación de directiva de suscripción: `The template deployment failed because of policy violation... (código: RequestDisallowedByAzure)`. Esto ocurre porque las políticas de la suscripción estudiantil restringen ciertas regiones por defecto.
   - **Solución:** Se modificó la región de hospedaje en el asistente de creación seleccionando **`westus2`**, lo cual permitió superar la restricción de directiva y avanzar exitosamente en la creación del recurso.

3. **Configuración de Integración con GitHub:**
   - En la sección de detalles de implementación, seleccionar **GitHub** como origen de código.
   - Vincular la cuenta de usuario de GitHub (`ocastillocc`), seleccionando el repositorio `pokedex` y la rama principal `main`.
   - Configurar los detalles de compilación detectados automáticamente para Angular:
     - **Ubicación de la aplicación:** `/`.
     - **Ubicación de la API:** *(Dejar en blanco)*.
     - **Ubicación de salida (Output location):** `dist/pokedex-angular`.

---

## 2. Corrección de Rutas de Imágenes (Adaptación de GitHub Pages a Azure)

- **Problema Detectado:** Una vez finalizado el despliegue inicial y accesible la URL pública generada por Azure (`https://mango-tree-0bfea311e.1.azurestaticapps.net`), se visualizó la interfaz correctamente pero con la ausencia total de las imágenes de los Pokémon. Al revisar el código fuente, se identificó que el proyecto original estaba configurado específicamente para rutas estáticas de GitHub Pages (`imagesPath: '/pokedex-angular/assets/images'`).
- **Solución Aplicada:** 
  - Se abrió el archivo de entorno de producción en VS Code: `src/environments/environment.prod.ts`.
  - Se modificó la línea de la propiedad `imagesPath` cambiándola a **`'/assets/images'`**.
  - Tras guardar los cambios y realizar el envío (*push*) al repositorio, el flujo de integración continua recompiló la aplicación y las imágenes comenzaron a cargarse de forma óptima en la plataforma de Azure.

---

## 3. Endurecimiento de Seguridad (Security Headers - Calificación A+)

Para cumplir con los estrictos requisitos de seguridad exigidos por Pueblo Paleta Inc. frente a ciberataques, la aplicación fue sometida a escaneos en la herramienta [Security Headers](https://securityheaders.com/).

1. **Diagnóstico Inicial:**
   - En el primer análisis sobre la URL pública de Azure, la aplicación obtuvo una calificación de **Categoría C**, debido a la ausencia de cabeceras de protección críticas como `Content-Security-Policy` (CSP), `X-Frame-Options` y `Permissions-Policy`.

2. **Implementación de Cabeceras Globales de Seguridad:**
   - Se creó un nuevo archivo de configuración en la raíz del proyecto Angular llamado **`src/staticwebapp.config.json`**.
   - Dentro de este archivo se definieron las políticas de seguridad estrictas bajo el objeto `globalHeaders`, especificando restricciones para scripts, estilos y fuentes permitidas, así como políticas de permisos para componentes de hardware y sensores del navegador.

3. **Modificación del Archivo de Compilación (`angular.json`):**
   - Para garantizar que Azure procese correctamente el archivo de configuración, se realizaron dos cambios clave en `angular.json`:
     - En la sección `architect.build.options.assets`, se añadió la ruta `"src/staticwebapp.config.json"` para asegurar que el compilador lo incluya dentro de la carpeta de distribución (`dist/`).
     - En la sección `architect.build.configurations.production`, se añadió el bloque de optimización configurando **`"inlineCritical": false`** dentro de las opciones de estilos. Esto evita que Angular inyecte scripts de carga diferida en línea dentro del HTML, lo cual bloquearía la ejecución bajo las estrictas políticas de la cabecera CSP.
   - Adicionalmente, se eliminó el script heredado de compatibilidad para SPA de GitHub Pages ubicado en `src/index.html` para evitar conflictos de ejecución en la consola del navegador.

4. **Resultado Final del Escaneo:**
   - Tras aplicar las modificaciones y actualizar el repositorio, se repitió el escaneo de seguridad en `securityheaders.com`, obteniendo de forma exitosa la calificación máxima **A+**, cumpliendo con los estándares requeridos para obtener el puntaje completo en la rúbrica de evaluación.

---

## 4. Comandos de Git Utilizados en el Proceso

Para gestionar el versionamiento y la publicación automatizada de los cambios hacia el repositorio remoto, se ejecutaron los siguientes comandos en la terminal de Git:

```bash
# Clonar el repositorio base del proyecto
git clone [https://github.com/ocastillocc/pokedex.git](https://github.com/ocastillocc/pokedex.git)

# Verificar el estado de los archivos modificados
git status

# Añadir todos los cambios locales al área de intercambio (staging)
git add .

# Registrar los cambios realizados con un mensaje descriptivo del commit
git commit -m "Configuración de Azure Static Web Apps, corrección de rutas de imágenes y cabeceras de seguridad A+"

# Enviar los cambios al repositorio remoto en la rama principal
git push origin main