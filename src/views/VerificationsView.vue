<template>
  <v-container>
    <v-row>
      <v-col>
        <h2 class="text-h5 mb-4">
          <v-icon class="mr-2">mdi-shield-check</v-icon>
          IoT Verifications
        </h2>
      </v-col>
      <v-col cols="auto">
        <v-btn
          color="green"
          variant="tonal"
          prepend-icon="mdi-refresh"
          @click="fetchVerifications"
        >
          Refresh
        </v-btn>
      </v-col>
    </v-row>

    <v-card>
      <v-data-table
        :headers="headers"
        :items="verifications"
        :loading="loading"
        loading-text="Loading verifications..."
        no-data-text="No verifications yet"
      >
        <template v-slot:item.risk_level="{ item }">
          <v-chip :color="riskColor(item.risk_level)" size="small">
            {{ item.risk_level }}
          </v-chip>
        </template>
        <template v-slot:item.verification_status="{ item }">
          <v-chip
            :color="item.verification_status === 'VERIFIED SAFE' ? 'green' : 'red'"
            size="small"
          >
            {{ item.verification_status }}
          </v-chip>
        </template>
        <template v-slot:item.verification_hash="{ item }">
          <span class="text-caption text-grey">
            {{ item.verification_hash?.substring(0, 16) }}...
          </span>
        </template>
        <template v-slot:item.verified_at="{ item }">
          {{ formatDate(item.verified_at) }}
        </template>
      </v-data-table>
    </v-card>

    <v-row class="mt-4">
      <v-col cols="12" md="4">
        <v-card color="green" variant="tonal">
          <v-card-text class="text-center">
            <div class="text-h4">{{ lowRiskCount }}</div>
            <div>LOW Risk</div>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col cols="12" md="4">
        <v-card color="orange" variant="tonal">
          <v-card-text class="text-center">
            <div class="text-h4">{{ mediumRiskCount }}</div>
            <div>MEDIUM Risk</div>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col cols="12" md="4">
        <v-card color="red" variant="tonal">
          <v-card-text class="text-center">
            <div class="text-h4">{{ highRiskCount }}</div>
            <div>HIGH Risk</div>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { supabase } from '../supabase'

const verifications = ref([])
const loading = ref(false)

const headers = [
  { title: 'Listing ID', key: 'listing_id' },
  { title: 'Temp (°C)', key: 'temperature' },
  { title: 'Humidity (%)', key: 'humidity' },
  { title: 'Moisture (%)', key: 'moisture' },
  { title: 'Weight (kg)', key: 'weight' },
  { title: 'Risk Level', key: 'risk_level' },
  { title: 'Status', key: 'verification_status' },
  { title: 'Hash', key: 'verification_hash' },
  { title: 'Verified At', key: 'verified_at' }
]

const lowRiskCount = computed(() => verifications.value.filter(v => v.risk_level === 'LOW').length)
const mediumRiskCount = computed(() => verifications.value.filter(v => v.risk_level === 'MEDIUM').length)
const highRiskCount = computed(() => verifications.value.filter(v => v.risk_level === 'HIGH').length)

const riskColor = (risk) => {
  const colors = { LOW: 'green', MEDIUM: 'orange', HIGH: 'red' }
  return colors[risk] || 'grey'
}

const fetchVerifications = async () => {
  loading.value = true
  const { data, error } = await supabase
    .from('iot_verifications')
    .select('*')
    .order('verified_at', { ascending: false })

  if (!error) verifications.value = data
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

onMounted(fetchVerifications)
</script>