# Lector QR - Control de Activos 📦

Aplicación web progresiva diseñada para el escaneo continuo de códigos QR de activos industriales y control de vigilancia. Los datos capturados se procesan en el navegador y se envían en tiempo real hacia hojas de cálculo de Google Sheets.

Este proyecto fue reestructurado con una **arquitectura desacoplada** para cumplir proactivamente con las políticas de seguridad de Google Workspace (efectivas a partir de octubre de 2026), las cuales bloquean el uso de permisos de hardware sensibles (como la cámara vía `getUserMedia()`) dentro de los iframes generados por Google Apps Script (`HtmlService`).

## 🏗️ Arquitectura del Sistema

- **Frontend (Cliente):** Alojado de forma estática y pública en GitHub Pages (`https://ti260.github.io/activoqr`). Maneja la interfaz de usuario, el acceso al hardware de la cámara mediante WebRTC, la decodificación de la imagen en el lienzo (`canvas`) utilizando la librería `jsQR` y la capa de autenticación.
- **Seguridad y Autenticación:** Implementada a través de Google Identity Services. Despliega un escudo de acceso que bloquea el escáner a usuarios externos. El sistema extrae el token JWT e inspecciona el dominio del correo electrónico en el cliente. La inclusión de múltiples dominios autorizados asegura la continuidad de la operación en piso de almacén durante la migración de cuentas y servidores programada para antes de septiembre de 2026.
- **Backend (API):** Un script nativo de Google Apps Script configurado como Aplicación Web y ejecutado mediante el método `doPost`. Actúa como un webhook que recibe cargas útiles (payloads) en formato JSON, procesa las reglas de enrutamiento y escribe en la base de datos.
- **Base de Datos:** Ecosistema de Google Sheets. El backend separa físicamente los registros en dos libros de cálculo distintos dependiendo del tipo de evento (Registro General de Activos vs. Bitácora de Vigilancia).

## 🚀 Guía de Despliegue y Configuración

### 1. Configuración del Backend (Google Apps Script)

1. Despliega el archivo `Código.gs` en tu proyecto de Apps Script. Este código debe contener únicamente las funciones lógicas y el método `doPost(e)`.
2. Dirígete a **Implementar > Nueva implementación**.
3. **Tipo:** Selecciona _Aplicación web_.
4. **Ejecutar como:** _Yo_ (La cuenta administrativa propietaria de los archivos).
5. **Quién tiene acceso:** _Cualquier persona_ (Este permiso es un requisito estricto y obligatorio en la consola de Google para evitar que el navegador del usuario bloquee la petición por políticas de CORS).
6. Copia la URL generada.

### 2. Configuración de Seguridad (Google Cloud Console)

1. Dentro del proyecto de Google Cloud asociado, navega a **APIs y servicios > Pantalla de consentimiento de OAuth**.
2. Configura el tipo de usuario como **Interno** (esto restringe el entorno a la organización y omite la necesidad de auditoría o verificación por parte de Google).
3. Dirígete a **Credenciales > Crear credenciales > ID de cliente de OAuth** (Tipo: Aplicación web).
4. En **Orígenes de JavaScript autorizados**, añade la raíz exacta del entorno de producción: `https://ti260.github.io` (Sin rutas secundarias ni barras diagonales finales).
5. Copia el **ID de Cliente** autogenerado.

### 3. Configuración del Frontend (GitHub Pages)

Abre el archivo `index.html` y actualiza las constantes de configuración ubicadas en el bloque principal del script:

- `client_id`: Inserta el ID de cliente de OAuth generado en Google Cloud.
- `APPS_SCRIPT_URL`: Inserta la URL de despliegue generada por Apps Script.
- `ALLOWED_DOMAINS`: Arreglo de cadenas que contiene los dominios corporativos válidos para iniciar sesión.

## 🗃️ Estructura de Datos (Formato QR)

El motor de decodificación requiere que la cadena de texto incrustada en el código QR siga un patrón de pares `Clave:Valor` delimitados por comas.

**Formato estricto requerido:**
`Activo:[String],id:[Num/String],CompanyName:[String],AssetName:[String]`

**Ejemplo de código válido:**

```text
Activo:Grupo Trebol,id:30,CompanyName:Bajio Green,AssetName:Paletizadora 180 mm
```
