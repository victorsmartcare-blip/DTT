# App DTT → Envio para Google Drive

Este projeto contém uma página `index.html` que envia o arquivo `DTT.pdf` para uma pasta do Google Drive via Google Apps Script.

## Passos rápidos

1. **Backend (Apps Script)**
   - Acesse https://script.google.com e crie um projeto.
   - Cole o código do backend abaixo em `Code.gs` e troque `FOLDER_ID` pelo ID da sua pasta no Drive.
   - Implemente como **Aplicativo da Web** com acesso **Qualquer pessoa com o link** e copie a URL `/exec`.

```javascript
function doPost(e) {
  try {
    const FOLDER_ID = 'COLE_AQUI_O_ID_DA_PASTA_DO_DRIVE'; // <-- ajuste

    if (!e || !e.postData || !e.postData.contents) {
      return _json({ ok: false, error: 'Sem corpo na requisição' }, 400);
    }

    const contentType = e.postData.type || 'application/json';
    let data;
    if (contentType.indexOf('application/json') !== -1) {
      data = JSON.parse(e.postData.contents);
    } else {
      const params = e.parameter || {};
      data = params;
    }

    if (data.hello) {
      return _json({ ok: true, message: 'Endpoint ativo' });
    }

    if (!data.base64) return _json({ ok: false, error: 'Campo base64 não enviado' }, 400);

    const bytes = Utilities.base64Decode(data.base64);
    const blob = Utilities.newBlob(bytes, 'application/pdf', data.filename || 'arquivo.pdf');

    const folder = DriveApp.getFolderById(FOLDER_ID);
    const file = folder.createFile(blob);

    const meta = data.meta || {};
    file.setDescription(JSON.stringify(meta));

    return _json({ ok: true, id: file.getId(), name: file.getName(), url: 'https://drive.google.com/file/d/' + file.getId() + '/view' });
  } catch (err) {
    return _json({ ok: false, error: String(err) }, 500);
  }
}

function doGet() { return ContentService.createTextOutput('OK'); }

function _json(obj) {
  const out = ContentService
    .createTextOutput(JSON.stringify(obj))
    .setMimeType(ContentService.MimeType.JSON);
  out.setHeader('Access-Control-Allow-Origin', '*');
  out.setHeader('Access-Control-Allow-Methods', 'POST, GET, OPTIONS');
  out.setHeader('Access-Control-Allow-Headers', 'Content-Type');
  return out;
}
```

2. **Frontend**
   - Mantenha `index.html` e `DTT.pdf` juntos.
   - Abra `index.html` e, no campo **Endpoint**, cole a URL `/exec` do Apps Script.
   - Clique **Testar endpoint** e depois **Enviar DTT.pdf para o Drive**.

## Publicar no GitHub Pages

1. Crie um repositório (ex.: `dtt-drive-app`) no GitHub.
2. Envie os arquivos (`index.html` e `DTT.pdf`).
3. Vá em **Settings → Pages** e selecione **Deploy from a branch** → **main** / **root**.
4. Acesse a URL do GitHub Pages e use o campo **Endpoint** para colar sua URL `/exec`.

### Via linha de comando
```bash
git init
git add .
git commit -m "DTT app: envio para Google Drive"
git branch -M main
git remote add origin https://github.com/SEU_USUARIO/dtt-drive-app.git
git push -u origin main
```
