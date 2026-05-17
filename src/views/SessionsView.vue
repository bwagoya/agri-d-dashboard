<template>
  <v-container>
    <v-row>
      <v-col>
        <h2 class="text-h5 mb-4">
          <v-icon class="mr-2">mdi-history</v-icon>
          USSD Sessions
        </h2>
      </v-col>
      <v-col cols="auto">
        <v-btn
          color="green"
          variant="tonal"
          prepend-icon="mdi-refresh"
          @click="fetchSessions"
        >
          Refresh
        </v-btn>
      </v-col>
    </v-row>

    <v-card>
      <v-data-table
        :headers="headers"
        :items="sessions"
        :loading="loading"
        loading-text="Loading sessions..."
        no-data-text="No sessions yet"
      >
        <template v-slot:item.status="{ item }">
          <v-chip :color="statusColor(item.status)" size="small">
            {{ item.status }}
          </v-chip>
        </template>
        <template v-slot:item.created_at="{ item }">
          {{ formatDate(item.created_at) }}
        </template>
      </v-data-table>
    </v-card>

    <v-row class="mt-4">
      <v-col cols="12" md="4">
        <v-card color="green" variant="tonal">
          <v-card-text class="text-center">
            <div class="text-h4">{{ completedCount }}</div>
            <div>Completed</div>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col cols="12" md="4">
        <v-card color="blue" variant="tonal">
          <v-card-text class="text-center">
            <div class="text-h4">{{ activeCount }}</div>
            <div>Active</div>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col cols="12" md="4">
        <v-card color="grey" variant="tonal">
          <v-card-text class="text-center">
            <div class="text-h4">{{ abandonedCount }}</div>
            <div>Abandoned</div>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { supabase } from '../supabase'

const sessions = ref([])
const loading = ref(false)

const headers = [
  { title: 'Session ID', key: 'session_id' },
  { title: 'Phone Number', key: 'phone_number' },
  { title: 'Current Step', key: 'current_step' },
  { title: 'Status', key: 'status' },
  { title: 'Created At', key: 'created_at' }
]

const completedCount = computed(() => sessions.value.filter(s => s.status === 'completed').length)
const activeCount = computed(() => sessions.value.filter(s => s.status === 'active').length)
const abandonedCount = computed(() => sessions.value.filter(s => s.status === 'abandoned').length)

const statusColor = (status) => {
  const colors = {
    completed: 'green',
    active: 'blue',
    abandoned: 'grey'
  }
  return colors[status] || 'grey'
}

const fetchSessions = async () => {
  loading.value = true
  const { data, error } = await supabase
    .from('ussd_sessions')
    .select('*')
    .order('created_at', { ascending: false })
    .limit(50)

  if (!error) sessions.value = data
  loading.value = false
}

const formatDate = (date) => {
  if (!date) return 'N/A'
  return new Date(date).toLocaleDateString('en-KE', {
    day: 'numeric',
    month: 'short',
    year: 'numeric',
    hour: '2-digit',
    minute: '2-digit'
  })
}

onMounted(fetchSessions)
</script>