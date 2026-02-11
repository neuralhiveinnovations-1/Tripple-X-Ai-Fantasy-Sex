triple-x-ai-fantasy-sex/
├── frontend/
│   ├── package.json
│   ├── src/
│   │   ├── App.js
│   │   ├── NeonParticles.js
│   │   ├── TemplatesPanel.js
│   │   └── index.js          (optional basic)
│   └── .env.example
├── backend/
│   ├── package.json
│   ├── server.js
│   └── .env.example
└── README.md
{
  "name": "triple-x-frontend",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "@react-three/drei": "^9.88.0",
    "@react-three/fiber": "^8.15.0",
    "@stripe/react-stripe-js": "^2.4.0",
    "@stripe/stripe-js": "^4.0.0",
    "axios": "^1.7.0",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-speech-recognition": "^3.10.0",
    "three": "^0.165.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build"
  },
  "browserslist": {
    "production": [">0.2%", "not dead", "not op_mini all"],
    "development": ["last 1 chrome version", "last 1 firefox version", "last 1 safari version"]
  }
}REACT_APP_STRIPE_PUB_KEY=pk_test_...
REACT_APP_BACKEND_URL=http://localhost:5000
import React, { useState, useEffect } from 'react';
import { Canvas } from '@react-three/fiber';
import { OrbitControls, Html } from '@react-three/drei';
import SpeechRecognition, { useSpeechRecognition } from 'react-speech-recognition';
import axios from 'axios';
import { loadStripe } from '@stripe/stripe-js';
import { Elements, CardElement, useStripe, useElements } from '@stripe/react-stripe-js';
import NeonParticles from './NeonParticles';
import TemplatesPanel from './TemplatesPanel';

const stripePromise = loadStripe(process.env.REACT_APP_STRIPE_PUB_KEY || 'pk_test_dummy');

function App() {
  const [user, setUser] = useState(null);
  const [isOwner, setIsOwner] = useState(false);
  const [subscribed, setSubscribed] = useState(false);
  const [prompt, setPrompt] = useState('');
  const [mode, setMode] = useState('both');
  const [model, setModel] = useState(null);
  const { transcript, resetTranscript } = useSpeechRecognition();

  useEffect(() => {
    // Replace with real auth later (JWT / session)
    const logged = localStorage.getItem('username') || 'guest';
    setUser(logged);
    setIsOwner(logged === 'Vaneeya');
    setSubscribed(logged === 'Vaneeya');
  }, []);

  const generateModel = async () => {
    if (!isOwner && !subscribed) return alert('Subscribe to unlock super sexy AI');
    try {
      const res = await axios.post(`${process.env.REACT_APP_BACKEND_URL}/api/triple-x/generate`, {
        prompt,
        mode
      }, { headers: { username: user } });
      setModel(res.data.modelUrl);
    } catch (err) {
      console.error(err);
    }
  };

  const PaymentForm = () => {
    const stripe = useStripe();
    const elements = useElements();

    const handlePay = async () => {
      if (!stripe || !elements) return;
      const { token, error } = await stripe.createToken(elements.getElement(CardElement));
      if (error) return console.error(error);
      try {
        const res = await axios.post(`${process.env.REACT_APP_BACKEND_URL}/api/triple-x/subscribe`, { token });
        if (res.data.success) setSubscribed(true);
      } catch (err) {}
    };

    return (
      <div style={{ margin: '20px 0' }}>
        <CardElement options={{ style: { base: { color: '#fff' } } }} />
        <button onClick={handlePay} style={{ marginTop: '10px' }}>Subscribe $19.99/mo</button>
      </div>
    );
  };

  return (
    <Elements stripe={stripePromise}>
      <div style={{ height: '100vh', display: 'flex', background: '#000', color: '#fff', fontFamily: 'Arial' }}>
        <div style={{ width: '30%', padding: '20px', overflowY: 'auto' }}>
          <h1 style={{ color: '#ff00ff', textShadow: '0 0 15px #ff00ff' }}>Triple X AI Fantasy Sex</h1>
          {isOwner ? <p style={{ color: '#00ffff' }}>Owner: Unlimited Access</p> : !subscribed && <PaymentForm />}
          
          <select value={mode} onChange={e => setMode(e.target.value)} style={{ margin: '10px 0', width: '100%' }}>
            <option value="real">Ultra-Realistic Sexy</option>
            <option value="stylized">Hyper-Stylized Hentai</option>
            <option value="both">Both / Hybrid</option>
          </select>

          <textarea
            value={prompt}
            onChange={e => setPrompt(e.target.value)}
            placeholder="Describe your fantasy... (voice or text)"
            style={{ width: '100%', height: '100px', marginBottom: '10px' }}
          />

          <div style={{ margin: '10px 0' }}>
            <button onClick={generateModel}>Generate</button>
            <button onClick={() => SpeechRecognition.startListening({ continuous: true })}>Voice Start</button>
            <button onClick={() => {
              SpeechRecognition.stopListening();
              setPrompt(transcript);
              generateModel();
              resetTranscript();
            }}>Voice Stop & Generate</button>
          </div>

          <TemplatesPanel />
        </div>

        <Canvas style={{ width: '70%' }}>
          <ambientLight intensity={0.6} />
          <pointLight position={[10, 10, 10]} intensity={1.5} color="#ff00ff" />
          <NeonParticles count={1000} />
          {model ? (
            <mesh>
              <Html center>Model Loaded: {model}</Html>
              {/* Replace with <GLTFLoader url={model} /> when real GLB */}
            </mesh>
          ) : (
            <Html center>Preview your super sexy creation here</Html>
          )}
          <OrbitControls />
        </Canvas>
      </div>
    </Elements>
  );
}

