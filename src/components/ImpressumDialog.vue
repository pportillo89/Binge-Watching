<template>
  <!-- Usamos una propiedad computada para sincronizar el v-model de forma segura con el padre -->
  <v-dialog v-model="isOpen" max-width="450px">
    <v-card class="rounded-xl pa-4">
      <v-card-title class="font-weight-bold text-h5 px-0 pt-0 d-flex justify-space-between align-center">
        <span>Impressum</span>
        <v-btn icon="mdi-close" variant="text" size="small" @click="closeDialog"></v-btn>
      </v-card-title>
      
      <v-divider class="mb-4"></v-divider>

      <v-card-text class="px-0 py-0 text-body-2">
        <p class="font-weight-bold text-primary mb-1">Angaben gemäß § 5 TMG:</p>
        <p class="mb-4 text-medium-emphasis">
          Paola Marcela Portillo Martell<br>
          In der Ebene 3<br>
          97218, Gerbrunn, Deutschland
        </p>

        <p class="font-weight-bold text-primary mb-1">Kontakt:</p>
        <p class="mb-4 text-medium-emphasis">
          E-Mail: paolaportillo02@email.com
        </p>

        <v-alert
          type="info"
          variant="tonal"
          density="compact"
          class="text-caption mt-4 rounded-lg"
          icon="mdi-information-outline"
        >
          <b>Hinweis:</b> Dies ist eine Coding-Challenge / ein Proof of Concept (PoC) zu Bewerbungszwecken. 
          Es werden keine kommerziellen Absichten verfolgt.
        </v-alert>
      </v-card-text>

      <v-card-actions class="justify-end pe-0 pb-0 pt-4">
        <v-btn color="grey-darken-1" variant="text" @click="closeDialog">
          Close
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script lang="ts" setup>
import { computed } from 'vue'

// Recibe el estado del booleano v-model desde el componente padre
const props = defineProps<{
  modelValue: boolean
}>()

const emit = defineEmits(['update:modelValue'])

// Computed inteligente para manejar el flujo bidireccional del v-model
const isOpen = computed({
  get: () => props.modelValue,
  set: (value) => emit('update:modelValue', value)
})

const closeDialog = () => {
  emit('update:modelValue', false)
}
</script>