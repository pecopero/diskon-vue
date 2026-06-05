<template>
  <div id="app-wrapper">
    <Transition name="fade">
      <div v-if="toast.show" class="toast-container">
        <div class="toast">{{ toast.message }}</div>
      </div>
    </Transition>

    <div class="page-content">
      <div class="page-header">
        <div>
          <div class="page-title">Daftar Diskon</div>
          <div v-if="discounts.length > 0" class="page-subtitle">Total jumlah diskon: {{ discounts.length }}</div>
        </div>
        <div class="header-actions">
          <template v-if="selectedIds.length > 0">
            <button class="btn btn-outline btn-sm" @click="clearSelection">Batalkan</button>
            <button class="btn btn-danger btn-sm" @click="openBulkDelete">
              <span class="material-icons-round">delete</span> Hapus
            </button>
          </template>
          <template v-else-if="discounts.length > 0">
            <button class="btn btn-primary btn-sm" @click="openAdd">
              <span class="material-icons-round">add</span> Tambah diskon
            </button>
          </template>
        </div>
      </div>

      <div v-if="apiUrl" class="toolbar">
        <div class="search-field">
          <span class="material-icons-round">search</span>
          <input v-model="searchQuery" placeholder="Cari diskon" />
        </div>
        <div class="outlet-chip" :class="{ open: outletOpen }" @click="outletOpen = !outletOpen">
          <span class="material-icons-round">storefront</span>
          <span class="outlet-name">{{ outletLabel }}</span>
          <span class="material-icons-round chevron">expand_more</span>
          <Transition name="fade">
            <div v-if="outletOpen" class="outlet-panel" @click.stop>
              <div class="field-group" style="margin-bottom:12px">
                <div class="field-wrap" :class="{ error: urlError }">
                  <span class="field-label">API URL crudcrud.com</span>
                  <input class="field-input" v-model="apiUrlInput" placeholder="https://crudcrud.com/api/bc0c.../diskon" />
                </div>
                <div v-if="urlError" class="field-error">{{ urlError }}</div>
              </div>
              <button class="btn btn-primary btn-sm" @click="applyApiUrl">Terapkan</button>
            </div>
          </Transition>
        </div>
      </div>

      <div class="card">
        <div v-if="!apiUrl" class="empty-state">
          <span class="material-icons-round" style="font-size:56px;color:var(--md-outline-variant)">link_off</span>
          <div class="empty-title">Atur API URL Terlebih Dahulu</div>
          <div class="empty-desc">Masukkan endpoint crudcrud.com Anda untuk mulai mengelola diskon.</div>
          <div class="inline-setup">
            <div class="field-wrap" :class="{ error: urlError }" style="width:100%">
              <span class="field-label">API URL crudcrud.com</span>
              <input class="field-input" v-model="apiUrlInput"
                placeholder="https://crudcrud.com/api/bc0c.../diskon"
                @keyup.enter="applyApiUrl" />
            </div>
            <div v-if="urlError" class="field-error" style="align-self:flex-start">{{ urlError }}</div>
            <button class="btn btn-primary" @click="applyApiUrl">Terapkan</button>
          </div>
        </div>

        <div v-else-if="loading"><div class="spinner"></div></div>

        <div v-else-if="discounts.length === 0" class="empty-state">
          <div class="empty-illustration">
            <svg viewBox="0 0 140 110" fill="none" xmlns="http://www.w3.org/2000/svg">
              <ellipse cx="70" cy="75" rx="52" ry="28" fill="#FF7043"/>
              <ellipse cx="70" cy="68" rx="48" ry="36" fill="#FF8A65"/>
              <rect x="34" y="25" width="52" height="48" rx="6" fill="#4CAF50" transform="rotate(-8 34 25)"/>
              <text x="50" y="60" font-size="24" font-weight="900" fill="white" transform="rotate(-8 50 60)">%</text>
              <circle cx="94" cy="28" r="7" fill="#C8E6C9" stroke="#4CAF50" stroke-width="2"/>
              <circle cx="42" cy="33" r="5" fill="#A5D6A7" stroke="#388E3C" stroke-width="1.5"/>
              <ellipse cx="70" cy="98" rx="38" ry="9" fill="#FFD54F" opacity="0.7"/>
            </svg>
          </div>
          <div class="empty-title">Belum Ada Diskon</div>
          <div class="empty-desc">Silahkan tambah diskon untuk menarik pelanggan dan meningkatkan penjualan.</div>
          <button class="btn btn-primary" @click="openAdd">
            <span class="material-icons-round">add</span> Tambah diskon
          </button>
        </div>

        <div v-else-if="filteredDiscounts.length === 0" class="empty-state" style="padding:48px 24px">
          <span class="material-icons-round" style="font-size:44px;color:var(--md-outline-variant)">search_off</span>
          <div class="empty-title">Tidak Ditemukan</div>
          <div class="empty-desc">Tidak ada diskon yang cocok dengan "{{ searchQuery }}"</div>
        </div>

        <template v-else>
          <div class="table-header">
            <div class="cb-wrap">
              <div class="cb-custom" :class="{ checked: allSelected, indeterminate: someSelected && !allSelected }" @click="toggleSelectAll">
                <span class="material-icons-round">{{ allSelected ? 'check' : 'remove' }}</span>
              </div>
            </div>
            <div class="th-cell" @click="toggleSort('name')">
              Nama Diskon <span class="material-icons-round sort-icon">{{ sortField==='name'?(sortAsc?'arrow_upward':'arrow_downward'):'unfold_more' }}</span>
            </div>
            <div class="th-cell" @click="toggleSort('value')">
              Nilai Diskon <span class="material-icons-round sort-icon">{{ sortField==='value'?(sortAsc?'arrow_upward':'arrow_downward'):'unfold_more' }}</span>
            </div>
            <div></div>
          </div>
          <div v-for="item in filteredDiscounts" :key="item._id" class="table-row">
            <div class="cb-wrap">
              <div class="cb-custom" :class="{ checked: selectedIds.includes(item._id) }" @click="toggleSelect(item._id)">
                <span class="material-icons-round">check</span>
              </div>
            </div>
            <div class="row-name">{{ item.nama_diskon }}<span v-if="isNew(item._id)" class="badge-baru">baru</span></div>
            <div class="row-value">{{ formatValue(item) }}</div>
            <div><button class="icon-btn" @click="openEdit(item)"><span class="material-icons-round">edit</span></button></div>
          </div>
        </template>
      </div>
    </div>

    <Transition name="fade">
      <DiskonForm v-if="modal.show" :mode="modal.mode" :initial-data="modal.item"
        :existing-names="existingNames" :saving="saving"
        @close="modal.show=false" @save="onSave" @delete="onRequestDelete" />
    </Transition>

    <Transition name="fade">
      <DeleteConfirm v-if="deleteModal.show" :item-name="deleteModal.itemName" :loading="saving"
        @confirm="deleteModal.bulk ? confirmBulkDelete() : confirmDelete()"
        @cancel="deleteModal.show=false" />
    </Transition>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import DiskonForm from './components/DiskonForm.vue'
