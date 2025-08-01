<script lang="ts" setup>
import { ref, computed } from 'vue'

interface RegistroSemanal {
  proyecto: string
  año: number
  semana: number
  horas: number
  descripcion: string
}

const proyectos = ['Proyecto A', 'Proyecto B', 'Proyecto C']
const selectedProyecto = ref(proyectos[0])
const año = ref(new Date().getFullYear())
const semana = ref(getCurrentWeek())
const horas = ref<number | null>(null)
const descripcion = ref('')
const historial = ref<RegistroSemanal[]>([])

// Función para obtener la semana actual del año
function getCurrentWeek(): number {
  const now = new Date()
  const start = new Date(now.getFullYear(), 0, 1)
  const days = Math.floor((now.getTime() - start.getTime()) / (24 * 60 * 60 * 1000))
  return Math.ceil((days + start.getDay() + 1) / 7)
}

// Función para obtener el rango de fechas de una semana específica
function getWeekRange(año: number, semana: number): string {
  const firstDay = new Date(año, 0, 1 + (semana - 1) * 7)
  const dayOfWeek = firstDay.getDay()
  const mondayOffset = dayOfWeek === 0 ? -6 : 1 - dayOfWeek
  
  const monday = new Date(firstDay)
  monday.setDate(firstDay.getDate() + mondayOffset)
  
  const sunday = new Date(monday)
  sunday.setDate(monday.getDate() + 6)
  
  const formatDate = (date: Date) => {
    return date.toLocaleDateString('es-ES', { 
      day: '2-digit', 
      month: '2-digit' 
    })
  }
  
  return `${formatDate(monday)} - ${formatDate(sunday)}`
}

// Computed para mostrar el rango de fechas de la semana seleccionada
const semanaSeleccionadaRango = computed(() => {
  return getWeekRange(año.value, semana.value)
})

function registrarHoras() {
  if (!horas.value || !descripcion.value) return

  // Verificar si ya existe un registro para este proyecto, año y semana
  const existeRegistro = historial.value.find(
    r => r.proyecto === selectedProyecto.value && 
         r.año === año.value && 
         r.semana === semana.value
  )

  if (existeRegistro) {
    // Actualizar el registro existente
    existeRegistro.horas = horas.value
    existeRegistro.descripcion = descripcion.value
  } else {
    // Crear nuevo registro
    historial.value.push({
      proyecto: selectedProyecto.value,
      año: año.value,
      semana: semana.value,
      horas: horas.value,
      descripcion: descripcion.value
    })
  }

  // Limpiar formulario
  horas.value = null
  descripcion.value = ''
}

// Computed para ordenar el historial por año y semana (más reciente primero)
const historialOrdenado = computed(() => {
  return [...historial.value].sort((a, b) => {
    if (a.año !== b.año) return b.año - a.año
    return b.semana - a.semana
  })
})
</script>

<template>
  <div class="bg-white p-6 rounded-lg shadow space-y-4">
    <h2 class="text-xl font-semibold text-gray-700">Registrar Horas Semanales</h2>

    <div class="space-y-4">
      <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
        <div>
          <label class="block text-gray-600 mb-1">Año</label>
          <input 
            type="number" 
            v-model="año" 
            :min="2020" 
            :max="2030"
            class="w-full border border-gray-300 rounded p-2 focus:border-blue-500 focus:outline-none"
          />
        </div>
        
        <div>
          <label class="block text-gray-600 mb-1">Semana del año</label>
          <input 
            type="number" 
            v-model="semana" 
            :min="1" 
            :max="53"
            class="w-full border border-gray-300 rounded p-2 focus:border-blue-500 focus:outline-none"
          />
        </div>
      </div>

      <div class="bg-blue-50 p-3 rounded">
        <p class="text-sm text-blue-700">
          <strong>Semana {{ semana }}, {{ año }}:</strong> {{ semanaSeleccionadaRango }}
        </p>
      </div>

      <div>
        <label class="block text-gray-600 mb-1">Proyecto</label>
        <select v-model="selectedProyecto" class="w-full border border-gray-300 rounded p-2 focus:border-blue-500 focus:outline-none">
          <option v-for="p in proyectos" :key="p" :value="p">{{ p }}</option>
        </select>
      </div>

      <div>
        <label class="block text-gray-600 mb-1">Horas trabajadas</label>
        <input 
          type="number" 
          min="0" 
          max="168"
          step="0.5"
          v-model="horas" 
          class="w-full border border-gray-300 rounded p-2 focus:border-blue-500 focus:outline-none"
          placeholder="Ej: 40"
        />
      </div>

      <div>
        <label class="block text-gray-600 mb-1">Descripción de actividades</label>
        <textarea 
          v-model="descripcion" 
          class="w-full border border-gray-300 rounded p-2 focus:border-blue-500 focus:outline-none" 
          rows="3"
          placeholder="Describe las principales actividades realizadas durante la semana..."
        />
      </div>

      <button 
        @click="registrarHoras" 
        :disabled="!horas || !descripcion"
        class="w-full bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700 disabled:bg-gray-400 disabled:cursor-not-allowed transition-colors"
      >
        {{ historial.find(r => r.proyecto === selectedProyecto && r.año === año && r.semana === semana) ? 'Actualizar Registro' : 'Registrar Horas' }}
      </button>
    </div>

    <div v-if="historial.length" class="mt-8">
      <h3 class="text-lg font-semibold mb-4 text-gray-700">Historial de Registros</h3>
      <div class="space-y-3">
        <div 
          v-for="(item, index) in historialOrdenado" 
          :key="index" 
          class="border border-gray-200 p-4 rounded-lg bg-gray-50 hover:bg-gray-100 transition-colors"
        >
          <div class="flex justify-between items-start mb-2">
            <div>
              <h4 class="font-semibold text-gray-800">{{ item.proyecto }}</h4>
              <p class="text-sm text-gray-600">
                Semana {{ item.semana }}, {{ item.año }} 
                <span class="text-gray-500">({{ getWeekRange(item.año, item.semana) }})</span>
              </p>
            </div>
            <div class="text-right">
              <span class="text-lg font-bold text-blue-600">{{ item.horas }}h</span>
            </div>
          </div>
          <p class="text-sm text-gray-700 mt-2">{{ item.descripcion }}</p>
        </div>
      </div>
    </div>
  </div>
</template>