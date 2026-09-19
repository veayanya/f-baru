<template>
  <div class="login-root">

    <!-- Panel Kiri: Gambar -->
    <div class="login-left">
      <!--
        Ganti src di bawah dengan path gambar Anda.
        Gambar login berada di public/images/login-bg.jpg
        lalu bind: :src="loginBg"
        Atau langsung string path: src="/images/login-bg.jpg"
      -->
      <img :src="loginBg" alt="" class="login-left-img" />
    </div>

    <!-- Panel Kanan: Form Login -->
    <div class="login-right">
      <div class="login-form-box">

        <span class="star-ornament" aria-hidden="true">✦</span>
        <h2 class="form-title">Login</h2>
        <p class="form-sub">Selamat datang kembali di Sistem Analisis Valuasi Prakiraan Dampak Program</p>

        <form @submit.prevent="handleLogin">
          <!-- Username -->
          <div class="field" :class="{ 'has-error': errors.username }">
            <i data-lucide="user" class="field-ico"></i>
            <input
              v-model="form.username"
              type="text"
              placeholder="Username"
              autocomplete="username"
              :disabled="loading"
              @input="errors.username = ''"
            />
          </div>
          <p v-if="errors.username" class="field-err">{{ errors.username }}</p>

          <!-- Password -->
          <div class="field" :class="{ 'has-error': errors.password }">
            <i data-lucide="lock" class="field-ico"></i>
            <input
              v-model="form.password"
              :type="showPassword ? 'text' : 'password'"
              placeholder="Password"
              autocomplete="current-password"
              :disabled="loading"
              @input="errors.password = ''"
            />
            <button
              type="button"
              class="field-ico-right"
              @click="showPassword = !showPassword"
              tabindex="-1"
              :aria-label="showPassword ? 'Sembunyikan password' : 'Tampilkan password'"
            >
              <i :data-lucide="showPassword ? 'eye-off' : 'eye'"></i>
            </button>
          </div>
          <p v-if="errors.password" class="field-err">{{ errors.password }}</p>

          <!-- Error login -->
          <div v-if="loginError" class="login-error-alert">
            <i data-lucide="alert-circle"></i>
            <span>{{ loginError }}</span>
          </div>

          <!-- Submit -->
          <button type="submit" class="login-btn" :disabled="loading">
            <span v-if="loading" class="btn-spin">
              <i data-lucide="loader-2" class="spin-icon"></i>
              Memverifikasi...
            </span>
            <span v-else>
              Login
              <i data-lucide="arrow-right"></i>
            </span>
          </button>
        </form>

        <p class="login-credit">ⓒ Copyright 2026 Bapperida Kab. Cirebon. All rights reserved</p>
      </div>
    </div>

  </div>
</template>

<script setup>
import { ref, onMounted, nextTick } from 'vue';
import { apiFetch } from '@/utils/api';

// ── Ganti path ini dengan gambar Anda ──────────────────────────────────────
// Opsi 1: import langsung (recommended, di-bundle oleh Vite)
const loginBg = '/images/login-bg.jpg';
// Opsi 2: kalau mau pakai URL string statis, hapus import di atas dan ganti:
// const loginBg = '/images/login-bg.jpg';
// ───────────────────────────────────────────────────────────────────────────

const emit = defineEmits(['login-success']);

const form = ref({ username: '', password: '' });
const errors = ref({ username: '', password: '' });
const showPassword = ref(false);
const loading = ref(false);
const loginError = ref('');
const year = new Date().getFullYear();

function validate() {
  let ok = true;
  errors.value.username = '';
  errors.value.password = '';
  if (!form.value.username.trim()) {
    errors.value.username = 'Username wajib diisi.';
    ok = false;
  }
  if (!form.value.password) {
    errors.value.password = 'Password wajib diisi.';
    ok = false;
  }
  return ok;
}

async function handleLogin() {
  if (!validate()) return;
  loading.value = true;
  loginError.value = '';
  try {
    const res = await apiFetch('/api/auth/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      credentials: 'include',
      body: JSON.stringify({
        username: form.value.username.trim(),
        password: form.value.password
      })
    });
    const data = await res.json();
    if (!res.ok) {
      loginError.value = data.error || 'Login gagal. Periksa username dan password.';
      return;
    }
    if (data.token) {
      localStorage.setItem('auth_token', data.token);
    }
    emit('login-success', data.user);
  } catch (err) {
    loginError.value = err.name === 'AbortError'
      ? 'Server backend tidak merespons. Tunggu beberapa saat lalu coba lagi.'
      : 'Tidak dapat terhubung ke server. Pastikan backend sudah berjalan.';
  } finally {
    loading.value = false;
  }
}

onMounted(() => {
  nextTick(() => {
    if (window.lucide) window.lucide.createIcons();
  });
});
</script>

<style scoped>
/* === ROOT === */
.login-root {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
  gap: 0;
  padding: 6vh 5vw;
  box-sizing: border-box;
  font-family: 'Outfit', 'Inter', sans-serif;
  background: #ffffff;
}

.login-root::before {
  content: '';
  position: absolute;
  width: min(1100px, 90vw);
  height: min(78vh, 620px);
  border-radius: 20px;
  background: #f5f2ec;
  box-shadow: 0 24px 60px rgba(30, 47, 94, 0.16);
}

