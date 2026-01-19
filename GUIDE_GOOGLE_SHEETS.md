# Guía de Integración con Google Sheets (Google Apps Script)

Este proyecto está diseñado para funcionar como un sitio estático (en GitHub Pages, Vercel, etc.) y usar Google Sheets como base de datos gratuita.

## Pasos para configurar el Backend:

1. **Crear la Hoja de Cálculo**
   - Entrá a [Google Sheets](https://sheets.new) y creá una nueva hoja.
   - En la primera fila, creá los encabezados exactos:
     `id`, `createdAt`, `clientName`, `clientPhone`, `clientEmail`, `serviceId`, `date`, `time`, `comments`, `status`

2. **Abrir Google Apps Script**
   - En la hoja, andá a `Extensiones` > `Apps Script`.

3. **Pegar el Código**
   - Borrá el código que aparece y pegá el siguiente script:

\`\`\`javascript
const SHEET_NAME = "Hoja 1"; // Asegurate que coincida con el nombre de la pestaña

function doPost(e) {
  const lock = LockService.getScriptLock();
  lock.tryLock(10000);

  try {
    const doc = SpreadsheetApp.getActiveSpreadsheet();
    const sheet = doc.getSheetByName(SHEET_NAME);
    
    const data = JSON.parse(e.postData.contents);
    
    const newRow = [
      Utilities.getUuid(), // ID único
      new Date(),          // Fecha creación
      data.clientName,
      data.clientPhone,
      data.clientEmail,
      data.serviceId,
      data.date,
      data.time,
      data.comments,
      'pending'            // Estado inicial
    ];

    sheet.appendRow(newRow);

    return ContentService.createTextOutput(JSON.stringify({ result: "success", row: newRow }))
      .setMimeType(ContentService.MimeType.JSON);

  } catch (e) {
    return ContentService.createTextOutput(JSON.stringify({ result: "error", error: e }))
      .setMimeType(ContentService.MimeType.JSON);
  } finally {
    lock.releaseLock();
  }
}

function doGet(e) {
   // Esto sirve para leer los turnos desde el Admin Panel
   const doc = SpreadsheetApp.getActiveSpreadsheet();
   const sheet = doc.getSheetByName(SHEET_NAME);
   const rows = sheet.getDataRange().getValues();
   const headers = rows.shift(); // Sacar encabezados
   
   const data = rows.map(row => {
     let obj = {};
     headers.forEach((h, i) => obj[h] = row[i]);
     return obj;
   });
   
   return ContentService.createTextOutput(JSON.stringify(data))
    .setMimeType(ContentService.MimeType.JSON);
}
\`\`\`

4. **Implementar como Web App**
   - Hacé clic en el botón azul "Implementar" > "Nueva implementación".
   - **Tipo:** Aplicación web.
   - **Descripción:** "API Turnos".
   - **Ejecutar como:** "Yo" (tu email).
   - **Quién tiene acceso:** **"Cualquier persona"** (Importante para que el frontend pueda enviar datos).
   - Copiá la URL que te da (termina en `/exec`).

5. **Conectar con el Frontend**
   - En el archivo `client/src/lib/googleSheets.ts`, reemplazá la lógica "Mock" por `fetch('TU_URL_DE_APPS_SCRIPT', ...)`
   
   Ejemplo de implementación real en `googleSheets.ts`:
   
   \`\`\`typescript
   const API_URL = "TU_URL_DE_GOOGLE_SCRIPT";
   
   // ... en createAppointment
   await fetch(API_URL, {
     method: "POST",
     body: JSON.stringify(data),
     mode: "no-cors" // Necesario para Google Scripts a veces, o manejar CORS headers en el script
   });
   \`\`\`

## Notas de Seguridad
- Esta configuración es básica. El PIN del admin panel en este prototipo está en el frontend (`1234`).
- Para mayor seguridad, considerá usar Firebase o Supabase, que también tienen planes gratuitos generosos y son más seguros que Google Sheets para datos sensibles.
