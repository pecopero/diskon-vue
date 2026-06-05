<template>
  <div class="modal-overlay" @click.self="$emit('cancel')">
    <div class="modal modal-sm">
      <div class="modal-header">
        <div class="modal-title">Hapus Diskon</div>
      </div>
      <p style="font-size:14px;line-height:1.6">
        <template v-if="itemName">
          Apakah Anda yakin ingin menghapus diskon <strong>{{ itemName }}</strong>?
        </template>
        <template v-else>
          Apakah Anda yakin ingin menghapus diskon yang dipilih?
        </template>
      </p>
      <ul style="list-style:disc;padding-left:20px;margin-top:8px">
        <li style="font-size:14px;color:var(--md-on-surface-variant)">Diskon yang dihapus tidak bisa dikembalikan lagi.</li>
      </ul>
      <div class="modal-footer modal-footer-end" style="gap:10px">
        <button class="btn btn-danger-outline btn-sm" @click="$emit('cancel')">Batalkan</button>
        <button class="btn btn-danger btn-sm" @click="$emit('confirm')" :disabled="loading">
          {{ loading ? 'Menghapus...' : 'Hapus' }}
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
defineProps({
  itemName: { type: String, default: '' },
  loading: { type: Boolean, default: false },
})
defineEmits(['confirm', 'cancel'])
</script>
