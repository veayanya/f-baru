<template>
  <div class="erenja">
    <!-- ── Toolbar: identitas studio + mode pengguna ─────────────────── -->
    <div class="er-toolbar">
      <div class="er-brand">
        <div class="er-brand-icon" aria-hidden="true"><i class="fa-solid fa-shield-halved"></i></div>
        <div class="er-brand-text">
          <h3 class="er-brand-title">
            e-Renja Bapperida Studio
            <span class="badge badge-primary">{{ roleLabel }}</span>
          </h3>
          <p class="er-brand-sub">Pemerintah Kabupaten Cirebon • Standar F4 (215 x 330 mm)</p>
        </div>
      </div>

      <label class="er-role">
        <span>Mode kerja</span>
        <select class="form-input-sm" v-model="userRole" @change="onRoleChange">
          <option value="admin">Administrator Bapperida (Super Admin)</option>
          <option value="bapperida">Tim Verifikator Perencana Bapperida</option>
          <option value="opd">Operator Perangkat Daerah (OPD)</option>
        </select>
      </label>
    </div>

    <!-- ── Navigasi modul ─────────────────────────────────────────────── -->
    <div class="er-tabs" role="tablist" aria-label="Modul e-Renja Studio">
      <button
        v-for="t in tabs"
        :key="t.id"
        type="button"
        role="tab"
        :aria-selected="currentTab === t.id"
        :class="['er-tab', { active: currentTab === t.id }]"
        @click="currentTab = t.id"
      >
        <i :class="['fa-solid', t.icon]" aria-hidden="true"></i>
        <span>{{ t.label }}</span>
      </button>
    </div>

    <!-- ══ MODUL: DASHBOARD ═════════════════════════════════════════════ -->
    <section v-show="currentTab === 'dashboard'" class="er-panel" role="tabpanel">
      <div class="er-panel-head">
        <div>
          <h4 class="er-h">Dashboard Dokumen Perencanaan</h4>
          <p class="er-sub">Ringkasan status penyusunan Renja, RKPD, dan dokumen perencanaan 71 Perangkat Daerah.</p>
        </div>
        <button type="button" class="btn btn-primary btn-sm" @click="currentTab = 'editor'">
          <i class="fa-solid fa-plus" aria-hidden="true"></i> Buat Dokumen Renja Baru
        </button>
      </div>

      <div class="er-metrics">
        <div v-for="m in metrics" :key="m.label" class="er-metric">
          <div class="er-metric-label">{{ m.label }}</div>
          <div class="er-metric-value" :class="m.tone">
            {{ m.value }} <small>{{ m.unit }}</small>
          </div>
          <div class="er-metric-note">{{ m.note }}</div>
        </div>
      </div>

      <div class="document-pane">
        <div class="pane-header">
          <span class="pane-title">Dokumen Perencanaan Terbaru (71 OPD)</span>
          <span class="er-sub">Tahun Anggaran 2027</span>
        </div>
        <div class="er-table-scroll">
          <table class="er-table">
            <thead>
              <tr>
                <th>Lampiran</th>
                <th>Perangkat Daerah</th>
                <th>Jenis Template</th>
                <th>Status</th>
                <th>Aksi</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="d in documents" :key="d.lampiran">
                <td class="er-code">{{ d.lampiran }}</td>
                <td class="er-strong">{{ d.opd }}</td>
                <td><span :class="['er-tag', d.jenis === 'RENJA' ? 'er-tag-renja' : 'er-tag-rkpd']">{{ d.jenis }}</span></td>
                <td><span :class="['badge', statusBadge[d.status].cls]">{{ statusBadge[d.status].label }}</span></td>
                <td>
                  <button type="button" class="btn btn-secondary btn-sm" @click="openDocument(d)">
                    {{ d.aksi }}
                  </button>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </section>

    <!-- ══ MODUL: SMART EDITOR F4 ═══════════════════════════════════════ -->
    <section v-show="currentTab === 'editor'" class="er-panel" role="tabpanel">
      <div class="document-pane">
        <div class="er-ribbon">
          <div class="er-ribbon-tabs" role="tablist" aria-label="Pita menu editor">
            <button
              type="button"
              role="tab"
              :aria-selected="activeRibbon === 'utama'"
              :class="['er-ribbon-tab', { active: activeRibbon === 'utama' }]"
              @click="activeRibbon = 'utama'"
            >Utama</button>
            <button
              type="button"
              role="tab"
              :aria-selected="activeRibbon === 'layout_tabel'"
              :class="['er-ribbon-tab', { active: activeRibbon === 'layout_tabel' }]"
              @click="activeRibbon = 'layout_tabel'"
            >
              <i class="fa-solid fa-table-cells" aria-hidden="true"></i> Layout Tabel
            </button>
          </div>

          <div v-show="activeRibbon === 'utama'" class="er-ribbon-body">
            <div class="er-ribbon-group">
              <span class="er-ribbon-label">Kertas</span>
              <span class="er-chip er-chip-on">F4 / Folio (215 x 330 mm)</span>
              <span class="er-chip">Margin 2,0 cm</span>
            </div>
            <div class="er-ribbon-group">
              <span class="er-ribbon-label">Huruf</span>
              <span class="er-chip">Bookman Old Style 12 pt</span>
              <span class="er-chip er-chip-danger">Tanpa cetak tebal</span>
            </div>
            <button
              type="button"
              class="btn btn-primary btn-sm"
              @click="simulate('Export DOCX', 'Berkas DOCX F4, margin 2 cm, Bookman Old Style 12 pt disiapkan.')"
            >
              <i class="fa-solid fa-file-word" aria-hidden="true"></i> Export DOCX (F4 margin 2 cm)
            </button>
          </div>

          <div v-show="activeRibbon === 'layout_tabel'" class="er-ribbon-body">
            <div class="er-ribbon-group">
              <button type="button" class="btn btn-secondary btn-sm" @click="addRow('top')">+ Baris atas</button>
              <button type="button" class="btn btn-secondary btn-sm" @click="addRow('bottom')">+ Baris bawah</button>
            </div>
            <div class="er-ribbon-group">
              <button type="button" class="btn btn-secondary btn-sm" @click="simulate('Merge cells', 'Sel yang dipilih digabung.')">Merge cells</button>
              <button type="button" class="btn btn-secondary btn-sm" @click="simulate('Split cells', 'Sel yang dipilih dipisah.')">Split cells</button>
            </div>
            <div class="er-ribbon-group">
              <span class="er-ribbon-label">Perataan sel</span>
              <div class="er-align-grid" role="group" aria-label="Perataan sel">
                <button
                  v-for="i in 9"
                  :key="i"
                  type="button"
                  :class="['er-align-cell', { active: alignCell === i }]"
                  :aria-label="'Posisi ' + i"
                  :aria-pressed="alignCell === i"
                  @click="alignCell = i"
                ></button>
              </div>
            </div>
            <span class="er-chip er-chip-on">AutoFit 100%</span>
          </div>
        </div>

        <!-- Kanvas: kertas selalu putih agar sama dengan hasil cetak, di tema terang maupun gelap -->
        <div class="er-canvas">
          <article class="er-paper">
            <div class="er-perbup">
              <div>LAMPIRAN LIII</div>
              <div>PERATURAN BUPATI CIREBON</div>
              <div>NOMOR&emsp;&emsp;&emsp;&emsp;TAHUN 2026</div>
              <div>TENTANG</div>
              <div>RENCANA KERJA PERANGKAT DAERAH TAHUN 2027</div>
            </div>

            <div class="er-chapter">
              <div>BAB I</div>
              <div>PENDAHULUAN</div>
            </div>

            <div class="er-para">
              <div class="er-para-title">1.1. Latar Belakang</div>
              <p>
                Rencana Kerja Perangkat Daerah (Renja PD) Kecamatan Losari Kabupaten Cirebon Tahun 2027 merupakan dokumen perencanaan tahunan yang disusun untuk memberikan arah kebijakan, program, dan kegiatan OPD. Penyusunan dokumen dilakukan dengan mematuhi ketentuan standar penulisan naskah dinas resmi Pemerintah Kabupaten Cirebon (ukuran kertas F4 / Folio 215 x 330 mm, margin seragam 2,0 cm di seluruh sisi, dan font Bookman Old Style 12 pt).
              </p>
            </div>

            <div class="er-para">
              <div class="er-para-title">Tabel 1.1 Matriks Evaluasi Capaian Renja Tahun Berjalan</div>
              <table class="er-paper-table">
                <thead>
                  <tr>
                    <th>No</th>
                    <th>Kode Program / Kegiatan</th>
                    <th>Indikator Kinerja</th>
                    <th>Target 2026</th>
                    <th>Realisasi (%)</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="(row, idx) in rows" :key="row.id">
                    <td class="c">{{ idx + 1 }}</td>
                    <td contenteditable="true" @blur="row.kode = $event.target.innerText.trim()">{{ row.kode }}</td>
                    <td contenteditable="true" @blur="row.indikator = $event.target.innerText.trim()">{{ row.indikator }}</td>
                    <td class="c" contenteditable="true" @blur="row.target = $event.target.innerText.trim()">{{ row.target }}</td>
                    <td class="c" contenteditable="true" @blur="row.realisasi = $event.target.innerText.trim()">{{ row.realisasi }}</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </article>
        </div>
      </div>
    </section>

    <!-- ══ MODUL: MESIN CUCI DOKUMEN ════════════════════════════════════ -->
    <section v-show="currentTab === 'formatter'" class="er-panel" role="tabpanel">
      <div class="er-panel-head">
        <div>
          <h4 class="er-h">Mesin Cuci Dokumen Smart Template</h4>
          <p class="er-sub er-sub-wide">
            Unggah berkas MS Word (.docx). Dokumen dirapikan otomatis: kata pengantar dan daftar isi untuk Renja dibuang, kertas dipaksa F4 dengan margin 2 cm, dan cetak tebal dihapus.
          </p>
        </div>
      </div>

      <div class="document-pane">
        <div class="pane-body er-form-body">
          <div class="er-form-grid">
            <label class="er-field">
              <span class="form-label-sm">Perangkat Daerah (OPD)</span>
              <select class="form-input" v-model="selectedOpd">
                <option v-for="o in opdOptions" :key="o" :value="o">{{ o }}</option>
              </select>
            </label>
            <label class="er-field">
              <span class="form-label-sm">Jenis dokumen template</span>
              <select class="form-input" v-model="selectedTemplate">
                <option v-for="t in templateOptions" :key="t.kode" :value="t.kode">{{ t.label }}</option>
              </select>
            </label>
            <label class="er-field">
              <span class="form-label-sm">Tahun anggaran</span>
              <select class="form-input" v-model="selectedYear">
                <option value="2027">2027 (tahun berjalan)</option>
                <option value="2026">2026</option>
              </select>
            </label>
          </div>

          <label
            :class="['er-drop', { over: dragOver, filled: !!pickedFile }]"
            @dragover.prevent="dragOver = true"
            @dragleave.prevent="dragOver = false"
            @drop.prevent="onDrop"
          >
            <input ref="fileInput" type="file" accept=".docx" class="er-file" @change="onPickFile" />
            <span class="er-drop-icon" aria-hidden="true"><i class="fa-solid fa-cloud-arrow-up"></i></span>
            <template v-if="pickedFile">
              <span class="er-drop-title">{{ pickedFile.name }}</span>
              <span class="er-sub">{{ formatSize(pickedFile.size) }} • klik untuk mengganti berkas</span>
            </template>
            <template v-else>
              <span class="er-drop-title">Klik atau tarik berkas MS Word (.docx) ke sini</span>
              <span class="er-sub">Aturan aktif: <strong class="er-accent">{{ selectedTemplate }}</strong></span>
            </template>
          </label>

          <div class="er-form-actions">
            <button type="button" class="btn btn-secondary btn-sm" :disabled="!pickedFile" @click="clearFile">
              Hapus berkas
            </button>
            <button type="button" class="btn btn-primary" :disabled="!pickedFile" @click="washDocument">
              <i class="fa-solid fa-soap" aria-hidden="true"></i> Cuci dokumen
            </button>
          </div>
        </div>
      </div>
    </section>

    <!-- ══ MODUL: SMART TEMPLATE MANAGER ════════════════════════════════ -->
    <section v-show="currentTab === 'templates'" class="er-panel" role="tabpanel">
      <div class="er-panel-head">
        <div>
          <h4 class="er-h">Kelola Smart Template Dokumen</h4>
          <p class="er-sub">Daftar template dokumen perencanaan daerah yang aktif di sistem.</p>
        </div>
        <button type="button" class="btn btn-primary btn-sm" @click="openTemplateModal">
          <i class="fa-solid fa-plus" aria-hidden="true"></i> Tambah Template Manual
        </button>
      </div>

      <div class="er-cards">
        <div v-for="t in templates" :key="t.kode" class="er-card">
          <div class="er-card-top">
            <span :class="['er-tag', t.kode === 'RKPD' ? 'er-tag-rkpd' : 'er-tag-renja']">{{ t.kode }}</span>
            <span class="badge badge-success">Aktif</span>
          </div>
          <h5 class="er-card-title">{{ t.nama }}</h5>
          <p class="er-card-text">{{ t.deskripsi }}</p>
        </div>
      </div>
    </section>

    <!-- ══ MODUL: REVIEW & VERIFIKASI ═══════════════════════════════════ -->
    <section v-show="currentTab === 'review'" class="er-panel" role="tabpanel">
      <div class="document-pane">
        <div class="pane-header er-review-head">
          <div>
            <span class="page-kicker">Modul Review Verifikator Bapperida</span>
            <h4 class="er-h">Verifikasi Dokumen Perencanaan (Semua Template)</h4>
          </div>
          <label class="er-field er-review-select">
            <span class="form-label-sm">Dokumen target review</span>
            <select class="form-input-sm" v-model="selectedReviewDoc">
              <option v-for="d in reviewDocs" :key="d" :value="d">{{ d }}</option>
            </select>
          </label>
        </div>

        <div class="pane-body er-review-body">
          <div class="er-meta">
            <div class="er-meta-item">
              <span class="er-meta-label">Dokumen target</span>
              <div class="er-strong">{{ selectedReviewDoc }}</div>
            </div>
            <div class="er-meta-item">
              <span class="er-meta-label">Status verifikasi saat ini</span>
              <div><span :class="['badge', reviewStatusView.cls]">{{ reviewStatusView.label }}</span></div>
            </div>
            <div class="er-meta-item">
              <span class="er-meta-label">Standar validasi</span>
              <div class="er-strong er-accent">Page setup F4 (215 x 330 mm), margin 2 cm</div>
            </div>
          </div>

          <div>
            <h5 class="er-h5">Hasil pemindaian skema dan struktur dokumen</h5>
            <div class="er-checks">
              <div class="er-check">
                <span>Kelengkapan bab dan sub-bab struktur</span>
                <span class="er-ok"><i class="fa-solid fa-circle-check" aria-hidden="true"></i> Sesuai schema</span>
              </div>
              <div class="er-check">
                <span>Matriks pagu indikatif dan program</span>
                <span class="er-ok"><i class="fa-solid fa-circle-check" aria-hidden="true"></i> Sesuai SIPD</span>
              </div>
            </div>
          </div>

          <label class="er-field">
            <span class="form-label-sm">Catatan koreksi dan verifikasi Bapperida</span>
            <textarea
              class="form-input er-textarea"
              rows="3"
              placeholder="Tuliskan catatan verifikasi atau arahan revisi untuk Perangkat Daerah..."
              :value="reviewNotes[selectedReviewDoc] || ''"
              @input="reviewNotes[selectedReviewDoc] = $event.target.value"
            ></textarea>
          </label>

          <div class="er-form-actions er-decision">
            <button type="button" class="btn btn-secondary" @click="decide('revision')">
              <i class="fa-solid fa-rotate-left" aria-hidden="true"></i> Minta revisi OPD
            </button>
            <button type="button" class="btn btn-primary" @click="decide('approved')">
              <i class="fa-solid fa-check-double" aria-hidden="true"></i> Setujui dokumen
            </button>
          </div>
        </div>
      </div>
    </section>

    <!-- ══ MODUL: MONITORING ════════════════════════════════════════════ -->
    <section v-show="currentTab === 'monitoring'" class="er-panel" role="tabpanel">
      <div class="er-panel-head">
        <div>
          <h4 class="er-h">Monitoring Penyerahan Dokumen 71 OPD</h4>
          <p class="er-sub">Pantau progres pengunggahan dan verifikasi dokumen seluruh Perangkat Daerah Kabupaten Cirebon.</p>
        </div>
      </div>
      <div class="document-pane">
        <div class="pane-body er-progress-body">
          <div class="er-progress-meta">
            <span>Progres penyerahan: {{ progressPct }}%</span>
            <span>{{ opdSelesai }} dari {{ opdTotal }} OPD selesai</span>
          </div>
          <div
            class="er-progress"
            role="progressbar"
            aria-label="Progres penyerahan dokumen"
            aria-valuemin="0"
            aria-valuemax="100"
            :aria-valuenow="progressPct"
          >
            <div class="er-progress-fill" :style="{ width: progressPct + '%' }"></div>
          </div>
        </div>
      </div>
    </section>

    <!-- ══ MODUL: MASTER OPD & NOMENKLATUR ══════════════════════════════ -->
    <section v-show="currentTab === 'master'" class="er-panel" role="tabpanel">
      <div class="er-panel-head">
        <div>
          <h4 class="er-h">Master Data 71 OPD dan Nomenklatur (Admin)</h4>
          <p class="er-sub">Pemetaan nomor Lampiran Romawi Perbup resmi untuk 31 Dinas/Badan dan 40 Kecamatan.</p>
        </div>
        <button type="button" class="btn btn-primary btn-sm" @click="openMasterModal">
          <i class="fa-solid fa-plus" aria-hidden="true"></i> Tambah Master OPD
        </button>
      </div>
      <div class="document-pane">
        <div class="er-table-scroll">
          <table class="er-table">
            <thead>
              <tr>
                <th>Kode Lampiran</th>
                <th>Nama Perangkat Daerah</th>
                <th>Kategori</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="o in masterOpd" :key="o.kode + o.nama">
                <td class="er-code">{{ o.kode }}</td>
                <td>{{ o.nama }}</td>
                <td>{{ o.kategori }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </section>

    <!-- ══ MODAL: Tambah Template Manual ════════════════════════════════ -->
    <Teleport to="body">
      <div v-if="showTemplateModal" class="er-overlay" @click.self="showTemplateModal = false" @keydown.esc="showTemplateModal = false">
        <div class="er-modal" role="dialog" aria-modal="true" aria-labelledby="er-tpl-title">
          <h4 id="er-tpl-title" class="er-h">Tambah Template Manual</h4>
          <label class="er-field">
            <span class="form-label-sm">Kode template</span>
            <input ref="tplKodeInput" v-model.trim="tplForm.kode" class="form-input" type="text" placeholder="Contoh: KUA_PPAS" />
          </label>
          <label class="er-field">
            <span class="form-label-sm">Nama template</span>
            <input v-model.trim="tplForm.nama" class="form-input" type="text" placeholder="Contoh: Kebijakan Umum APBD & PPAS" />
          </label>
          <div class="er-form-actions">
            <button type="button" class="btn btn-secondary btn-sm" @click="showTemplateModal = false">Batal</button>
            <button type="button" class="btn btn-primary btn-sm" @click="saveTemplate">Simpan template</button>
          </div>
        </div>
      </div>

      <!-- ══ MODAL: Tambah Master OPD ═══════════════════════════════════ -->
      <div v-if="showMasterModal" class="er-overlay" @click.self="showMasterModal = false" @keydown.esc="showMasterModal = false">
        <div class="er-modal" role="dialog" aria-modal="true" aria-labelledby="er-opd-title">
          <h4 id="er-opd-title" class="er-h">Tambah Master OPD</h4>
          <label class="er-field">
            <span class="form-label-sm">Kode lampiran</span>
            <input ref="opdKodeInput" v-model.trim="opdForm.kode" class="form-input" type="text" placeholder="Contoh: LAMPIRAN LIV" />
          </label>
          <label class="er-field">
            <span class="form-label-sm">Nama perangkat daerah</span>
            <input v-model.trim="opdForm.nama" class="form-input" type="text" placeholder="Contoh: Kecamatan Sedong" />
          </label>
          <label class="er-field">
            <span class="form-label-sm">Kategori</span>
            <select v-model="opdForm.kategori" class="form-input">
              <option>Dinas / Badan Utama</option>
              <option>Kecamatan</option>
            </select>
          </label>
          <div class="er-form-actions">
            <button type="button" class="btn btn-secondary btn-sm" @click="showMasterModal = false">Batal</button>
            <button type="button" class="btn btn-primary btn-sm" @click="saveMasterOpd">Simpan OPD</button>
          </div>
        </div>
      </div>
    </Teleport>
  </div>
</template>

<script setup>
import { ref, reactive, computed, nextTick } from 'vue';
import { useAnalysis } from '../composables/useAnalysis';

const { showNotification } = useAnalysis();

/* ── Navigasi & mode ─────────────────────────────────────────────────── */
const tabs = [
  { id: 'dashboard', label: 'Dashboard', icon: 'fa-chart-line' },
  { id: 'editor', label: 'Smart Editor F4', icon: 'fa-file-signature' },
  { id: 'formatter', label: 'Mesin Cuci Dokumen', icon: 'fa-soap' },
  { id: 'templates', label: 'Smart Template Manager', icon: 'fa-cubes' },
  { id: 'review', label: 'Review & Verifikasi', icon: 'fa-user-check' },
  { id: 'monitoring', label: 'Monitoring 71 OPD', icon: 'fa-desktop' },
  { id: 'master', label: 'Master OPD & Nomenklatur', icon: 'fa-database' }
];

const currentTab = ref('review');
const userRole = ref('bapperida');
const activeRibbon = ref('utama');

const roleLabels = {
  admin: 'Mode Super Admin',
  bapperida: 'Mode Verifikator Bapperida',
  opd: 'Mode Operator OPD'
};
const roleLabel = computed(() => roleLabels[userRole.value]);

function onRoleChange() {
  currentTab.value = { admin: 'master', bapperida: 'review', opd: 'editor' }[userRole.value];
}

// Prototipe ini memakai data contoh; aksi yang belum tersambung ke mesin dokumen
// ditandai "Simulasi" agar tidak dikira sudah benar-benar memproses berkas.
function simulate(title, message) {
  showNotification(`Simulasi: ${title}`, message, 'info');
}

/* ── Dashboard ───────────────────────────────────────────────────────── */
const opdTotal = 71;
const opdSelesai = 58;
const opdReview = 9;
const opdDraft = 4;

const metrics = [
  { label: 'Total OPD terdaftar', value: opdTotal, unit: 'Perangkat Daerah', note: '31 Dinas/Badan dan 40 Kecamatan', tone: '' },
  { label: 'Dokumen disetujui', value: opdSelesai, unit: 'dokumen', note: 'Telah diverifikasi Bapperida', tone: 'tone-ok' },
  { label: 'Dalam review Bapperida', value: opdReview, unit: 'dokumen', note: 'Sedang ditelaah tim perencana', tone: 'tone-warn' },
  { label: 'Draft / dalam proses', value: opdDraft, unit: 'dokumen', note: 'Sedang diisi oleh OPD', tone: 'tone-info' }
];

const statusBadge = {
  approved: { label: 'Disetujui', cls: 'badge-success' },
  review: { label: 'Review Bapperida', cls: 'badge-warning' },
  revision: { label: 'Perlu revisi OPD', cls: 'badge-danger' },
  draft: { label: 'Draft', cls: 'badge-secondary' },
  pending: { label: 'Menunggu verifikasi', cls: 'badge-warning' }
};

const documents = [
  { lampiran: 'LAMPIRAN LIII', opd: 'Kecamatan Losari', jenis: 'RENJA', status: 'approved', aksi: 'Buka Editor F4', tab: 'editor' },
  { lampiran: 'LAMPIRAN IV', opd: 'Dinas Pendidikan', jenis: 'RENJA', status: 'review', aksi: 'Review Catatan', tab: 'review', doc: 'Dinas Pendidikan - Renja 2027' },
  { lampiran: 'LAMPIRAN V', opd: 'Dinas Kesehatan', jenis: 'RKPD', status: 'draft', aksi: 'Verifikasi RKPD', tab: 'review', doc: 'Dinas Kesehatan - Dokumen RKPD 2027' }
];

function openDocument(d) {
  if (d.doc) selectedReviewDoc.value = d.doc;
  currentTab.value = d.tab;
}

/* ── Smart Editor F4 ─────────────────────────────────────────────────── */
let rowSeq = 0;
const newRow = (over = {}) => ({ id: ++rowSeq, kode: '', indikator: '', target: '', realisasi: '', ...over });

const rows = ref([
  newRow({ kode: '7.01.01.2.01.01', indikator: 'Jumlah Layanan Administrasi Perangkat Daerah', target: '100%', realisasi: '100%' }),
  newRow({ kode: '7.01.01.2.01.06', indikator: 'Cakupan Pembinaan Ketentraman & Ketertiban Umum', target: '12 Bulan', realisasi: '100%' })
]);

function addRow(where) {
  if (where === 'top') rows.value.unshift(newRow());
  else rows.value.push(newRow());
}

const alignCell = ref(5);

/* ── Mesin Cuci Dokumen ──────────────────────────────────────────────── */
const opdOptions = [
  'Kecamatan Losari (LAMPIRAN LIII)',
  'Dinas Pendidikan (LAMPIRAN IV)',
  'Dinas Kesehatan (LAMPIRAN V)'
];
const selectedOpd = ref(opdOptions[0]);
const selectedTemplate = ref('RENJA');
const selectedYear = ref('2027');

const baseTemplateOptions = [
  { kode: 'RENJA', label: 'RENJA (Rencana Kerja Perangkat Daerah)' },
  { kode: 'RKPD', label: 'RKPD (Rencana Kerja Pemerintah Daerah)' },
  { kode: 'EVALUASI_RKPD', label: 'EVALUASI RKPD' },
  { kode: 'KUA_PPAS', label: 'KUA-PPAS' }
];

const fileInput = ref(null);
const pickedFile = ref(null);
const dragOver = ref(false);

function setFile(file) {
  if (!file) return;
  if (!/\.docx$/i.test(file.name)) {
    showNotification('Format tidak didukung', 'Pilih berkas MS Word dengan ekstensi .docx.', 'warning');
    return;
  }
  pickedFile.value = file;
}
function onPickFile(e) {
  setFile(e.target.files?.[0]);
}
function onDrop(e) {
  dragOver.value = false;
  setFile(e.dataTransfer?.files?.[0]);
}
function clearFile() {
  pickedFile.value = null;
  if (fileInput.value) fileInput.value.value = '';
}
function formatSize(bytes) {
  if (bytes < 1024) return `${bytes} B`;
  if (bytes < 1024 * 1024) return `${(bytes / 1024).toFixed(1)} KB`;
  return `${(bytes / 1024 / 1024).toFixed(1)} MB`;
}
function washDocument() {
  simulate('Cuci dokumen', `${pickedFile.value.name} dirapikan ke F4, margin 2 cm, Bookman 12 pt, tanpa cetak tebal (aturan ${selectedTemplate.value}).`);
}

/* ── Smart Template Manager ──────────────────────────────────────────── */
const templates = ref([
  { kode: 'RENJA', nama: 'Rencana Kerja Perangkat Daerah (Renja)', deskripsi: 'Khusus Renja: tanpa cover, kata pengantar, dan daftar isi; header Perbup 5 baris di halaman 1; tanpa cetak tebal.' },
  { kode: 'RKPD', nama: 'Rencana Kerja Pemerintah Daerah (RKPD)', deskripsi: 'Format kabupaten: memiliki cover, kata pengantar, dan daftar isi otomatis.' }
]);

const templateOptions = computed(() => {
  const known = new Set(baseTemplateOptions.map(t => t.kode));
  const extra = templates.value
    .filter(t => !known.has(t.kode))
    .map(t => ({ kode: t.kode, label: `${t.kode} (${t.nama})` }));
  return [...baseTemplateOptions, ...extra];
});

const showTemplateModal = ref(false);
const tplForm = reactive({ kode: '', nama: '' });
const tplKodeInput = ref(null);

function openTemplateModal() {
  tplForm.kode = '';
  tplForm.nama = '';
  showTemplateModal.value = true;
  nextTick(() => tplKodeInput.value?.focus());
}
function saveTemplate() {
  const kode = tplForm.kode.toUpperCase().replace(/\s+/g, '_');
  if (!kode || !tplForm.nama) {
    showNotification('Data belum lengkap', 'Isi kode dan nama template terlebih dahulu.', 'warning');
    return;
  }
  if (templates.value.some(t => t.kode === kode)) {
    showNotification('Kode sudah dipakai', `Template ${kode} sudah terdaftar.`, 'warning');
    return;
  }
  templates.value.push({ kode, nama: tplForm.nama, deskripsi: 'Template manual. Aturan struktur belum dikonfigurasi.' });
  showTemplateModal.value = false;
  showNotification('Template ditambahkan', `Template ${kode} berhasil didaftarkan.`, 'success');
}

/* ── Review & Verifikasi ─────────────────────────────────────────────── */
const reviewDocs = [
  'Dinas Kesehatan - Dokumen RKPD 2027',
  'Dinas Pendidikan - Renja 2027',
  'BKAD - Dokumen KUA-PPAS 2027',
  'Bapperida - Evaluasi RKPD Triwulan IV',
  'Dokumen Acuan Skema RPJMD/LKPJ'
];
const selectedReviewDoc = ref(reviewDocs[0]);
const reviewStatus = reactive({});
const reviewNotes = reactive({});

const reviewStatusView = computed(() => statusBadge[reviewStatus[selectedReviewDoc.value] || 'pending']);

function decide(decision) {
  reviewStatus[selectedReviewDoc.value] = decision;
  if (decision === 'approved') {
    showNotification('Dokumen disetujui', `${selectedReviewDoc.value} disetujui oleh Tim Bapperida.`, 'success');
  } else {
    showNotification('Revisi diminta', `${selectedReviewDoc.value} dikembalikan ke OPD untuk direvisi.`, 'warning');
  }
}

/* ── Monitoring ──────────────────────────────────────────────────────── */
const progressPct = computed(() => Math.round((opdSelesai / opdTotal) * 1000) / 10);

/* ── Master OPD ──────────────────────────────────────────────────────── */
const masterOpd = ref([
  { kode: 'LAMPIRAN I', nama: 'Sekretariat Daerah', kategori: 'Dinas / Badan Utama' },
  { kode: 'LAMPIRAN LIII', nama: 'Kecamatan Losari', kategori: 'Kecamatan' },
  { kode: 'LAMPIRAN LXXI', nama: 'Kecamatan Weru', kategori: 'Kecamatan' }
]);

const showMasterModal = ref(false);
const opdForm = reactive({ kode: '', nama: '', kategori: 'Kecamatan' });
const opdKodeInput = ref(null);

function openMasterModal() {
  opdForm.kode = '';
  opdForm.nama = '';
  opdForm.kategori = 'Kecamatan';
  showMasterModal.value = true;
  nextTick(() => opdKodeInput.value?.focus());
}
function saveMasterOpd() {
  const kode = opdForm.kode.toUpperCase();
  if (!kode || !opdForm.nama) {
    showNotification('Data belum lengkap', 'Isi kode lampiran dan nama perangkat daerah.', 'warning');
    return;
  }
  masterOpd.value.push({ kode, nama: opdForm.nama, kategori: opdForm.kategori });
  showMasterModal.value = false;
  showNotification('OPD ditambahkan', `${opdForm.nama} (${kode}) masuk ke master data.`, 'success');
}
</script>

<style scoped>
/* Semua warna memakai token tema Sintra (assets/style.css), jadi terang/gelap ikut otomatis.
   Pengecualian: kertas F4 sengaja putih permanen karena mewakili hasil cetak. */

.erenja {
  display: flex;
  flex-direction: column;
  gap: 16px;
  min-width: 0;
  color: var(--text-primary);
}

/* ── Toolbar ─────────────────────────────────────────────────────────── */
.er-toolbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  flex-wrap: wrap;
}
.er-brand { display: flex; align-items: center; gap: 12px; min-width: 0; }
.er-brand-icon {
  width: 40px;
  height: 40px;
  flex-shrink: 0;
  border-radius: var(--border-radius-md);
  background: var(--primary-color);
  color: #fff;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.05rem;
}
.er-brand-title {
  font-family: var(--font-heading);
  font-size: 1rem;
  font-weight: 700;
  display: flex;
  align-items: center;
  gap: 10px;
  flex-wrap: wrap;
}
.er-brand-sub { font-size: 0.75rem; color: var(--text-secondary); margin-top: 2px; }
.er-role { display: flex; align-items: center; gap: 10px; font-size: 0.75rem; font-weight: 700; color: var(--text-secondary); }
.er-role select { width: auto; min-width: 260px; }

