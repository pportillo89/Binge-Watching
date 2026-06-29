<template>
  <v-dialog v-model="isOpen" max-width="500px">
    <v-card v-if="item" class="rounded-xl pa-2">
      
      <v-card-title class="d-flex justify-space-between align-center font-weight-bold text-h5 pt-4">
        <span>{{ item.title }}</span>
        <v-btn icon="mdi-close" variant="text" @click="closeDialog"></v-btn>
      </v-card-title>

      <v-divider class="mx-4 mb-3"></v-divider>

      <v-card-text class="pt-0">
        <div class="d-flex align-center mb-4" style="gap: 8px;">
          <v-chip :color="getColorPlatform(item.platform)" variant="flat">
            {{ item.platform || 'Unknown' }}
          </v-chip>
          <v-chip variant="outlined">
            {{ item.type }}
          </v-chip>
        </div>

        <v-list lines="one" class="bg-transparent px-0">
          <v-list-item v-if="item.type !== 'Movie'">
            <template #prepend>
              <v-icon color="primary" class="mr-3">mdi-television-play</v-icon>
            </template>
            <v-list-item-title class="text-body-1">
              <b>Current Progress:</b> Season {{ item.season }} — Episode {{ item.episode }}
            </v-list-item-title>
          </v-list-item>

          <v-list-item>
            <template #prepend>
              <v-icon color="secondary" class="mr-3">mdi-account-heart</v-icon>
            </template>
            <v-list-item-title class="text-body-1">
              <b>Recommended By:</b> {{ item.recommendedBy || 'Nobody' }}
            </v-list-item-title>
          </v-list-item>

         <!-- <v-list-item>
            <template #prepend>
              <v-icon color="success" class="mr-3">mdi-eye-check</v-icon>
            </template>
            <v-list-item-title class="text-body-1">
              <b>Status:</b> {{ item.type === 'Movie' ? 'Watched' : 'Watching' }}
            </v-list-item-title>
          </v-list-item>-->
        </v-list>
      </v-card-text>

      <v-card-actions class="justify-end pb-4 pr-4">
        <v-btn color="blue-grey-darken-1" variant="text" @click="closeDialog">
          Close
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script lang="ts" setup>
import { computed } from 'vue'

// 1. Definimos las propiedades que este componente recibe del padre
const props = defineProps<{
  modelValue: boolean // Controla si se muestra el diálogo
  item: any           // Recibe el objeto de la película seleccionada
  getColorPlatform: (platform: string) => string // Pasamos la función de colores para no duplicar código
}>()

// 2. Definimos los eventos que este componente puede enviar de vuelta al padre
const emit = defineEmits(['update:modelValue'])

// Computed inteligente para sincronizar el v-model con el padre
const isOpen = computed({
  get: () => props.modelValue,
  set: (value) => emit('update:modelValue', value)
})

const closeDialog = () => {
  emit('update:modelValue', false)
}
</script>