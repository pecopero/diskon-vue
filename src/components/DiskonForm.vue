<template>
  <div class="modal-overlay" @click.self="$emit('close')">
    <div class="modal">
      <div class="modal-header">
        <div class="modal-title">{{ mode === 'add' ? 'Tambah Diskon' : 'Ubah Diskon' }}</div>
        <button class="icon-btn" @click="$emit('close')">
          <span class="material-icons-round">close</span>
        </button>
      </div>

      <!-- Nama Diskon -->
      <div class="field-group">
        <div class="field-wrap" :class="{ error: errors.nama }">
          <span class="field-label">{{ mode === 'edit' ? originalName : 'Nama Diskon' }}</span>
          <input
            class="field-input"
            v-model="form.nama_diskon"
            placeholder="Misal: Diskon opening, diskon akhir tahun"
            style="padding-right: 40px"
            @input="errors.nama = ''"
          />
          <div v-if="errors.nama" class="error-icon">i</div>
        </div>
        <div v-if="errors.nama" class="field-error">{{ errors.nama }}</div>
      </div>

      <!-- Nilai Diskon -->
      <div class="field-group">
        <div class="diskon-row">
          <div class="field-wrap" :class="{ error: errors.nilai }" style="flex:1">
            <span class="field-label">Diskon</span>
            <div class="diskon-input-inner">
              <span v-if="form.tipe === 'rp'" class="unit-prefix">Rp</span>
              <input
                class="field-input"
                type="number"
                min="0"
                v-model.number="form.nilai"
                placeholder="0"
                :style="form.tipe === 'rp' ? 'padding-left: 4px' : ''"
                @input="errors.nilai = ''"
              />
              <span v-if="form.tipe === 'persen'" class="unit-suffix">%</span>
            </div>
          </div>
          <div class="type-toggle">
            <button
              class="type-btn"
              :class="{ active: form.tipe === 'persen' }"
              @click="form.tipe = 'persen'"
            >
              <span class="material-icons-round" v-if="form.tipe === 'persen'">check</span>
              %
            </button>
            <button
              class="type-btn"
              :class="{ active: form.tipe === 'rp' }"
              @click="form.tipe = 'rp'"
            >
              <span class="material-icons-round" v-if="form.tipe === 'rp'">check</span>
              Rp
            </button>
          </div>
        </div>
        <div v-if="errors.nilai" class="field-error">{{ errors.nilai }}</div>
      </div>

      <!-- Footer -->
      <div class="modal-footer" :class="mode === 'edit' ? 'modal-footer-space' : 'modal-footer-end'">
        <button v-if="mode === 'edit'" class="btn btn-danger-outline" @click="$emit('delete')">Hapus</button>
        <button class="btn btn-primary" @click="save" :disabled="saving">
          {{ saving ? 'Menyimpan...' : 'Simpan' }}
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { reactive, ref } from 'vue'

const props = defineProps({
  mode: { type: String, default: 'add' }, // 'add' | 'edit'
  initialData: { type: Object, default: null },
  existingNames: { type: Array, default: () => [] },
  saving: { type: Boolean, default: false },
})

const emit = defineEmits(['close', 'save', 'delete'])

const originalName = props.initialData?.nama_diskon || ''

const form = reactive({
  nama_diskon: props.initialData?.nama_diskon || '',
  nilai: props.initialData?.nilai ?? '',
  tipe: props.initialData?.tipe || 'persen',
})

const errors = reactive({ nama: '', nilai: '' })

function save() {
  errors.nama = ''
  errors.nilai = ''
  let ok = true

  const nama = form.nama_diskon.trim()
  if (!nama) {
    errors.nama = 'Nama diskon tidak boleh kosong.'
    ok = false
  } else {
    const dup = props.existingNames.find(
      n => n.toLowerCase() === nama.toLowerCase() && n !== originalName
    )
    if (dup) {
      errors.nama = 'Nama diskon sudah digunakan, silahkan gunakan nama lain.'
      ok = false
    }
  }

  if (form.nilai === '' || form.nilai === null) {
    errors.nilai = 'Diskon tidak boleh kosong.'
    ok = false
  }

  if (ok) emit('save', { nama_diskon: nama, nilai: form.nilai, tipe: form.tipe })
}
</script>

<style scoped>
.diskon-row { display: flex; gap: 8px; align-items: flex-start; }
.diskon-input-inner { display: flex; align-items: center; }
.unit-prefix {
  color: var(--md-on-surface-variant); font-size: 14px;
  padding-left: 12px; padding-top: 14px; padding-bottom: 14px; align-self: center; white-space: nowrap;
}
.unit-suffix {
  color: var(--md-on-surface-variant); font-size: 14px; padding-right: 12px; white-space: nowrap;
}
.type-toggle {
  display: flex; border: 1.5px solid var(--md-outline);
  border-radius: var(--radius-xl); overflow: hidden; height: 50px; flex-shrink: 0;
}
.type-btn {
  padding: 0 14px; font-size: 14px; font-weight: 500;
  font-family: 'Google Sans', 'Roboto', sans-serif;
  cursor: pointer; border: none; background: transparent;
  color: var(--md-on-surface-variant); display: flex; align-items: center; gap: 4px;
  transition: all var(--transition);
}
.type-btn .material-icons-round { font-size: 14px; }
.type-btn.active { background: var(--md-primary-container); color: var(--md-primary); }
</style>
