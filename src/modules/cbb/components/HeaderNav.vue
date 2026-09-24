<template>
  <header class="cbb-glass-panel" style="padding: 0.875rem 1.5rem; margin-bottom: 1.25rem; display: flex; align-items: center; justify-content: space-between; flex-wrap: wrap; gap: 1rem;">
    <!-- Logo & Title -->
    <div style="display: flex; align-items: center; gap: 0.875rem;">
      <div style="width: 44px; height: 44px; border-radius: 12px; background: var(--primary-color); display: flex; align-items: center; justify-content: center; font-size: 1.5rem; box-shadow: 0 4px 12px color-mix(in srgb, var(--primary-color) 40%, transparent); border: 1px solid var(--border-color-strong);">
        🏛️
      </div>
      <div>
        <div style="display: flex; align-items: center; gap: 0.5rem; flex-wrap: wrap;">
          <h1 style="font-size: 1.125rem; font-weight: 700; color: var(--text-primary); letter-spacing: -0.01em;">
            BAPPERIDA KABUPATEN CIREBON
          </h1>
          <span class="cbb-badge cbb-badge-emerald" style="font-size: 0.6875rem;">
            <span class="cbb-pulse-indicator"></span> Asisten AI Aktif
          </span>
        </div>
        <p style="font-size: 0.775rem; color: var(--text-secondary);">
          Asisten Perencanaan Pembangunan, Penelitian &amp; Pengembangan Daerah
        </p>
      </div>
    </div>

    <!-- Right Controls: Tool Modals only -->
    <div style="display: flex; align-items: center; gap: 0.75rem; flex-wrap: wrap;">
      <button
        id="btn-diagnosis-open"
        @click="$emit('open-diagnosis')"
        class="cbb-btn-secondary"
        style="padding: 0.45rem 0.85rem; font-size: 0.775rem; border-color: color-mix(in srgb, var(--danger-color) 40%, transparent);"
      >
        <span>🩺</span> Diagnosis Masalah
      </button>

      <button
        id="btn-calculator-open"
        @click="$emit('open-calculator')"
        class="cbb-btn-secondary"
        style="padding: 0.45rem 0.85rem; font-size: 0.775rem;"
      >
        <span>🧮</span> Kalkulator Kinerja
      </button>

      <button
        id="btn-docs-open"
        @click="$emit('open-docs')"
        class="cbb-btn-secondary"
        style="padding: 0.45rem 0.85rem; font-size: 0.775rem;"
      >
        <span>📚</span> Dokumen &amp; Regulasi
      </button>

      <button
        id="btn-embed-open"
        @click="$emit('open-embed')"
        class="cbb-btn-primary"
        style="padding: 0.45rem 0.95rem; font-size: 0.775rem;"
      >
        <span>⚡</span> Sematkan ke Web
      </button>
    </div>
  </header>
</template>

<script setup>
defineEmits(['open-diagnosis', 'open-calculator', 'open-docs', 'open-embed']);
</script>