import DeleteConfirm from './components/DeleteConfirm.vue'

const apiUrl = ref(localStorage.getItem('diskon_api_url') || '')
const apiUrlInput = ref(apiUrl.value)
const outletLabel = ref(localStorage.getItem('diskon_outlet_label') || 'Kopi Anak Bangsa')
const outletOpen = ref(false)
const urlError = ref('')
const showSetup = ref(false)

function applyApiUrl() {
  const url = apiUrlInput.value.trim()
  if (!url) { urlError.value = 'URL tidak boleh kosong.'; return }
  if (!url.startsWith('http')) { urlError.value = 'URL tidak valid.'; return }
  urlError.value = ''
  const parts = url.split('/')
  const resource = parts[parts.length - 1] || 'diskon'
  outletLabel.value = resource.charAt(0).toUpperCase() + resource.slice(1)
  apiUrl.value = url
  localStorage.setItem('diskon_api_url', url)
  localStorage.setItem('diskon_outlet_label', outletLabel.value)
  outletOpen.value = false
  showSetup.value = false
  fetchDiscounts()
}

const discounts = ref([])
const loading = ref(false)
const saving = ref(false)
const searchQuery = ref('')
const sortField = ref('name')
const sortAsc = ref(true)
const selectedIds = ref([])
const newIds = ref([])
const toast = ref({ show: false, message: '' })
const modal = ref({ show: false, mode: 'add', item: null })
const deleteModal = ref({ show: false, bulk: false, itemId: null, itemName: '' })

