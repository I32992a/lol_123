[index.html](https://github.com/user-attachments/files/22649556/index.html)
<!doctype html>
<html>
<head>
  <meta charset="utf-8" />
  <title>Offline Assignment AI</title>
</head>
<body>
  <h1>Upload assignment brief (offline)</h1>
  <form id="uploadForm">
    <input type="file" id="fileInput" name="file" required />
    <button type="submit">Upload</button>
  </form>

  <h2>Results</h2>
  <p><b>Paraphrased:</b></p>
  <pre id="original">—</pre>
  <p><b>Humanized:</b></p>
  <pre id="humanized">—</pre>
  <p><b>AI Detection:</b></p>
  <pre id="gptzero">—</pre>

  <script>
    const form = document.getElementById('uploadForm');
    form.addEventListener('submit', async (e) => {
      e.preventDefault();
      const fileInput = document.getElementById('fileInput');
      if (!fileInput.files.length) return alert('Choose a file');

      const fd = new FormData();
      fd.append('file', fileInput.files[0]);

      const url = 'http://localhost:3000/upload';

      document.getElementById('original').textContent = 'Processing...';
      document.getElementById('humanized').textContent = 'Processing...';
      document.getElementById('gptzero').textContent = 'Processing...';

      try {
        const resp = await fetch(url, { method: 'POST', body: fd });
        if (!resp.ok) throw new Error('Server error ' + resp.status);
        const data = await resp.json();
        document.getElementById('original').textContent = data.original || '—';
        document.getElementById('humanized').textContent = data.humanized || '—';
        document.getElementById('gptzero').textContent = JSON.stringify(data.gptZeroResult, null, 2) || '—';
      } catch (err) {
        document.getElementById('original').textContent = 'Error: ' + err.message;
        document.getElementById('humanized').textContent = '';
        document.getElementById('gptzero').textContent = '';
      }
    });
  </script>
</body>
</html>
