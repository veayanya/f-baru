<template>
  <div class="main-dashboard-container">
    <div class="page-header">
      <span class="page-kicker">Beranda</span>
      <h2 class="page-title-lg">Sistem Informasi Analisis Valuasi</h2>
      <p class="page-desc">Overview status sistem, statistik arsip, dan integrasi Artificial Intelligence Bapperida.</p>
    </div>

    <!-- Stats Grid -->
    <div class="stats-grid">
      
      <!-- Card: Model Gemini 2.5 Flash -->
      <div class="stat-card">
        <div class="stat-icon" style="background: rgba(46, 125, 116, 0.15); color: #2E7D74;">
          <i data-lucide="cpu"></i>
        </div>
        <div class="stat-content">
          <div class="stat-label">Model Gemini 2.5 Flash</div>
          <div class="stat-value" style="color: var(--success-color); display: flex; align-items: center; gap: 8px;">
            <div class="status-dot pulsing"></div> Siap
          </div>
          <div class="stat-subtext">Terhubung via Google AI Studio</div>
        </div>
      </div>

      <!-- Card: Jumlah Dokumen -->
      <div class="stat-card">
        <div class="stat-icon" style="background: rgba(201, 123, 61, 0.15); color: #C97B3D;">
          <i data-lucide="file-text"></i>
        </div>
        <div class="stat-content">
          <div class="stat-label">Arsip Dokumen RKA</div>
          <div class="stat-value">{{ rkis.length }} <span style="font-size: 1rem; color: var(--text-muted);">Dokumen</span></div>
          <div class="stat-subtext">Tersimpan dalam database</div>
        </div>
      </div>

      <!-- Card: Chatbot Status -->
      <div class="stat-card">
        <div class="stat-icon" style="background: rgba(60, 156, 109, 0.15); color: #3C9C6D;">
          <i data-lucide="message-square"></i>
        </div>
        <div class="stat-content">
          <div class="stat-label">Agentic AI Chatbot</div>
          <div class="stat-value" style="color: var(--warning-color, #C97B3D); display: flex; align-items: center; gap: 8px;">
            <div class="status-dot" style="background-color: var(--warning-color, #C97B3D);"></div> Belum Siap
          </div>
          <div class="stat-subtext">Bapperida AI Assistant sedang dalam pengembangan</div>
        </div>
      </div>

      <!-- Card: Kecepatan Upload -->
      <div class="stat-card">
        <div class="stat-icon" style="background: rgba(139, 92, 246, 0.15); color: #8b5cf6;">
          <i data-lucide="zap"></i>
        </div>
        <div class="stat-content">
          <div class="stat-label">Kecepatan Analisis AI</div>
          <div class="stat-value">1.8s <span style="font-size: 0.9rem; color: var(--text-muted);">/ doc</span></div>
          <div class="stat-subtext">Rata-rata waktu hingga hasil Nilai Prakiraan Dampak muncul</div>
        </div>
      </div>

      <!-- Card: Penggunaan Penyimpanan Storage -->
      <div class="stat-card">
        <div class="stat-icon" style="background: rgba(37, 99, 235, 0.15); color: #2563eb;">
          <i data-lucide="database"></i>
        </div>
        <div class="stat-content" style="width: 100%;">
          <div class="stat-label">Penggunaan Penyimpanan</div>
          <div class="stat-value">{{ storageUsage.usedLabel }} <span style="font-size: 1rem; color: var(--text-muted);">/ {{ storageUsage.quotaLabel }}</span></div>
          <div class="storage-bar">
            <div class="storage-bar-fill" :style="{ width: Math.min(storageUsage.usagePercent, 100) + '%' }"></div>
          </div>
          <div class="stat-subtext">Storage untuk seluruh pengguna ({{ storageUsage.usagePercent }}% terpakai)</div>
        </div>
      </div>

    </div>

    <!-- Banner: Analisis Valuasi Prakiraan Dampak Program -->
    <div class="banner-card" @click="goToAnalyzer">
      <div class="banner-content">
        <div class="banner-badge">Fitur Utama</div>
        <h3 class="banner-title">Analisis Valuasi Prakiraan Dampak Program</h3>
        <p class="banner-text">Mulai Analisis Valuasi Prakiraan Dampak Program dengan unggah dokumen RKA untuk mendapat hasil analisis sesuai panduan kecerdasan buatan.</p>
        <button class="btn btn-primary banner-btn">
          Mulai Analisis <i data-lucide="arrow-right" style="width: 16px; height: 16px;"></i>
        </button>
      </div>
      <div class="banner-visual">
        <i data-lucide="bar-chart-2"></i>
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, nextTick, reactive } from 'vue';
import { useAnalysis } from '../composables/useAnalysis';
import { apiFetch } from '../utils/api';

const { rkis, currentTab } = useAnalysis();

