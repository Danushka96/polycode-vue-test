<script setup>
import { ref } from 'vue';

const name = ref('danushka');
const loading = ref(false);
const results = ref([]);

const endpoints = [
  {
    id: 'get-hello',
    method: 'GET',
    url: (n) => `https://ep96ageg0.api.c190.us-east-1.537413656254.polycode.app/hello?name=${n}`,
    label: 'GET /hello',
  },
  {
    id: 'get-hello-cache',
    method: 'GET',
    url: (n) => `https://ep96ageg0.api.c190.us-east-1.537413656254.polycode.app/hello-cache?name=${n}`,
    label: 'GET /hello-cache',
  },
  {
    id: 'post-hello-cache',
    method: 'POST',
    url: (n) => `https://ep96ageg0.api.c190.us-east-1.537413656254.polycode.app/hello-cache`,
    body: (n) => ({ name: n }),
    label: 'POST /hello-cache',
  },
];

const checkLatency = async () => {
  loading.value = true;
  results.value = [];

  // Initialize results placeholders
  results.value = endpoints.map(ep => ({
    ...ep,
    status: 'pending',
    latency: null,
    xCache: null,
    response: null,
    error: null
  }));

  const promises = endpoints.map(async (endpoint, index) => {
    const startTime = performance.now();
    try {
      const options = {
        method: endpoint.method,
        headers: {
          'Content-Type': 'application/json',
        },
      };

      if (endpoint.body) {
        options.body = JSON.stringify(endpoint.body(name.value));
      }

      const url = typeof endpoint.url === 'function' ? endpoint.url(name.value) : endpoint.url;
      
      const res = await fetch(url, options);
      const endTime = performance.now();
      
      const data = await res.json();
      const xCache = res.headers.get('x-cache') || 'N/A';
      
      results.value[index] = {
        ...results.value[index],
        status: 'success',
        latency: (endTime - startTime).toFixed(2),
        xCache,
        response: data,
      };
    } catch (err) {
      const endTime = performance.now();
      results.value[index] = {
        ...results.value[index],
        status: 'error',
        latency: (endTime - startTime).toFixed(2),
        error: err.message,
      };
    }
  });

  await Promise.all(promises);
  loading.value = false;
};
</script>

<template>
  <div class="monitor-container">
    <div class="glass-panel">
      <h1 class="title">API Latency Monitor</h1>
      
      <div class="controls">
        <div class="input-group">
          <label for="name-input">Name Variable</label>
          <input 
            id="name-input" 
            v-model="name" 
            type="text" 
            placeholder="Enter name..."
            @keyup.enter="checkLatency"
          />
        </div>
        <button @click="checkLatency" :disabled="loading" class="run-btn">
          <span v-if="!loading">Run Monitor</span>
          <span v-else class="loader"></span>
        </button>
      </div>

      <div class="results-grid">
        <div 
          v-for="result in results" 
          :key="result.id" 
          class="result-card"
          :class="result.status"
        >
          <div class="card-header">
            <span class="method" :class="result.method.toLowerCase()">{{ result.method }}</span>
            <span class="url-label">{{ result.label }}</span>
          </div>

          <div class="metrics">
            <div class="metric">
              <span class="metric-label">Latency</span>
              <span class="metric-value">{{ result.latency ? `${result.latency}ms` : '--' }}</span>
            </div>
            <div class="metric">
              <span class="metric-label">X-Cache</span>
              <span class="metric-value-tag" :class="result.xCache?.toLowerCase().includes('hit') ? 'hit' : 'miss'">
                {{ result.xCache || '--' }}
              </span>
            </div>
          </div>

          <div class="response-preview">
            <span class="preview-label">Response Body</span>
            <pre v-if="result.response">{{ JSON.stringify(result.response, null, 2) }}</pre>
            <div v-else-if="result.error" class="error-msg">{{ result.error }}</div>
            <div v-else class="placeholder">Waiting...</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Reset & Base */
