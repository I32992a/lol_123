[package.json](https://github.com/user-attachments/files/22649512/package.json)
{
  "name": "chatbot-backend",
  "version": "1.0.0",
  "main": "server.js",
  "type": "module",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "node-fetch": "^3.3.2",
    "cors": "^2.8.5"
  }
}
[backend.js](https://github.com/user-attachments/files/22649514/backend.js)
import express from "express";
import fetch from "node-fetch";
import cors from "cors";

const app = express();
app.use(cors());
app.use(express.json());

const HF_TOKEN = process.env.HF_TOKEN; // token stored in Render env vars

// Proxy route
app.post("/api/generate", async (req, res) => {
  try {
    const response = await fetch(
      "https://api-inference.huggingface.co/models/gpt2",
      {
        method: "POST",
        headers: {
          "Authorization": `Bearer ${HF_TOKEN}`,
          "Content-Type": "application/json",
        },
        body: JSON.stringify(req.body),
      }
    );
    const data = await response.json();
    res.json(data);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Serve static frontend
app.use(express.static("public"));

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`✅ Server running on port ${PORT}`));
[index.html](https://github.com/user-attachments/files/22649516/index.html)
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