export default App;
import { useRef, useMemo } from 'react';
import { useFrame } from '@react-three/fiber';
import * as THREE from 'three';

export default function NeonParticles({ count = 1000 }) {
  const points = useRef();

  const positions = useMemo(() => {
    const pos = new Float32Array(count * 3);
    for (let i = 0; i < count * 3; i += 3) {
      pos[i] = (Math.random() - 0.5) * 50;
      pos[i + 1] = (Math.random() - 0.5) * 30;
      pos[i + 2] = (Math.random() - 0.5) * 50;
    }
    return pos;
  }, [count]);

  const colors = useMemo(() => {
    const col = new Float32Array(count * 3);
    const palette = ['#ff00ff', '#00ffff', '#ff69b4', '#00ff9f'];
    for (let i = 0; i < count; i++) {
      const c = new THREE.Color(palette[Math.floor(Math.random() * palette.length)]);
      col[i * 3] = c.r; col[i * 3 + 1] = c.g; col[i * 3 + 2] = c.b;
    }
    return col;
  }, [count]);

  useFrame(({ clock }) => {
    if (!points.current) return;
    const t = clock.getElapsedTime() * 0.15;
    const pos = points.current.geometry.attributes.position.array;
    for (let i = 1; i < count * 3; i += 3) {
      pos[i] += Math.sin(t + pos[i - 1] * 0.1) * 0.003;
      if (pos[i] > 15) pos[i] = -15;
    }
    points.current.geometry.attributes.position.needsUpdate = true;
  });

  return (
    <points ref={points}>
      <bufferGeometry>
        <bufferAttribute attach="attributes-position" count={count} array={positions} itemSize={3} />
        <bufferAttribute attach="attributes-color" count={count} array={colors} itemSize={3} />
      </bufferGeometry>
      <pointsMaterial size={0.15} vertexColors transparent opacity={0.8} blending={THREE.AdditiveBlending} depthWrite={false} />
    </points>
  );
}import React, { useState } from 'react';

export default function TemplatesPanel() {
  const [selected, setSelected] = useState(null);

  const templates = [
    { id: 1, name: 'Beach Bombshell', img: 'https://via.placeholder.com/120x160/ff00ff/000?text=Beach' },
    { id: 2, name: 'Goth Seductress', img: 'https://via.placeholder.com/120x160/00ffff/000?text=Goth' },
    { id: 3, name: 'Alpha Male', img: 'https://via.placeholder.com/120x160/ff69b4/000?text=Alpha' },
    { id: 4, name: 'Girl-on-Girl', img: 'https://via.placeholder.com/120x160/00ff9f/000?text=Duo' },
  ];

  return (
    <div style={{ marginTop: '30px' }}>
      <h2 style={{ color: '#ff00ff', textShadow: '0 0 10px #ff00ff' }}>Neon Template Vault</h2>
      <div style={{ display: 'grid', gridTemplateColumns: 'repeat(auto-fill, minmax(120px, 1fr))', gap: '15px' }}>
        {templates.map(t => (
          <div
            key={t.id}
            onClick={() => setSelected(t.id)}
            style={{
              border: selected === t.id ? '3px solid #00ffff' : '2px solid #ff00ff',
              borderRadius: '8px',
              overflow: 'hidden',
              boxShadow: selected === t.id ? '0 0 25px #00ffff' : '0 0 15px rgba(255,0,255,0.5)',
              cursor: 'pointer',
              transition: 'all 0.3s'
            }}
          >
            <img src={t.img} alt={t.name} style={{ width: '100%', height: '160px', objectFit: 'cover' }} />
            <div style={{ textAlign: 'center', padding: '8px', background: 'rgba(0,0,0,0.7)', color: '#00ffff' }}>
              {t.name}
            </div>
          </div>
        ))}
      </div>
    </div>
  );
}{
  "name": "triple-x-backend",
  "version": "1.0.0",
  "main": "server.js",
  "dependencies": {
    "express": "^4.19.2",
    "cors": "^2.8.5",
    "body-parser": "^1.20.2",
    "stripe": "^16.2.0",
    "mongoose": "^8.3.1",
    "replicate": "^0.25.2"
  },
  "scripts": {
    "start": "node server.js"
  }
}STRIPE_SECRET_KEY=sk_test_...
REPLICATE_API_TOKEN=r8_...
MONGO_URI=mongodb+srv://...
PORT=5000
OWNER_USERNAME=Vaneeya
const express = require('express');
const cors = require('cors');
const bodyParser = require('body-parser');
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);
const mongoose = require('mongoose');
const Replicate = require('replicate');

