<script lang="ts" setup>
interface ProyectoHoras {
  nombre: string
  horas: number
}

const resumen = {
  porProyecto: [
    { nombre: 'Proyecto A', horas: 24 },
    { nombre: 'Proyecto B', horas: 16 }
  ] as ProyectoHoras[],
  cargaSemanal: '40 horas esta semana'
}
</script>

<template>
  <div class="space-y-4">
    <div class="bg-white p-6 rounded shadow">
      <h2 class="text-lg font-semibold text-gray-700">Horas por Proyecto</h2>
      <ul class="mt-2 space-y-1">
        <li v-for="item in resumen.porProyecto" :key="item.nombre">
          {{ item.nombre }}: <strong>{{ item.horas }}h</strong>
        </li>
      </ul>
    </div>

    <div class="bg-white p-6 rounded shadow">
      <h2 class="text-lg font-semibold text-gray-700">Carga Semanal</h2>
      <p class="mt-2 text-gray-600">{{ resumen.cargaSemanal }}</p>
    </div>
  </div>
</template>