/* === LEFT PANEL === */
.login-left {
  position: relative;
  z-index: 1;
  flex: 0 1 min(48%, 520px);
  height: min(78vh, 620px);
  overflow: hidden;
  border-radius: 20px 0 0 20px;
  background: #0c1428; /* fallback kalau gambar belum load */
}

.login-left-img {
  width: 100%;
  height: 100%;
  object-fit: cover;       /* crop rapi, isi penuh panel */
  object-position: center; /* titik fokus gambar — ubah ke 'top' / 'left' dll sesuai gambar */
  display: block;
}

/* === RIGHT PANEL === */
.login-right {
  z-index: 1;
  flex: 0 1 min(52%, 560px);
  height: min(78vh, 620px);
  box-sizing: border-box;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 0 20px 20px 0;
  background: #f5f2ec;
  padding: 40px 24px;
}

.login-form-box {
  width: 100%;
  max-width: 300px;
}

/* Star ornament */
.star-ornament {
  display: block;
  font-size: 1.5rem;
  color: #b8912a;
  margin-bottom: 14px;
  line-height: 1;
}

/* Form title */
.form-title {
  font-size: 2rem;
  font-weight: 400;
  color: #1e2640;
  margin: 0 0 6px;
  letter-spacing: -0.4px;
}

.form-sub {
  font-size: 0.82rem;
  color: #7a8099;
  margin: 0 0 28px;
}

/* Input field */
.field {
  position: relative;
  margin-bottom: 6px;
}

.field input {
  width: 100%;
  padding: 13px 14px 13px 42px;
  background: #ffffff;
  border: 1.5px solid #e2ddd5;
  border-radius: 12px;
  font-size: 0.88rem;
  color: #1e2640;
  outline: none;
  transition: border-color 0.2s, box-shadow 0.2s;
  box-sizing: border-box;
  font-family: inherit;
}

.field input::placeholder {
  color: #b0aaa0;
}

.field input:focus {
  border-color: #2e3f6e;
  box-shadow: 0 0 0 3px rgba(46, 63, 110, 0.1);
}

.field input:disabled {
  opacity: 0.55;
  cursor: not-allowed;
}

.field.has-error input {
  border-color: #c0394b;
}

/* Field icons */
.field-ico {
  position: absolute;
  left: 13px;
  top: 50%;
  transform: translateY(-50%);
  width: 16px;
  height: 16px;
  color: #b0aaa0;
  pointer-events: none;
}

.field-ico-right {
  position: absolute;
  right: 13px;
  top: 50%;
  transform: translateY(-50%);
  background: none;
  border: none;
  cursor: pointer;
  padding: 0;
  display: flex;
  align-items: center;
  color: #b0aaa0;
  transition: color 0.15s;
}

.field-ico-right:hover { color: #555; }

.field-ico-right i {
  width: 16px;
  height: 16px;
}

/* Field error */
.field-err {
  font-size: 0.72rem;
  color: #c0394b;
  margin: 0 0 10px 4px;
}

/* Login error alert */
.login-error-alert {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 10px 14px;
  background: rgba(192, 57, 75, 0.08);
  border: 1px solid rgba(192, 57, 75, 0.22);
  border-radius: 10px;
  color: #a02a3a;
  font-size: 0.8rem;
  margin-bottom: 12px;
  animation: shake 0.4s ease;
}

.login-error-alert i {
  width: 15px;
  height: 15px;
  flex-shrink: 0;
}

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  20%, 60% { transform: translateX(-4px); }
  40%, 80% { transform: translateX(4px); }
}

/* Login button */
.login-btn {
  width: 100%;
  padding: 14px;
  background: #1e2f5e;
  border: none;
  border-radius: 12px;
  font-size: 0.9rem;
  font-weight: 500;
  color: #ffffff;
  cursor: pointer;
  margin-top: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  font-family: inherit;
  transition: background 0.2s, transform 0.15s, box-shadow 0.2s;
  letter-spacing: 0.03em;
}

.login-btn:hover:not(:disabled) {
  background: #162348;
  transform: translateY(-1px);
  box-shadow: 0 6px 18px rgba(30, 47, 94, 0.28);
}

.login-btn:active:not(:disabled) {
  transform: translateY(0);
  box-shadow: none;
}

.login-btn:disabled {
  opacity: 0.65;
  cursor: not-allowed;
}

.login-btn i {
  width: 16px;
  height: 16px;
}

.btn-spin {
  display: flex;
  align-items: center;
  gap: 8px;
}

.spin-icon {
  animation: spin 1s linear infinite;
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

/* Credit */
.login-credit {
  font-size: 0.68rem;
  color: #b0aaa0;
  text-align: center;
  margin: 24px 0 0;
}

/* === RESPONSIVE === */
@media (max-width: 768px) {
  .login-root {
    flex-direction: column;
    align-items: stretch;
    justify-content: flex-start;
    padding: 0;
  }
  .login-root::before { display: none; }
  .login-left {
    flex: 0 0 220px;
    height: 220px;
    min-height: 220px;
    border-radius: 0;
  }
  .login-right {
    flex: 1;
    height: auto;
    min-height: calc(100vh - 220px);
    border-radius: 0;
    padding: 32px 24px;
  }
  .form-title { font-size: 1.6rem; }
}
</style>
