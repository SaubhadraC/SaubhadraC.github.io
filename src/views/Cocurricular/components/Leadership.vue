<template>
  <div class="border-l-4 border-[#1980e6] pl-6 py-4 pr-4 rounded-md bg-slate-50">
    <div class="flex items-center justify-between mb-2">
      <h3 class="text-lg font-semibold text-[#0e141b]">{{ leadership.role }}</h3>
      <span class="text-sm text-gray-500">{{ leadership.time_period }}</span>
    </div>
    <div class="flex flex-col md:flex-row md:items-center gap-2 md:gap-4 mb-2">
      <div class="flex items-center space-x-1">
        <v-icon small class="text-[#1980e6]">mdi-office-building</v-icon>
        <span class="text-gray-600">{{ leadership.organization }}</span>
        <a :href="leadership.web_link" target="_blank" v-if="leadership.web_link"
          class="text-[#1980e6] hover:underline text-sm">
          <span v-if="leadership.web_link"><v-icon size="16" class="text-gray-500">mdi-open-in-new</v-icon></span>
        </a>
      </div>
      <div v-if="leadership.host" class="flex items-center space-x-1">
        <v-icon small class="text-[#1980e6]">mdi-home</v-icon>
        <span class="text-gray-700">{{ leadership.host }}</span>
      </div>
      <div class="flex items-center space-x-1">
        <v-icon small class="text-[#1980e6]">mdi-map-marker</v-icon>
        <span class="text-gray-700">{{ leadership.location }}</span>
      </div>
    </div>
    <p class="text-gray-700 mb-3">{{ leadership.description }}</p>
    <div class="flex flex-wrap gap-2">
      <span v-for="skill in leadership.skills.split(',').map(item => item.trim())" :key="skill"
        class="px-2 py-1 bg-blue-100 text-blue-800 text-xs rounded">
        {{ skill }}
      </span>
    </div>
  </div>
</template>

<script setup>


defineProps({
  leadership: Object
})

</script>