let toastTimer = null
function showToast(msg) {
  clearTimeout(toastTimer)
  toast.value = { show: true, message: msg }
  toastTimer = setTimeout(() => { toast.value.show = false }, 3000)
}

async function fetchDiscounts() {
  if (!apiUrl.value) return
  loading.value = true
  try {
    const res = await fetch(apiUrl.value)
    if (!res.ok) throw new Error()
    discounts.value = await res.json()
  } catch {
    showToast('Gagal memuat data. Periksa API URL Anda.')
  } finally {
    loading.value = false
  }
}

const filteredDiscounts = computed(() => {
  let list = [...discounts.value]
  if (searchQuery.value) {
    const q = searchQuery.value.toLowerCase()
    list = list.filter(d => d.nama_diskon?.toLowerCase().includes(q))
  }
  list.sort((a, b) => {
    const va = sortField.value === 'name' ? (a.nama_diskon || '') : (Number(a.nilai) || 0)
    const vb = sortField.value === 'name' ? (b.nama_diskon || '') : (Number(b.nilai) || 0)
    if (typeof va === 'string') return sortAsc.value ? va.localeCompare(vb) : vb.localeCompare(va)
    return sortAsc.value ? va - vb : vb - va
  })
  return list
})

const existingNames = computed(() => discounts.value.map(d => d.nama_diskon))
const allSelected = computed(() => filteredDiscounts.value.length > 0 && filteredDiscounts.value.every(d => selectedIds.value.includes(d._id)))
const someSelected = computed(() => filteredDiscounts.value.some(d => selectedIds.value.includes(d._id)))

function formatValue(item) {
  if (item.tipe === 'rp') return 'Rp ' + Number(item.nilai).toLocaleString('id-ID')
  return item.nilai + '%'
}
function isNew(id) { return newIds.value.includes(id) }
function toggleSort(f) { if (sortField.value === f) sortAsc.value = !sortAsc.value; else { sortField.value = f; sortAsc.value = true } }
function toggleSelect(id) { const i = selectedIds.value.indexOf(id); if (i >= 0) selectedIds.value.splice(i,1); else selectedIds.value.push(id) }
function toggleSelectAll() {
  if (allSelected.value) selectedIds.value = selectedIds.value.filter(id => !filteredDiscounts.value.find(d=>d._id===id))
  else filteredDiscounts.value.forEach(d => { if (!selectedIds.value.includes(d._id)) selectedIds.value.push(d._id) })
}
function clearSelection() { selectedIds.value = [] }
function openAdd() { modal.value = { show: true, mode: 'add', item: null } }
function openEdit(item) { modal.value = { show: true, mode: 'edit', item } }

async function onSave(payload) {
  saving.value = true
  try {
    if (modal.value.mode === 'add') {
      const res = await fetch(apiUrl.value, { method: 'POST', headers: {'Content-Type':'application/json'}, body: JSON.stringify(payload) })
      if (!res.ok) throw new Error()
      const created = await res.json()
      discounts.value.push(created)
      newIds.value.push(created._id)
      setTimeout(() => { newIds.value = newIds.value.filter(id => id !== created._id) }, 10000)
      showToast(`${payload.nama_diskon} berhasil ditambahkan.`)
    } else {
      const id = modal.value.item._id
      const res = await fetch(`${apiUrl.value}/${id}`, { method: 'PUT', headers: {'Content-Type':'application/json'}, body: JSON.stringify(payload) })
      if (!res.ok) throw new Error()
      const idx = discounts.value.findIndex(d => d._id === id)
      if (idx >= 0) discounts.value[idx] = { ...discounts.value[idx], ...payload }
      showToast(`${payload.nama_diskon} berhasil disimpan.`)
    }
    modal.value.show = false
  } catch { showToast('Terjadi kesalahan. Coba lagi.') }
  finally { saving.value = false }
}

