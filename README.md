[server.js](https://github.com/user-attachments/files/22649615/server.js)
const express = require('express');
const fetch = require('node-fetch');
const cors = require('cors');

const app = express();
app.use(cors());
app.use(express.json());

// Paste your Hugging Face API token here:
const API_TOKEN = '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';

// Model to use
const MODEL = 'EleutherAI/gpt-neo-125M';

app.post('/chat', async (req, res) => {
  const { message } = req.body;
  if (!message) return res.json({ reply: 'Please send a message.' });

  try {
    const response = await fetch(`https://api-inference.huggingface.co/models/${MODEL}`, {
      method: 'POST',
      headers: {
        Authorization: `Bearer ${API_TOKEN}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ inputs: message }),
    });
    const data = await response.json();

    if (data.error) {
      return res.json({ reply: data.error });
    }

    let botReply = data[0].generated_text;
    if (botReply.toLowerCase().startsWith(message.toLowerCase())) {
      botReply = botReply.slice(message.length).trim();
    }
    res.json({ reply: botReply });
  } catch (error) {
    res.json({ reply: 'Error contacting Hugging Face API.' });
  }
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