// Nilai default ditampilkan sambil menunggu (atau jika gagal memuat) data asli
const storageUsage = reactive({
  usedLabel: '0 MB',
  quotaLabel: '0 MB',
  usagePercent: 0
});

function goToAnalyzer() {
  currentTab.value = 'analyzer';
}

async function loadStorageUsage() {
  try {
    const res = await apiFetch('/api/v1/db/storage-summary');
    if (!res.ok) return;
    const data = await res.json();
    if (data.usedLabel) storageUsage.usedLabel = data.usedLabel;
    if (data.quotaLabel) storageUsage.quotaLabel = data.quotaLabel;
    if (typeof data.usagePercent === 'number') storageUsage.usagePercent = data.usagePercent;
  } catch (err) {
    // Diam-diam gunakan nilai default jika endpoint belum tersedia / gagal
  }
}

onMounted(() => {
  if (window.lucide) {
    nextTick(() => window.lucide.createIcons());
  }
  loadStorageUsage();
});
</script>

<style scoped>
.main-dashboard-container {
  display: flex;
  flex-direction: column;
  gap: 24px;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 16px;
}

.stat-card {
  background: var(--bg-secondary);
  border: 1px solid var(--border-color);
  border-radius: var(--border-radius-lg);
  padding: 20px;
  display: flex;
  align-items: flex-start;
  gap: 16px;
  box-shadow: var(--shadow-raised);
  transition: transform 0.2s, box-shadow 0.2s;
}

.stat-card:hover {
  transform: translateY(-2px);
  box-shadow: var(--shadow-raised-hover);
}

.stat-icon {
  width: 48px;
  height: 48px;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.stat-icon i {
  width: 24px;
  height: 24px;
}

.stat-content {
  display: flex;
  flex-direction: column;
}

.stat-label {
  font-size: 0.85rem;
  color: var(--text-secondary);
  font-weight: 600;
  margin-bottom: 4px;
}

.stat-value {
  font-size: 1.5rem;
  font-weight: 800;
  color: var(--text-primary);
  font-family: var(--font-heading);
  margin-bottom: 2px;
}

.stat-subtext {
  font-size: 0.75rem;
  color: var(--text-muted);
}

.storage-bar {
  width: 100%;
  height: 6px;
  border-radius: 4px;
  background: rgba(37, 99, 235, 0.12);
  overflow: hidden;
  margin: 6px 0 6px;
}

.storage-bar-fill {
  height: 100%;
  border-radius: 4px;
  background: #2563eb;
  transition: width 0.4s ease;
}

.status-dot {
  width: 10px;
  height: 10px;
  background-color: var(--success-color);
  border-radius: 50%;
  position: relative;
}

.status-dot.pulsing::after {
  content: '';
  position: absolute;
  inset: 0;
  border-radius: 50%;
  background-color: var(--success-color);
  animation: pulse-ring 2s infinite;
}

@keyframes pulse-ring {
  0% { transform: scale(1); opacity: 0.8; }
  100% { transform: scale(2.5); opacity: 0; }
}

.banner-card {
  background: var(--gradient-brand);
  border-radius: var(--border-radius-lg);
  padding: 32px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: white;
  box-shadow: 0 10px 30px var(--primary-glow);
  cursor: pointer;
  position: relative;
  overflow: hidden;
  transition: transform 0.2s;
}

.banner-card:hover {
  transform: scale(1.01);
}

.banner-card::before {
  content: '';
  position: absolute;
  top: 0; right: 0; bottom: 0; left: 0;
  background-image: var(--retro-dot-color);
  background-size: var(--retro-grid-size) var(--retro-grid-size);
  opacity: 0.5;
  pointer-events: none;
}

.banner-content {
  position: relative;
  z-index: 1;
  max-width: 600px;
}

.banner-badge {
  background: rgba(255, 255, 255, 0.2);
  display: inline-block;
  padding: 4px 10px;
  border-radius: 20px;
  font-size: 0.75rem;
  font-weight: 700;
  letter-spacing: 0.5px;
  margin-bottom: 12px;
}

.banner-title {
  font-family: var(--font-heading);
  font-size: 1.8rem;
  font-weight: 800;
  margin-bottom: 12px;
}

.banner-text {
  font-size: 0.95rem;
  color: rgba(255, 255, 255, 0.85);
  line-height: 1.5;
  margin-bottom: 24px;
}

.banner-btn {
  background: white;
  color: var(--primary-color);
  border: none;
  font-weight: 700;
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 10px 20px;
  border-radius: 8px;
}

.banner-btn:hover {
  background: #f0f0f0;
}

.banner-visual {
  position: relative;
  z-index: 1;
  opacity: 0.15;
  transform: scale(4);
  transform-origin: center right;
  margin-right: 40px;
}
</style>
