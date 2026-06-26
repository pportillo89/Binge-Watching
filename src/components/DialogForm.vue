<template>
    <v-dialog :model-value="modelValue" @update:model-value="$emit('update:modelValue', $event)" max-width="500px" persistent>
        <v-card class="rounded-xl pa-4">
            <v-card-title class="font-weight-bold"> 
                {{ itemToEdit ? 'Edit Item' : 'Add to List' }}
            </v-card-title>
            <v-card-text>
                <v-text-field 
                    v-model="newItem.title"
                    label="Title"
                    variant="outlined"
                >
                </v-text-field>

                <v-select
                    v-model="newItem.type"
                    :items="['Movie', 'Serie']"
                    label="Type"
                    variant="outlined"
                >
                </v-select>

                <v-combobox
                    v-model="newItem.platform"
                    :items="['Netflix', 'Prime Video', 'Disney+', 'HBO Max']"
                    label="Where can I watch it?"
                    variant="outlined"
                >
                </v-combobox>

                <v-row v-if="newItem.type !== 'Movie'">
                <v-col cols="6">
                    <v-text-field v-model.number="newItem.season" label="Season" type="number" variant="outlined"></v-text-field>
                </v-col>
                <v-col cols="6">
                    <v-text-field v-model.number="newItem.episode" label="Current Episode" type="number" variant="outlined"></v-text-field>
                </v-col>
            </v-row>

            <v-text-field v-model="newItem.recommendedBy" label="Recommended By" variant="outlined"></v-text-field>

        </v-card-text>

        <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn variant="text" @click="close">Cancel</v-btn>
            <v-btn color="primary" variant="flat" class="px-4" @click="save">Save</v-btn>
        </v-card-actions>
        </v-card>
    </v-dialog>
</template>

<script setup>
    //importamos ref que funciona para guardar datos y avisa a vue si el dato cambia lo actualiza
    import { ref, watch} from 'vue';

    //Definimos propiedades (props) y eventos (emits)
    /*defineProps({
        modelValue: Boolean
    })*/

    const props = defineProps({
        modelValue: Boolean,
        itemToEdit: {
            type: Object,
            default: null
        }
    })

    const emit = defineEmits(['update:modelValue', 'save-item', 'update-item'])

    // Estado local del formulario
    const newItem = ref({
        title: '',
        type: 'Serie',
        platform: '',
        season: 1,
        episode: 1,
        recommendedBy: ''
    })

    // REACCIÓN: Si nos pasan un item para editar, rellenamos el formulario con sus datos.
    // Si viene null, limpiamos el formulario para modo "Crear".
    watch(() => props.modelValue, (isOpen) => {
        if (isOpen) {
            if (props.itemToEdit) {
            newItem.value = { ...props.itemToEdit } // Copia de los datos a editar
            } else {
            newItem.value = { title: '', type: 'Serie', platform: '', season: 1, episode: 1, recommendedBy: '' }
            }
        }
    })

    const close = () => {
        emit('update:modelValue', false)
    }

    //ESta funcion solo crea 
/*    const save = () => {
        if (newItem.value.title) {
            // Enviamos una copia de los datos al componente padre
            emit('save-item', { ...newItem.value })
            
            // Reiniciamos el formulario para la próxima vez
            newItem.value = { title: '', type: 'Serie', platform: '', season: 1, episode: 1, recommendedBy: '' }
            close()
        }
    }*/

    const save = () => {
        if (newItem.value.title) {
            if (props.itemToEdit) {
            // Si estábamos editando, avisamos al padre que actualice
            emit('update-item', { ...newItem.value })
            } else {
            // Si era nuevo, avisamos que lo agregue
            emit('save-item', { ...newItem.value })
            }
            close()
        }
    }

</script>