<template>
  <v-app>
    <v-app-bar border color="blue-grey">

      <!--El titulo largo aparece si es pc o tablet
      d-none: es para ocultar en celulares y d-sm-block para que se muestre en pantallas y tablets-->
          <v-app-bar-title class="font-weight-bold d-none d-sm-block">
            Welcome to BingeWatch
          </v-app-bar-title>

          <!--El titulo corto aparece si es celular
          d-block d-sm-none: hace lo contrario, solo muestra el tutulo si es celular-->
          <v-app-bar-title class="font-weight-bold d-block d-sm-none">
            BingeWatch
         </v-app-bar-title>

          <v-spacer></v-spacer>

          <v-text-field
            v-model="textSearch"
            hide-details="auto"
            label="Search"
            style="max-width: 250px; min-width: 150px;"
            variant="outlined"
            class="mr-2 mr-sm-4"
            @keydown.enter="searchWatch"
            > 
            <!--style="max-width: 250px; min-width: 150px;" se agrega un max y min para que sea flexible-->

            <template #append-inner>
              <v-btn
               icon="mdi-magnify"
               variant="text"
               density="comfortable"
               @click="searchWatch"
              > 
              <!--@click="searchWatch" Es mi funcion de busqueda-->
                <v-icon></v-icon>
              </v-btn>
            </template>

          </v-text-field> 
    </v-app-bar>

    <v-main>
      <v-container fluid class="bg-blue-grey-lighten-5 fill-height align-start">
        <v-row class="mb-6"> 
           <v-col cols="12" class="d-flex justify-center justify-sm-start">
            
            <v-btn-toggle
              v-model="activeFilter"
              color="blue-grey-lighten-2"
              mandatory
              rounded="xl"
            >
              <v-btn value="All">All</v-btn>
              <v-btn value="Movie">🎬 Movies</v-btn>
              <v-btn value="Serie">📺 Series</v-btn>

            </v-btn-toggle>
           </v-col>
        </v-row>

        <v-row v-if="itemsFilter.length === 0" justify="center" class="mt-10">
          <v-col cols="12" class="text-center text-medium-emphasis">
            <v-icon icon="mdi-movie-open-remove" size="large" class="mb-2"></v-icon>
            <p>There are no items to display in this category.</p>
          </v-col>
        </v-row>

        <!--Haremos el loop de tarjetas, aca se crean las tarjetas que muestra mi serie o pelicula-->
        <v-row>
          <v-col
          v-for="(item, index) in itemsFilter"
          :key="index"
          cols="12"
          sm="6"
          md="4"
          >

          <v-card border variant="outlined" class="rounded-xl pa-3">
            <v-card-item>
              <div class="d-flex justify-space-between align-center mb-2">

                <v-chip size="small" :color="getColorPlatform(item.platform)" variant="flat">
                  {{ item.platform || 'Unknown'}}
                </v-chip>

                <v-chip size="small" variant="outlined" class="ml-auto">
                  {{ item.type }}
                </v-chip>

              </div>

              <v-card-title class="text-h6 font-weight-bold px-0">
                {{ item.title }}
              </v-card-title>
            </v-card-item>

            <v-card-text class="text-medium-emphasis">
              <div v-if="item.type !=='Movie'" class="d-flex align-center my-2">
                  <div class="d-flex align-center justify-space-between my-2">
                    <span class="text-body-2 font-weight-bold text-medium-emphasis mr-2">Progress:</span>
                      <div class="d-flex align-center" style="gap: 16px;">
                        <div class="d-flex align-center">
                          <span class="mr-1">Seasson:</span>
                          <b class="text-body-2 mr-2">{{ item.season }}</b>
                          <v-btn icon="mdi-plus" size="x-small" variant="tonal" color="secondary" @click="plusSeason(item)" />
                        </div>

                        <div class="d-flex align-center">
                          <span class="mr-1">Ep.</span>
                          <b class="text-body-2 mr-2">{{ item.episode }}</b>
                          <v-btn icon="mdi-plus" size="x-small" variant="tonal" color="primary" @click="item.episode++" />
                        </div>
                    </div>
                  </div>
                </div>
              <div class="text-caption mt-2">
                  <v-icon icon="mdi-account-heart" size="small" class="mr-1"></v-icon>
                  Recommended By: <span class="text-primary">{{ item.recommendedBy || 'Nobody' }}</span>
                </div>
            </v-card-text>

            <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn icon="mdi-pencil-outline" size="small" color="primary" variant="text" @click="openToEdit(item)"></v-btn>
                <v-btn icon="mdi-delete-outline" size="small" color="error" variant="text" @click="deleteItem(item)"></v-btn>
            </v-card-actions>

          </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-main>

    <!--Boton de agregar una nueva pelicula o serie -->
    <v-btn
      icon="mdi-plus"
      color="blue-grey"
      size="large"
      location="bottom right"
      position="fixed"
      class="ma-6"
      @click="openToCreate"
    >
  </v-btn>

  <dialog-form
    v-model="dialogForm"
    :item-to-edit="itemSelectedToEdit"
    @save-item="addNewItem"
    @update-item="updateExistsItem"
  />
    
  </v-app>
