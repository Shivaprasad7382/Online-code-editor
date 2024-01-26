import React, { useState } from 'react';
import axios from 'axios';

function App() {
  const [code, setCode] = useState('');
  const [language, setLanguage] = useState('python');
  const [output, setOutput] = useState('');

  return (
    <div>
      <textarea rows="10" cols="50" value={code} onChange={(e) => setCode(e.target.value)}></textarea>
      <select value={language} onChange={(e) => setLanguage(e.target.value)}>
        <option value="python">Python</option>
        <option value="javascript">JavaScript</option>
      </select>
      <button onClick={async () => setOutput((await axios.post('http://localhost:3001/execute', { code, language })).data.output)}>Run Code</button>
      <h2>Output:</h2>
      <pre>{output}</pre>
    </div>
  );
}

export default App;
//backend
const express = require('express');
const bodyParser = require('body-parser');
const { exec } = require('child_process');

const app = express();
const port = 3001;

app.use(bodyParser.json());

app.post('/execute', async (req, res) => {
  const { code, language } = req.body;
  if (!code || !language) return res.status(400).json({ error: 'Code and language are required' });
  exec(getExecutionCommand(language, code), (error, stdout, stderr) => {
    if (error) return res.status(500).json({ error: stderr });
    res.json({ output: stdout });
  });
});

function getExecutionCommand(language, code) {
  if (language === 'python') return `python -c "${code}"`;
  else if (language === 'javascript') return `node -e "${code}"`;
  else return '';
}

app.listen(port, () => console.log(`Server is running on http://localhost:${port}`));