.monitor-container {
  min-height: 100vh;
  width: 100%;
  padding: 2rem;
  background: radial-gradient(circle at top left, #1a2a6c, #b21f1f, #fdbb2d);
  background: linear-gradient(135deg, #0f172a 0%, #1e293b 100%);
  color: #fff;
  font-family: 'Inter', sans-serif;
  display: flex;
  justify-content: center;
  align-items: flex-start;
}

.glass-panel {
  background: rgba(255, 255, 255, 0.05);
  backdrop-filter: blur(16px);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 24px;
  padding: 3rem;
  width: 100%;
  max-width: 1200px;
  box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.5);
}

.title {
  font-size: 2.5rem;
  font-weight: 700;
  margin-bottom: 2rem;
  background: linear-gradient(to right, #60a5fa, #c084fc);
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
  text-align: center;
}

/* Controls */
.controls {
  display: flex;
  gap: 1.5rem;
  justify-content: center;
  align-items: flex-end;
  margin-bottom: 3rem;
}

.input-group {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.input-group label {
  font-size: 0.875rem;
  color: #94a3b8;
  font-weight: 500;
}

input {
  background: rgba(15, 23, 42, 0.6);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 12px;
  padding: 0.75rem 1rem;
  color: #fff;
  font-size: 1rem;
  width: 300px;
  transition: all 0.3s ease;
}

input:focus {
  outline: none;
  border-color: #60a5fa;
  box-shadow: 0 0 0 4px rgba(96, 165, 250, 0.1);
}

.run-btn {
  background: linear-gradient(135deg, #3b82f6 0%, #2563eb 100%);
  border: none;
  border-radius: 12px;
  padding: 0.75rem 2rem;
  color: #fff;
  font-weight: 600;
  font-size: 1rem;
  cursor: pointer;
  transition: all 0.3s ease;
  height: 46px; /* Match input height */
  display: flex;
  align-items: center;
  justify-content: center;
  min-width: 140px;
}

.run-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 10px 15px -3px rgba(37, 99, 235, 0.4);
}

.run-btn:disabled {
  opacity: 0.7;
  cursor: not-allowed;
  transform: none;
}

/* Results Grid */
.results-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 2rem;
}

.result-card {
  background: rgba(30, 41, 59, 0.6);
  border: 1px solid rgba(255, 255, 255, 0.05);
  border-radius: 16px;
  padding: 1.5rem;
  transition: all 0.3s ease;
  display: flex;
  flex-direction: column;
}

.result-card:hover {
  background: rgba(30, 41, 59, 0.8);
  border-color: rgba(255, 255, 255, 0.1);
}

.result-card.success { border-top: 4px solid #10b981; }
.result-card.error { border-top: 4px solid #ef4444; }
.result-card.pending { border-top: 4px solid #64748b; opacity: 0.7; }

.card-header {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin-bottom: 1.5rem;
  padding-bottom: 1rem;
  border-bottom: 1px solid rgba(255, 255, 255, 0.05);
}

.method {
  font-size: 0.75rem;
  font-weight: 800;
  padding: 0.25rem 0.6rem;
  border-radius: 6px;
  text-transform: uppercase;
}

.method.get { background: rgba(59, 130, 246, 0.2); color: #60a5fa; }
.method.post { background: rgba(16, 185, 129, 0.2); color: #34d399; }

.url-label {
  font-weight: 600;
  font-size: 1.1rem;
  color: #f1f5f9;
}

.metrics {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1rem;
  margin-bottom: 1.5rem;
}

.metric {
  background: rgba(15, 23, 42, 0.5);
  padding: 1rem;
  border-radius: 12px;
  text-align: center;
}

.metric-label {
  display: block;
  font-size: 0.75rem;
  color: #94a3b8;
  margin-bottom: 0.5rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.metric-value {
  font-size: 1.25rem;
  font-weight: 700;
  color: #fff;
}

.metric-value-tag {
  font-size: 1rem;
  font-weight: 700;
  padding: 0.25rem 0.75rem;
  border-radius: 99px;
  background: rgba(255,255,255,0.1);
  color: #cbd5e1;
}

.metric-value-tag.hit { background: rgba(16, 185, 129, 0.2); color: #34d399; }
.metric-value-tag.miss { background: rgba(245, 158, 11, 0.2); color: #fbbf24; }

.response-preview {
  flex-grow: 1;
  background: rgba(0, 0, 0, 0.3);
  border-radius: 12px;
  padding: 1rem;
  font-family: 'JetBrains Mono', monospace;
  font-size: 0.875rem;
  overflow-x: auto;
  position: relative;
}

.preview-label {
  position: absolute;
  top: 0.5rem;
  right: 0.75rem;
  font-size: 0.7rem;
  color: #64748b;
  text-transform: uppercase;
}

pre {
  margin: 0;
  color: #e2e8f0;
  white-space: pre-wrap;
  word-break: break-all;
}

.placeholder {
  color: #64748b;
  font-style: italic;
  text-align: center;
  padding: 2rem 0;
}

.error-msg {
  color: #ef4444;
}

/* Loader */
.loader {
  width: 20px;
  height: 20px;
  border: 3px solid #ffffff;
  border-bottom-color: transparent;
  border-radius: 50%;
  display: inline-block;
  box-sizing: border-box;
  animation: rotation 1s linear infinite;
}

@keyframes rotation {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

@media (max-width: 768px) {
  .monitor-container { padding: 1rem; }
  .glass-panel { padding: 1.5rem; }
  .controls { flex-direction: column; align-items: stretch; }
  input, .run-btn { width: 100%; }
}
</style>
