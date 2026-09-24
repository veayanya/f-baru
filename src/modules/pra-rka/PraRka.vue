<template>
  <div class="pra-rka-module">
    <!-- ── Sub-navigasi modul Pra RKA ─────────────────────────────── -->
    <div class="pra-rka-nav">
      <button
        v-for="m in menus"
        :key="m.id"
        type="button"
        :class="['pra-rka-navbtn', { active: activeMenu === m.id }]"
        @click="activeMenu = m.id"
      >
        <i :class="['fa-solid', m.icon]"></i>
        <span>{{ m.label }}</span>
      </button>
    </div>

    <!-- ── Konten ─────────────────────────────────────────────────── -->
    <BeritaAcaraRenja v-if="activeMenu === 'berita-acara'" />

    <!-- Placeholder untuk menu lain yang akan dikembangkan -->
    <div v-else class="pra-rka-placeholder">
      <i class="fa-solid fa-tools" style="font-size:2rem; opacity:.35;"></i>
      <p>Modul <strong>{{ menus.find(m => m.id === activeMenu)?.label }}</strong> sedang dalam pengembangan.</p>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import BeritaAcaraRenja from './components/BeritaAcaraRenja.vue';

const menus = [
  { id: 'berita-acara', label: 'Berita Acara Verifikasi Renja PD', icon: 'fa-file-contract' },
  { id: 'formulir-renja', label: 'Formulir Renja PD', icon: 'fa-table-list' },
  { id: 'kesesuaian-rkpd', label: 'Kesesuaian RKPD', icon: 'fa-check-double' },
];

const activeMenu = ref('berita-acara');
</script>

<style scoped>
.pra-rka-module { display: flex; flex-direction: column; gap: 16px; }

.pra-rka-nav {
  display: flex;
  gap: 6px;
  flex-wrap: wrap;
  padding: 10px 14px;
  background: var(--bg-secondary);
  border: 1px solid var(--border-color);
  border-radius: var(--card-border-radius);
}

.pra-rka-navbtn {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 6px 14px;
  border-radius: var(--border-radius-md);
  border: 1px solid var(--border-color);
  background: var(--bg-tertiary);
  color: var(--text-secondary);
  font-size: 0.82rem;
  font-weight: 600;
  cursor: pointer;
  transition: all var(--transition-speed);
}
.pra-rka-navbtn:hover { background: var(--bg-hover); color: var(--text-primary); }
.pra-rka-navbtn.active {
  background: var(--primary-color);
  color: #fff;
  border-color: var(--primary-color);
}

.pra-rka-placeholder {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 12px;
  padding: 60px 20px;
  text-align: center;
  color: var(--text-secondary);
  background: var(--bg-secondary);
  border: 1px dashed var(--border-color);
  border-radius: var(--card-border-radius);
}
</style>