function onRequestDelete() {
  deleteModal.value = { show: true, bulk: false, itemId: modal.value.item._id, itemName: modal.value.item.nama_diskon }
  modal.value.show = false
}

async function confirmDelete() {
  saving.value = true
  try {
    const res = await fetch(`${apiUrl.value}/${deleteModal.value.itemId}`, { method: 'DELETE' })
    if (!res.ok) throw new Error()
    discounts.value = discounts.value.filter(d => d._id !== deleteModal.value.itemId)
    showToast(`${deleteModal.value.itemName} berhasil dihapus.`)
    deleteModal.value.show = false
  } catch { showToast('Gagal menghapus. Coba lagi.') }
  finally { saving.value = false }
}

function openBulkDelete() { deleteModal.value = { show: true, bulk: true, itemId: null, itemName: '' } }

async function confirmBulkDelete() {
  saving.value = true
  try {
    await Promise.all(selectedIds.value.map(id => fetch(`${apiUrl.value}/${id}`, { method: 'DELETE' })))
    discounts.value = discounts.value.filter(d => !selectedIds.value.includes(d._id))
    showToast('Diskon berhasil dihapus.')
    selectedIds.value = []
    deleteModal.value.show = false
  } catch { showToast('Gagal menghapus beberapa item.') }
  finally { saving.value = false }
}

function handleOutsideClick(e) { if (!e.target.closest('.outlet-chip')) outletOpen.value = false }
onMounted(() => { if (apiUrl.value) fetchDiscounts(); document.addEventListener('click', handleOutsideClick) })
onUnmounted(() => document.removeEventListener('click', handleOutsideClick))
</script>

