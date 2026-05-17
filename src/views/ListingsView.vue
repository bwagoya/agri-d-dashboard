<template>
  <v-container>
    <v-row>
      <v-col>
        <h2 class="text-h5 mb-4">
          <v-icon class="mr-2">mdi-sprout</v-icon>
          Produce Listings
        </h2>
      </v-col>
      <v-col cols="auto">
        <v-btn
          color="green"
          variant="tonal"
          prepend-icon="mdi-refresh"
          @click="fetchListings"
        >
          Refresh
        </v-btn>
      </v-col>
    </v-row>

    <v-card>
      <v-data-table
        :headers="headers"
        :items="listings"
        :loading="loading"
        loading-text="Loading listings..."
        no-data-text="No listings yet"
      >
        <template v-slot:item.status="{ item }">
          <v-chip :color="statusColor(item.status)" size="small">
            {{ item.status }}
          </v-chip>
        </template>
        <template v-slot:item.iot_verified="{ item }">
          <v-icon :color="item.iot_verified ? 'green' : 'grey'">
            {{ item.iot_verified ? 'mdi-check-circle' : 'mdi-clock-outline' }}
          </v-icon>
        </template>
        <template v-slot:item.blockchain_recorded="{ item }">
          <v-icon :color="item.blockchain_recorded ? 'green' : 'grey'">
            {{ item.blockchain_recorded ? 'mdi-check-circle' : 'mdi-clock-outline' }}
          </v-icon>
        </template>
        <template v-slot:item.listed_at="{ item }">
          {{ formatDate(item.listed_at) }}
        </template>
        <template v-slot:item.asked_price="{ item }">
          KES {{ item.asked_price?.toLocaleString() }}
        </template>
      </v-data-table>
    </v-card>

    <v-row class="mt-4">
      <v-col cols="12" md="3">
        <v-card color="blue" variant="tonal">
          <v-card-text class="text-center">
            <div class="text-h4">{{ listings.length }}</div>
            <div>Total Listings</div>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col cols="12" md="3">
        <v-card color="orange" variant="tonal">
          <v-card-text class="text-center">
            <div class="text-h4">{{ pendingCount }}</div>
            <div>Pending</div>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col cols="12" md="3">
        <v-card color="green" variant="tonal">
          <v-card-text class="text-center">
            <div class="text-h4">{{ verifiedCount }}</div>
            <div>Verified</div>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col cols="12" md="3">
        <v-card color="red" variant="tonal">
          <v-card-text class="text-center">
            <div class="text-h4">{{ flaggedCount }}</div>
            <div>Flagged</div>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { supabase } from '../supabase'

const listings = ref([])
const loading = ref(false)

const headers = [
  { title: 'Listing ID', key: 'listing_id' },
  { title: 'Phone', key: 'phone_number' },
  { title: 'Crop', key: 'crop_type' },
  { title: 'Quantity', key: 'quantity' },
  { title: 'Price (KES)', key: 'asked_price' },
  { title: 'Status', key: 'status' },
  { title: 'IoT', key: 'iot_verified' },
  { title: 'Blockchain', key: 'blockchain_recorded' },
  { title: 'Listed', key: 'listed_at' }
]

const pendingCount = computed(() => listings.value.filter(l => l.status === 'pending').length)
const verifiedCount = computed(() => listings.value.filter(l => l.status === 'verified').length)
const flaggedCount = computed(() => listings.value.filter(l => l.status === 'flagged').length)

const statusColor = (status) => {
  const colors = {
    pending: 'orange',
    verified: 'green',
    flagged: 'red',
    sold: 'blue',
    expired: 'grey'
  }
  return colors[status] || 'grey'
}

const fetchListings = async () => {
  loading.value = true
  const { data, error } = await supabase
    .from('produce_listings')
    .select('*')
    .order('listed_at', { ascending: false })

  if (!error) listings.value = data
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

onMounted(fetchListings)
</script>