# Rúbrica de clases - Nutrición

Página estática con envío a Google Apps Script + Google Sheets y PDF de la evaluación.

## Qué hace
- Muestra la nota en vivo mientras seleccionas los criterios.
- Al guardar: descarga un PDF local y además envía los datos + el PDF en base64 al Web App de Apps Script para guardarlo en Google Drive.

## Configurar Google Apps Script
Pegue este código (ajuste IDs) en Code.gs y despliegue como Web App (Acceso: Cualquiera con el enlace):

```
const SHEET_ID = 'PONGA_AQUI_EL_ID_DE_SU_SHEET';
const DRIVE_FOLDER_ID = 'PONGA_AQUI_EL_ID_DE_LA_CARPETA_DRIVE';

function doPost(e) {
  try {
    const payload = JSON.parse(e && (e.postData?.contents || e.parameter?.payload || '{}'));

    // 1) Guardar fila en Sheets
    const ss = SpreadsheetApp.openById(SHEET_ID);
    const sh = ss.getSheets()[0];
    const row = [
      new Date(),
      payload.resident || '',
      payload.evaluator || '',
      payload.evalDate || '',
      payload.classTopic || '',
      ...(payload.selections || []),
      payload.totalScore || '',
      payload.percentage || '',
      payload.comments || ''
    ];
    sh.appendRow(row);

    // 2) Guardar PDF en Drive si viene base64
    if (payload.pdfBase64 && payload.pdfFilename) {
      const bytes = Utilities.base64Decode(payload.pdfBase64);
      const blob = Utilities.newBlob(bytes, 'application/pdf', payload.pdfFilename);
      const folder = DriveApp.getFolderById(DRIVE_FOLDER_ID);
      const file = folder.createFile(blob);
      // Enlace al PDF al final de la fila (columna extra)
      sh.getRange(sh.getLastRow(), sh.getLastColumn() + 1).setValue(file.getUrl());
    }

    return ContentService.createTextOutput(JSON.stringify({ ok: true }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch (err) {
    return ContentService.createTextOutput(JSON.stringify({ ok: false, error: String(err) }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

Después copie la URL del Web App en `index.html` (const `SCRIPT_URL`).

## Probar
- Seleccione los 5 criterios y observe la nota en vivo.
- Pulse Guardar: se descargará el PDF y se enviará a Apps Script.
- Verifique la fila en Sheets y el PDF en la carpeta de Drive.
