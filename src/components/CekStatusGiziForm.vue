<script setup>
import { ref, computed } from 'vue'

const form = ref({
  nama: '',
  jk: 'Laki-laki',
  tglLahir: '',
  tglTimbang: new Date().toISOString().slice(0,10),
  berat: '',
  tinggi: '',
  posisi: 'Berbaring',
})

const ageMonths = computed(() => {
  if (!form.value.tglLahir || !form.value.tglTimbang) return null
  const b = new Date(form.value.tglLahir)
  const w = new Date(form.value.tglTimbang)
  if (isNaN(b) || isNaN(w) || w < b) return null
  let m = (w.getFullYear()-b.getFullYear())*12 + (w.getMonth()-b.getMonth())
  if (w.getDate() < b.getDate()) m -= 1
  return m
})

const result = ref(null)
const errorMsg = ref('')

function validate() {
  errorMsg.value = ''
  if (!form.value.nama.trim()) return (errorMsg.value='Nama wajib diisi'), false
  if (ageMonths.value==null) return (errorMsg.value='Tanggal lahir/penimbangan tidak valid'), false
  if (ageMonths.value < 0 || ageMonths.value > 59) return (errorMsg.value='Hanya untuk usia ≤ 59 bulan'), false
  const bb = Number(form.value.berat), tb = Number(form.value.tinggi)
  if (!bb || bb<=0) return (errorMsg.value='Berat badan tidak valid'), false
  if (!tb || tb<=0) return (errorMsg.value='Tinggi/Panjang badan tidak valid'), false
  return true
}

// DEMO kalkulasi (placeholder) – ganti dengan Z-Score WHO untuk produksi
function classifyDemo() {
  const bb = Number(form.value.berat)
  const tb = Number(form.value.tinggi)
  const bmi = bb / Math.pow(tb/100, 2)

  let kategori = 'Normal', badge = 'ok'
  if (bmi < 13.5) { kategori='Gizi Kurang (demo)'; badge='warn' }
  if (bmi < 12.8) { kategori='Gizi Buruk (demo)'; badge='danger' }
  if (bmi >= 17)  { kategori='Risiko Gemuk (demo)'; badge='warn' }
  if (bmi >= 18.5){ kategori='Obesitas (demo)'; badge='danger' }

  return { bmi: Number(bmi.toFixed(2)), kategori, badge }
}

function lihatHasil() {
  if (!validate()) { result.value=null; return }
  const demo = classifyDemo()
  result.value = {
    nama: form.value.nama.trim(),
    jk: form.value.jk,
    umurBulan: ageMonths.value,
    tanggal: form.value.tglTimbang,
    berat: Number(form.value.berat),
    tinggi: Number(form.value.tinggi),
    posisi: form.value.posisi,
    bmi: demo.bmi,
    kategori: demo.kategori,
    badge: demo.badge
  }
}
</script>

<template>
  <div class="card">
    <div class="card-head">Cek Status Gizi</div>
    <div class="card-body">
      <div class="grid">
        <div class="col-6">
          <label class="label">Nama Anak</label>
          <input class="input" v-model="form.nama" placeholder="Nama Anak" />
        </div>

        <div class="col-6">
          <label class="label">Jenis Kelamin</label>
          <select class="select" v-model="form.jk">
            <option>Laki-laki</option>
            <option>Perempuan</option>
          </select>
        </div>

        <div class="col-6">
          <label class="label">Tanggal Lahir</label>
          <input type="date" class="input" v-model="form.tglLahir" />
        </div>

        <div class="col-6">
          <label class="label">Tanggal Penimbangan</label>
          <input type="date" class="input" v-model="form.tglTimbang" />
        </div>

        <div class="col-6">
          <label class="label">Berat Badan</label>
          <div class="input-group">
            <input class="input" type="number" min="0" step="0.01" placeholder="Berat Badan" v-model="form.berat" />
            <div class="addon">KG</div>
          </div>
        </div>

        <div class="col-6">
          <label class="label">Panjang/Tinggi Badan</label>
          <div class="input-group">
            <input class="input" type="number" min="0" step="0.1" placeholder="Panjang/Tinggi Badan" v-model="form.tinggi" />
            <div class="addon">CM</div>
          </div>
        </div>

        <div class="col-6">
          <label class="label">Posisi</label>
          <select class="select" v-model="form.posisi">
            <option>Berbaring</option>
            <option>Berdiri</option>
          </select>
          <small class="helper">Umumnya: &lt; 24 bln berbaring; ≥ 24 bln berdiri.</small>
        </div>

        <div class="col-12" v-if="ageMonths!==null">
          <small class="helper">Umur per penimbangan: <b>{{ ageMonths }}</b> bulan</small>
        </div>

        <div class="col-12" v-if="errorMsg">
          <div class="result"><span class="badge danger">Error</span> {{ errorMsg }}</div>
        </div>

        <div class="col-12">
          <button class="btn" @click="lihatHasil">LIHAT HASIL</button>
        </div>

        <div class="col-12">
          <div class="divider"></div>
        </div>

        <div class="col-12">
          <button class="google" title="Demo only">
            <img width="18" height="18" alt="" src="https://www.gstatic.com/firebasejs/ui/2.0.0/images/auth/google.svg" />
            Masuk dengan Google
          </button>
          <div class="helper">Login untuk melihat & menyimpan data penimbangan (demo)</div>
        </div>

        <div class="col-12" v-if="result">
          <div class="result">
            <div style="display:flex;justify-content:space-between;align-items:center;gap:8px;flex-wrap:wrap;">
              <div><b>Hasil</b></div>
              <div class="badge" :class="result.badge">{{ result.kategori }}</div>
            </div>
            <div style="margin-top:6px">
              <div><b>Nama:</b> {{ result.nama }} ({{ result.jk }})</div>
              <div><b>Tanggal Penimbangan:</b> {{ result.tanggal }}</div>
              <div><b>Umur:</b> {{ result.umurBulan }} bulan</div>
              <div><b>Berat/Tinggi:</b> {{ result.berat }} kg / {{ result.tinggi }} cm</div>
              <div><b>BMI (demo):</b> {{ result.bmi }}</div>
            </div>
            <div class="helper" style="margin-top:8px">
              Kalkulasi <b>DEMO</b>. Untuk akurasi, gunakan Z-Score WHO (WFA/HFA/WFH/BMI).
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