</template>

<script lang="ts" setup>

  import { ref, computed, watch } from 'vue'
  //importamos nuestro componente hijo personalizado
  import DialogForm from './components/DialogForm.vue'


  //Estado de la UI al iniciar
   // Iniciamos el filtro con el valor "Todos" por defecto
  const activeFilter = ref('All')
  const dialogForm = ref(false)
  const textSearch = ref('') // Empieza vacío
  const itemSelectedToEdit = ref<any>(null)


// LOCAL STORAGE: Intentar cargar datos existentes, si no, usar lista vacía
  const saveData = localStorage.getItem('bingewatch_items')
  const items = ref(saveData ? JSON.parse(saveData) : [
    // Datos iniciales por defecto si el LocalStorage está vacío por primera vez
    { title: 'Stranger Things', type: 'Serie', platform: 'Netflix', season: 4, episode: 2, recommendedBy: 'Andrés' },
    { title: 'Inception', type: 'Movie', platform: 'Prime', season: 0, episode: 0, recommendedBy: 'Sofía'}
  ])

  // WATCHER: Detecta cualquier cambio en la lista (incluso aumentar episodios) y guarda en LocalStorage
  watch(items, (newItem) => {
  localStorage.setItem('bingewatch_items', JSON.stringify(newItem))
  }, { deep: true }) // 'deep: true' es clave para que detecte cuando cambia el número de episodio dentro de un objeto

  const searchWatch = () => {
    //Aca hay que insertar la logica del select
    console.log('Buscando')
  }

  // Filtrado reactivo
 /* const itemsFilter = computed( () => {
    if (activeFilter.value === 'All') {
      return items.value
    }
    return items.value.filter(item => item.type === activeFilter.value)
  })*/

  const itemsFilter = computed(() => {
    return items.value.filter((item :any)  => {
      // 1. Validar el filtro de la pestaña (All, Movie, Serie, etc.)
      const coincideTipo = activeFilter.value === 'All' || item.type === activeFilter.value

      // 2. Validar lo que el usuario escribió en la barra de búsqueda
      // Usamos || '' por si acaso el campo llega a estar indefinido en algún momento
      const coincideTexto = (item.title || '')
        .toLowerCase()
        .includes(textSearch.value.toLowerCase())

      // El elemento solo pasa el filtro si cumple AMBAS condiciones
      return coincideTipo && coincideTexto
    })
})

  // Colores personalizados por plataforma
  const getColorPlatform = (platform : string) => {
    const p = platform.toLowerCase()
    if (p.includes('netflix')) return 'red-darken-2'
    if (p.includes('prime')) return 'blue-darken-2'
    if (p.includes('disney')) return 'indigo-darken-3'
    if (p.includes('hbo')) return 'purple-darken-2'
  return 'grey-darken-1'  
  }

  // Lógica para añadir el nuevo elemento enviado desde el componente hijo
  const addNewItem = (itemRecived :any) => {
    items.value.push(itemRecived)
  }

  // Lógica para eliminar
  const deleteItem = (itemDelete :any) => {
    items.value = items.value.filter((i :any) => i !== itemDelete)
  }

//Logoca para EDITAR
  // Función que se activa al dar clic en el botón de lápiz de la tarjeta
  const openToEdit = (item: any) => {
    itemSelectedToEdit.value = item // Guardamos la referencia original
    dialogForm.value = true
  }

  // Lógica para ABRIR el formulario en modo CREAR NUEVO
  // (Le cambiamos el nombre a openToCreate para que no choque con addNewItem)
  // Función que se activa cuando le das al botón "+" flotante (Crear Nuevo)
  const openToCreate = () => {
    itemSelectedToEdit.value = null // Nos aseguramos de que vaya limpio
    dialogForm.value = true
  }

  // Función para procesar la actualización
  const updateExistsItem = (itemEditado: any) => {
    // Buscamos el elemento original en nuestra lista por su título original (o posición)
    const index = items.value.findIndex((i :any) => i.title === itemSelectedToEdit.value.title)
    if (index !== -1) {
      items.value[index] = itemEditado // Reemplazamos los datos viejos por los nuevos
    }
  }

  // Lógica para subir de temporada de forma inteligente
  const plusSeason = (item: any) => {
    item.season++
    item.episode = 1 // Al pasar de temporada, por lógica empezamos en el episodio 1
  }
 
</script>
