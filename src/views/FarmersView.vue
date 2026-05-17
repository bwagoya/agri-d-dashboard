<template>
  <v-container>
    <v-row>
      <v-col>
        <h2 class="text-h5 mb-4">
          <v-icon class="mr-2">mdi-account-group</v-icon>
          Registered Farmers
        </h2>
      </v-col>
      <v-col cols="auto">
        <v-btn
          color="green"
          variant="tonal"
          prepend-icon="mdi-refresh"
          @click="fetchFarmers"
        >
          Refresh
        </v-btn>
      </v-col>
    </v-row>

    <v-card>
      <v-data-table
        :headers="headers"
        :items="farmers"
        :loading="loading"
        loading-text="Loading farmers..."
        no-data-text="No farmers registered yet"
      >
        <template v-slot:item.is_verified="{ item }">
          <v-chip
            :color="item.is_verified ? 'green' : 'grey'"
            size="small"
          >
            {{ item.is_verified ? 'Verified' : 'Unverified' }}
          </v-chip>
        </template>
        <template v-slot:item.registered_at="{ item }">
          {{ formatDate(item.registered_at) }}
        </template>
      </v-data-table>
    </v-card>

    <v-row class="mt-4">
      <v-col cols="12" md="4">
        <v-card color="green" variant="tonal">
          <v-card-text class="text-center">
            <div class="text-h4">{{ farmers.length }}</div>
            <div>Total Farmers</div>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col cols="12" md="4">
        <v-card color="blue" variant="tonal">
          <v-card-text class="text-center">
            <div class="text-h4">{{ verifiedCount }}</div>
            <div>Verified Farmers</div>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col cols="12" md="4">
        <v-card color="orange" variant="tonal">
          <v-card-text class="text-center">
            <div class="text-h4">{{ unverifiedCount }}</div>
            <div>Unverified Farmers</div>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { supabase } from '../supabase'

const farmers = ref([])
const loading = ref(false)

const headers = [
  { title: 'Phone Number', key: 'phone_number' },
  { title: 'Name', key: 'name' },
  { title: 'Location', key: 'location' },
  { title: 'Farm Size', key: 'farm_size' },
  { title: 'Status', key: 'is_verified' },
  { title: 'Registered', key: 'registered_at' }
]

const verifiedCount = computed(() => farmers.value.filter(f => f.is_verified).length)
const unverifiedCount = computed(() => farmers.value.filter(f => !f.is_verified).length)

const fetchFarmers = async () => {
  loading.value = true
  const { data, error } = await supabase
    .from('farmers')
    .select('*')
    .order('registered_at', { ascending: false })

  if (!error) farmers.value = data
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

onMounted(fetchFarmers)
</script>