const app = express();
app.use(cors());
app.use(bodyParser.json());

mongoose.connect(process.env.MONGO_URI || 'mongodb://localhost/triplex', { useNewUrlParser: true });

const replicate = new Replicate({ auth: process.env.REPLICATE_API_TOKEN });

app.post('/api/triple-x/generate', async (req, res) => {
  const { prompt, mode } = req.body;
  const username = req.headers.username || 'guest';

  if (username !== process.env.OWNER_USERNAME) {
    // In real: check subscription from DB
    // For MVP: block non-owner
    return res.status(403).json({ error: 'Subscribe required for super sexy AI' });
  }

  let base = mode === 'real' 
    ? 'photorealistic ultra sexy woman, erotic body, detailed skin, seductive, NSFW, 8k'
    : mode === 'stylized'
    ? 'anime hentai style, exaggerated sexy curves, glossy, sparkling, over-the-top erotic, vibrant'
    : '(photorealistic:0.5), (anime hentai:0.5), blended sexy fantasy';

  try {
    const output = await replicate.run(
      "stability-ai/sdxl:39ed52f2a78e934b3ba6e2a89f5b1c712de7dfea535525255b1aa35c556aad08",
      { input: { prompt: `${base}, ${prompt}` } }
    );
    res.json({ modelUrl: output[0] }); // image for now – upgrade to 3D later
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

app.post('/api/triple-x/subscribe', async (req, res) => {
  const { token } = req.body;
  try {
    await stripe.charges.create({
      amount: 1999,
      currency: 'usd',
      source: token.id,
      description: 'Triple X Premium'
    });
    res.json({ success: true });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Backend on ${PORT}`));
const express = require('express');
const cors = require('cors');
const bodyParser = require('body-parser');
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY);
const mongoose = require('mongoose');
const Replicate = require('replicate');

const app = express();
app.use(cors());
app.use(bodyParser.json());

mongoose.connect(process.env.MONGO_URI || 'mongodb://localhost/triplex', { useNewUrlParser: true });

const replicate = new Replicate({ auth: process.env.REPLICATE_API_TOKEN });

app.post('/api/triple-x/generate', async (req, res) => {
  const { prompt, mode } = req.body;
  const username = req.headers.username || 'guest';

  if (username !== process.env.OWNER_USERNAME) {
    // In real: check subscription from DB
    // For MVP: block non-owner
    return res.status(403).json({ error: 'Subscribe required for super sexy AI' });
  }

  let base = mode === 'real' 
    ? 'photorealistic ultra sexy woman, erotic body, detailed skin, seductive, NSFW, 8k'
    : mode === 'stylized'
    ? 'anime hentai style, exaggerated sexy curves, glossy, sparkling, over-the-top erotic, vibrant'
    : '(photorealistic:0.5), (anime hentai:0.5), blended sexy fantasy';

  try {
    const output = await replicate.run(
      "stability-ai/sdxl:39ed52f2a78e934b3ba6e2a89f5b1c712de7dfea535525255b1aa35c556aad08",
      { input: { prompt: `${base}, ${prompt}` } }
    );
    res.json({ modelUrl: output[0] }); // image for now – upgrade to 3D later
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

app.post('/api/triple-x/subscribe', async (req, res) => {
  const { token } = req.body;
  try {
    await stripe.charges.create({
      amount: 1999,
      currency: 'usd',
      source: token.id,
      description: 'Triple X Premium'
    });
    res.json({ success: true });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Backend on ${PORT}`));
# Triple X AI Fantasy Sex

Owner (Vaneeya): full access  
Others: paywall via Stripe

## Deploy
1. Add env vars in Vercel dashboard
2. Deploy frontend (React) and backend (Node) separately
3. Connect frontend to backend URL