<style scoped>
#app-wrapper { min-height: 100vh; display: flex; flex-direction: column; }
.page-content { flex: 1; padding: 24px; max-width: 1200px; margin: 0 auto; width: 100%; }
.page-header { display: flex; align-items: flex-start; justify-content: space-between; gap: 16px; margin-bottom: 16px; flex-wrap: wrap; }
.page-title { font-size: 28px; font-weight: 700; font-family: 'Google Sans','Roboto',sans-serif; line-height: 1.2; }
.page-subtitle { font-size: 13px; color: var(--md-on-surface-variant); margin-top: 4px; }
.header-actions { display: flex; gap: 8px; align-items: center; flex-wrap: wrap; }
.toolbar { display: flex; align-items: center; gap: 12px; flex-wrap: wrap; margin-bottom: 16px; }
.search-field { display: flex; align-items: center; gap: 8px; padding: 8px 14px; border-radius: var(--radius-xl); border: 1.5px solid var(--md-outline); background: var(--md-surface); transition: border-color var(--transition); min-width: 200px; }
.search-field:focus-within { border-color: var(--md-primary); }
.search-field .material-icons-round { font-size: 18px; color: var(--md-on-surface-variant); }
.search-field input { border: none; outline: none; font-size: 14px; color: var(--md-on-surface); background: transparent; width: 160px; }
.outlet-chip { display: inline-flex; align-items: center; gap: 6px; padding: 8px 12px; border-radius: var(--radius-xl); border: 1.5px solid var(--md-outline); background: var(--md-surface); cursor: pointer; font-size: 14px; transition: border-color var(--transition); position: relative; user-select: none; }
.outlet-chip:hover { border-color: var(--md-primary); }
.outlet-chip .material-icons-round { font-size: 16px; color: var(--md-on-surface-variant); }
.chevron { transition: transform var(--transition); }
.outlet-chip.open .chevron { transform: rotate(180deg); }
.outlet-panel { position: absolute; top: calc(100% + 8px); left: 0; background: var(--md-surface); border-radius: var(--radius-md); border: 1px solid var(--md-outline); box-shadow: 0 4px 16px rgba(0,0,0,.1); padding: 16px; min-width: 340px; z-index: 100; }
.outlet-chip-setup { display: inline-flex; align-items: center; gap: 6px; padding: 8px 12px; border-radius: var(--radius-xl); border: 1.5px solid var(--md-outline); background: var(--md-surface); cursor: pointer; font-size: 14px; }
.outlet-chip-setup .material-icons-round { font-size: 16px; color: var(--md-on-surface-variant); }
.setup-panel { position: absolute; top: calc(100% + 8px); left: 0; background: var(--md-surface); border-radius: var(--radius-md); border: 1px solid var(--md-outline); box-shadow: 0 4px 16px rgba(0,0,0,.1); padding: 16px; min-width: 340px; z-index: 100; }
.table-header { display: grid; grid-template-columns: 40px 1fr 200px 40px; padding: 12px 20px; background: var(--md-surface); border-bottom: 1px solid var(--md-outline); }
.th-cell { font-size: 13px; font-weight: 600; font-family: 'Google Sans','Roboto',sans-serif; color: var(--md-on-surface); display: flex; align-items: center; gap: 4px; cursor: pointer; user-select: none; }
.sort-icon { font-size: 14px; color: var(--md-on-surface-variant); }
.table-row { display: grid; grid-template-columns: 40px 1fr 200px 40px; padding: 14px 20px; border-bottom: 1px solid var(--md-outline); align-items: center; transition: background var(--transition); }
.table-row:last-child { border-bottom: none; }
.table-row:hover { background: #F1F8F1; }
.row-name { display: flex; align-items: center; gap: 8px; font-size: 14px; }
.row-value { font-size: 14px; color: var(--md-on-surface-variant); }
.badge-baru { background: #E3F2FD; color: #1565C0; border: 1px solid #90CAF9; border-radius: 20px; padding: 2px 8px; font-size: 11px; font-weight: 500; }
.cb-wrap { display: inline-flex; align-items: center; justify-content: center; }
.cb-custom { width: 20px; height: 20px; border-radius: 4px; border: 2px solid var(--md-outline-variant); display: inline-flex; align-items: center; justify-content: center; cursor: pointer; transition: all var(--transition); }
.cb-custom.checked, .cb-custom.indeterminate { background: var(--md-primary); border-color: var(--md-primary); }
.cb-custom .material-icons-round { font-size: 14px; color: white; display: none; }
.cb-custom.checked .material-icons-round, .cb-custom.indeterminate .material-icons-round { display: block; }
.empty-state { display: flex; flex-direction: column; align-items: center; justify-content: center; padding: 80px 24px; gap: 12px; text-align: center; }
.inline-setup { display: flex; flex-direction: column; align-items: stretch; gap: 10px; width: 100%; max-width: 400px; margin-top: 4px; }
.empty-illustration { width: 140px; height: 110px; }
.empty-title { font-size: 18px; font-weight: 700; font-family: 'Google Sans','Roboto',sans-serif; }
.empty-desc { font-size: 14px; color: var(--md-on-surface-variant); max-width: 300px; line-height: 1.5; }
.toast-container { position: fixed; top: 24px; left: 50%; transform: translateX(-50%); z-index: 999; }
.toast { background: #1B3A1E; color: #fff; padding: 12px 24px; border-radius: var(--radius-xl); font-size: 14px; box-shadow: 0 4px 16px rgba(0,0,0,.2); animation: toastIn .3s ease; white-space: nowrap; }
@keyframes toastIn { from { opacity: 0; transform: translateY(-12px); } to { opacity: 1; transform: none; } }
.app-footer { padding: 16px 24px; font-size: 12px; color: var(--md-on-surface-variant); border-top: 1px solid var(--md-outline); background: var(--md-surface); }
@media (max-width: 600px) { .page-content { padding: 16px; } .table-header, .table-row { grid-template-columns: 36px 1fr 100px 36px; } }
</style>
