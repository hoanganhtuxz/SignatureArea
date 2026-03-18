<script setup lang="ts">
import SignatureArea from './components/SignatureArea.vue';
</script>

<template>
  <main class="app-container">
    <header class="app-header">
      <h1>e-Signature</h1>
      <p>Sign documents digitally on your device</p>
    </header>
    
    <section class="signing-section">
      <SignatureArea />
    </section>
    
    <footer class="app-footer">
      <p>Secure & fast digital signatures.</p>
    </footer>
  </main>
</template>

<style scoped>
.app-container {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  padding: 1.5rem;
  background: var(--bg-main);
  background-image: 
    radial-gradient(at 0% 0%, hsla(253,16%,7%,1) 0, transparent 50%), 
    radial-gradient(at 50% 0%, hsla(225,39%,30%,1) 0, transparent 50%), 
    radial-gradient(at 100% 0%, hsla(339,49%,30%,1) 0, transparent 50%);
  background-size: cover;
  background-attachment: fixed;
  color: var(--text-main);
  font-family: 'Inter', system-ui, sans-serif;
}

.app-container {
  background: #f8fafc;
  background-image: 
    radial-gradient(at 0% 0%, hsla(228,100%,74%,0.15) 0, transparent 50%), 
    radial-gradient(at 50% 0%, hsla(225,39%,80%,0.15) 0, transparent 50%), 
    radial-gradient(at 100% 0%, hsla(339,49%,80%,0.15) 0, transparent 50%);
}

.app-header {
  text-align: center;
  margin-bottom: 2rem;
  padding-top: 1rem;
}

.app-header h1 {
  font-size: 2.5rem;
  font-weight: 800;
  color: #0f172a;
  margin: 0 0 0.5rem 0;
  letter-spacing: -0.025em;
  background: linear-gradient(135deg, #1e293b, #4f46e5);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

.app-header p {
  color: #64748b;
  font-size: 1.125rem;
  margin: 0;
}

.signing-section {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: flex-start;
  width: 100%;
}

.app-footer {
  text-align: center;
  padding-top: 2rem;
  color: #94a3b8;
  font-size: 0.875rem;
}

@media (min-width: 640px) {
  .app-container {
    padding: 2rem;
  }
  
  .app-header h1 {
    font-size: 3rem;
  }
}
</style>