/* ── Tab modul ───────────────────────────────────────────────────────── */
.er-tabs {
  display: flex;
  gap: 6px;
  overflow-x: auto;
  padding-bottom: 4px;
  border-bottom: 1px solid var(--border-color);
  scrollbar-width: thin;
}
.er-tab {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  flex-shrink: 0;
  padding: 9px 14px;
  border: 1px solid transparent;
  border-radius: var(--border-radius-sm);
  background: transparent;
  color: var(--text-secondary);
  font-family: var(--font-body);
  font-size: 0.8rem;
  font-weight: 600;
  cursor: pointer;
  transition: background-color var(--transition-speed), color var(--transition-speed);
}
.er-tab:hover { background: var(--bg-tertiary); color: var(--text-primary); }
.er-tab.active { background: var(--primary-color); color: #fff; }
.er-tab:focus-visible,
.er-ribbon-tab:focus-visible,
.er-align-cell:focus-visible,
.er-drop:focus-within {
  outline: 2px solid var(--primary-color);
  outline-offset: 2px;
}

/* ── Panel & judul ───────────────────────────────────────────────────── */
.er-panel { display: flex; flex-direction: column; gap: 16px; min-width: 0; }
.er-panel-head { display: flex; align-items: flex-end; justify-content: space-between; gap: 16px; flex-wrap: wrap; }
.er-h { font-family: var(--font-heading); font-size: 1.15rem; font-weight: 700; color: var(--text-primary); }
.er-h5 { font-size: 0.85rem; font-weight: 700; margin-bottom: 10px; }
.er-sub { font-size: 0.78rem; color: var(--text-secondary); line-height: 1.45; }
.er-sub-wide { max-width: 640px; }
.er-strong { font-weight: 700; }
.er-accent { color: var(--primary-color); font-weight: 700; }
.er-code { font-weight: 700; color: var(--accent-hover); white-space: nowrap; }

/* ── Metrik ──────────────────────────────────────────────────────────── */
.er-metrics { display: grid; grid-template-columns: repeat(4, minmax(0, 1fr)); gap: 16px; }
.er-metric {
  background: var(--bg-secondary);
  border: 1px solid var(--border-color);
  border-radius: var(--card-border-radius);
  box-shadow: var(--card-shadow);
  padding: 18px 20px;
  display: flex;
  flex-direction: column;
  gap: 6px;
}
.er-metric-label { font-size: 0.78rem; font-weight: 600; color: var(--text-secondary); }
.er-metric-value { font-family: var(--font-heading); font-size: 1.9rem; font-weight: 800; line-height: 1.1; }
.er-metric-value small { font-family: var(--font-body); font-size: 0.75rem; font-weight: 500; color: var(--text-secondary); }
.er-metric-value.tone-ok { color: var(--success-hover); }
.er-metric-value.tone-warn { color: var(--warning-hover); }
.er-metric-value.tone-info { color: var(--info-hover); }
.er-metric-note { font-size: 0.72rem; color: var(--text-muted); }

/* ── Tabel ───────────────────────────────────────────────────────────── */
.er-table-scroll { overflow-x: auto; }
.er-table { width: 100%; border-collapse: collapse; font-size: 0.82rem; }
.er-table th {
  text-align: left;
  padding: 12px 16px;
  font-size: 0.75rem;
  font-weight: 700;
  color: var(--text-secondary);
  background: var(--bg-tertiary);
  border-bottom: 1px solid var(--border-color);
  white-space: nowrap;
}
.er-table td { padding: 12px 16px; border-bottom: 1px solid var(--border-color); vertical-align: middle; }
.er-table tbody tr:last-child td { border-bottom: none; }
.er-table tbody tr:hover { background: var(--bg-tertiary); }

.er-tag {
  display: inline-block;
  padding: 3px 10px;
  border-radius: 6px;
  font-size: 0.72rem;
  font-weight: 700;
}
.er-tag-renja { background: var(--primary-glow); color: var(--primary-color); }
.er-tag-rkpd { background: var(--accent-glow); color: var(--accent-hover); }

/* ── Ribbon editor ───────────────────────────────────────────────────── */
.er-ribbon { background: var(--bg-tertiary); border-bottom: 1px solid var(--border-color); padding: 10px 20px 14px; }
.er-ribbon-tabs { display: flex; gap: 18px; border-bottom: 1px solid var(--border-color); margin-bottom: 12px; }
.er-ribbon-tab {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 4px 0 8px;
  background: none;
  border: none;
  border-bottom: 2px solid transparent;
  color: var(--text-secondary);
  font-family: var(--font-body);
  font-size: 0.8rem;
  font-weight: 600;
  cursor: pointer;
}
.er-ribbon-tab:hover { color: var(--text-primary); }
.er-ribbon-tab.active { color: var(--primary-color); border-bottom-color: var(--primary-color); }
.er-ribbon-body { display: flex; align-items: center; gap: 12px 24px; flex-wrap: wrap; }
.er-ribbon-group { display: flex; align-items: center; gap: 8px; flex-wrap: wrap; }
.er-ribbon-label { font-size: 0.72rem; font-weight: 700; color: var(--text-secondary); }
.er-chip {
  padding: 5px 10px;
  border-radius: var(--border-radius-sm);
  border: 1px solid var(--border-color-strong);
  background: var(--bg-secondary);
  font-size: 0.75rem;
  font-weight: 600;
  color: var(--text-primary);
}
.er-chip-on { color: var(--primary-color); border-color: var(--primary-color); }
.er-chip-danger { color: var(--danger-hover); background: var(--danger-glow); border-color: transparent; }

.er-align-grid {
  display: grid;
  grid-template-columns: repeat(3, 12px);
  gap: 3px;
  padding: 4px;
  background: var(--bg-secondary);
  border: 1px solid var(--border-color-strong);
  border-radius: 6px;
}
.er-align-cell { width: 12px; height: 12px; padding: 0; border: none; border-radius: 2px; background: var(--border-color-strong); cursor: pointer; }
.er-align-cell:hover { background: var(--info-color); }
.er-align-cell.active { background: var(--primary-color); }

/* ── Kanvas & kertas F4 ──────────────────────────────────────────────── */
.er-canvas { background: var(--bg-primary); padding: 28px 20px; overflow: auto; max-height: 78vh; }
.er-paper {
  width: 215mm;
  min-height: 330mm;
  margin: 0 auto;
  padding: 20mm;
  background: #ffffff;
  color: #1a1d21;
  box-shadow: 0 12px 30px -10px rgba(17, 24, 39, 0.35);
  font-family: 'Bookman Old Style', 'Bookman', 'URW Bookman', Georgia, serif;
  font-size: 12pt;
  line-height: 1.5;
  display: flex;
  flex-direction: column;
  gap: 18px;
}
.er-perbup { width: 50%; margin-left: auto; font-size: 11pt; line-height: 1.35; padding-bottom: 14px; border-bottom: 1px solid #d3d7dc; }
.er-chapter { text-align: center; font-weight: 700; padding-top: 6px; }
.er-para { display: flex; flex-direction: column; gap: 8px; }
.er-para p { text-align: justify; }
.er-para-title { font-weight: 600; }
.er-paper-table { width: 100%; border-collapse: collapse; font-size: 10pt; }
.er-paper-table th,
.er-paper-table td { border: 1px solid #1a1d21; padding: 6px 8px; vertical-align: top; }
.er-paper-table th { background: #f1f3f5; text-align: center; font-weight: 600; }
.er-paper-table td.c { text-align: center; }
.er-paper-table td[contenteditable]:focus { outline: 2px solid #1b4d46; outline-offset: -2px; background: #f7fbfa; }

/* ── Form, dropzone, aksi ────────────────────────────────────────────── */
.er-form-body { display: flex; flex-direction: column; gap: 20px; padding: 24px; }
.er-form-grid { display: grid; grid-template-columns: repeat(3, minmax(0, 1fr)); gap: 16px; }
.er-field { display: flex; flex-direction: column; gap: 6px; min-width: 0; }
.er-textarea { resize: vertical; min-height: 84px; }
.er-file { position: absolute; width: 1px; height: 1px; opacity: 0; pointer-events: none; }
.er-drop {
  position: relative;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  padding: 36px 20px;
  text-align: center;
  border: 2px dashed var(--border-color-strong);
  border-radius: var(--card-border-radius);
  background: var(--bg-tertiary);
  cursor: pointer;
  transition: border-color var(--transition-speed), background-color var(--transition-speed);
}
.er-drop:hover,
.er-drop.over { border-color: var(--primary-color); background: var(--primary-glow); }
.er-drop.filled { border-style: solid; border-color: var(--primary-color); }
.er-drop-icon {
  width: 52px;
  height: 52px;
  border-radius: var(--border-radius-md);
  background: var(--primary-color);
  color: #fff;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.3rem;
}
.er-drop-title { font-size: 0.95rem; font-weight: 700; word-break: break-all; }
.er-form-actions { display: flex; justify-content: flex-end; gap: 10px; flex-wrap: wrap; }

/* ── Kartu template ──────────────────────────────────────────────────── */
.er-cards { display: grid; grid-template-columns: repeat(auto-fill, minmax(320px, 1fr)); gap: 16px; }
.er-card {
  background: var(--bg-secondary);
  border: 1px solid var(--border-color);
  border-radius: var(--card-border-radius);
  box-shadow: var(--card-shadow);
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}
.er-card-top { display: flex; align-items: center; justify-content: space-between; }
.er-card-title { font-family: var(--font-heading); font-size: 1rem; font-weight: 700; }
.er-card-text { font-size: 0.8rem; color: var(--text-secondary); line-height: 1.5; }

/* ── Review ──────────────────────────────────────────────────────────── */
.er-review-head { align-items: flex-end; gap: 16px; flex-wrap: wrap; }
.er-review-select { min-width: 280px; }
.er-review-body { display: flex; flex-direction: column; gap: 20px; padding: 24px; }
.er-meta { display: grid; grid-template-columns: repeat(3, minmax(0, 1fr)); gap: 12px; }
.er-meta-item {
  background: var(--bg-tertiary);
  border: 1px solid var(--border-color);
  border-radius: var(--border-radius-md);
  padding: 12px 14px;
  display: flex;
  flex-direction: column;
  gap: 6px;
  font-size: 0.82rem;
}
.er-meta-label { font-size: 0.72rem; font-weight: 700; color: var(--text-secondary); }
.er-checks { display: grid; grid-template-columns: repeat(2, minmax(0, 1fr)); gap: 10px; }
.er-check {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  padding: 11px 14px;
  border: 1px solid var(--border-color);
  border-radius: var(--border-radius-md);
  font-size: 0.8rem;
}
.er-ok { color: var(--success-hover); font-weight: 700; white-space: nowrap; }
.er-decision { padding-top: 16px; border-top: 1px solid var(--border-color); }

/* ── Progres ─────────────────────────────────────────────────────────── */
.er-progress-body { display: flex; flex-direction: column; gap: 12px; padding: 24px; }
.er-progress-meta { display: flex; justify-content: space-between; gap: 12px; flex-wrap: wrap; font-size: 0.85rem; font-weight: 700; }
.er-progress { height: 12px; border-radius: 999px; background: var(--bg-tertiary); box-shadow: var(--shadow-inset); overflow: hidden; }
.er-progress-fill { height: 100%; background: var(--primary-color); border-radius: 999px; }

/* ── Modal ───────────────────────────────────────────────────────────── */
.er-overlay {
  position: fixed;
  inset: 0;
  z-index: 10000;
  background: rgba(0, 0, 0, 0.55);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 16px;
}
.er-modal {
  width: 100%;
  max-width: 440px;
  background: var(--bg-secondary);
  color: var(--text-primary);
  border: 1px solid var(--border-color);
  border-radius: var(--card-border-radius);
  box-shadow: var(--shadow-raised-hover);
  padding: 24px;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

/* ── Responsif ───────────────────────────────────────────────────────── */
@media (max-width: 1100px) {
  .er-metrics { grid-template-columns: repeat(2, minmax(0, 1fr)); }
  .er-meta { grid-template-columns: 1fr; }
}
@media (max-width: 720px) {
  .er-form-grid,
  .er-checks,
  .er-metrics { grid-template-columns: 1fr; }
  .er-role { width: 100%; flex-direction: column; align-items: stretch; }
  .er-role select { min-width: 0; width: 100%; }
  .er-review-select { min-width: 0; width: 100%; }
}
</style>
