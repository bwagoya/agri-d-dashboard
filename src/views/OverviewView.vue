<template>
  <v-container>
    <h2 class="text-h5 mb-4">
      <v-icon class="mr-2">mdi-view-dashboard</v-icon>
      System Overview
    </h2>

    <!-- Stats Cards -->
    <v-row class="mb-4">
      <v-col cols="12" md="3">
        <v-card color="green" variant="tonal" rounded="lg">
          <v-card-text class="text-center pa-6">
            <v-icon size="40" color="green">mdi-account-group</v-icon>
            <div class="text-h3 font-weight-bold mt-2">{{ stats.totalFarmers }}</div>
            <div class="text-subtitle-1">Total Farmers</div>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col cols="12" md="3">
        <v-card color="blue" variant="tonal" rounded="lg">
          <v-card-text class="text-center pa-6">
            <v-icon size="40" color="blue">mdi-sprout</v-icon>
            <div class="text-h3 font-weight-bold mt-2">{{ stats.totalListings }}</div>
            <div class="text-subtitle-1">Total Listings</div>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col cols="12" md="3">
        <v-card color="orange" variant="tonal" rounded="lg">
          <v-card-text class="text-center pa-6">
            <v-icon size="40" color="orange">mdi-clock-outline</v-icon>
            <div class="text-h3 font-weight-bold mt-2">{{ stats.pendingListings }}</div>
            <div class="text-subtitle-1">Pending Verification</div>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col cols="12" md="3">
        <v-card color="purple" variant="tonal" rounded="lg">
          <v-card-text class="text-center pa-6">
            <v-icon size="40" color="purple">mdi-shield-check</v-icon>
            <div class="text-h3 font-weight-bold mt-2">{{ stats.totalVerifications }}</div>
            <div class="text-subtitle-1">IoT Verifications</div>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>

    <!-- Risk Level Breakdown -->
    <v-row class="mb-4">
      <v-col cols="12" md="4">
        <v-card rounded="lg">
          <v-card-title class="text-subtitle-1">
            <v-icon color="green" class="mr-2">mdi-check-circle</v-icon>
            LOW Risk
          </v-card-title>
          <v-card-text class="text-center">
            <div class="text-h3 font-weight-bold text-green">{{ stats.lowRisk }}</div>
            <div class="text-caption">listings verified safe</div>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col cols="12" md="4">
        <v-card rounded="lg">
          <v-card-title class="text-subtitle-1">
            <v-icon color="orange" class="mr-2">mdi-alert</v-icon>
            MEDIUM Risk
          </v-card-title>
          <v-card-text class="text-center">
            <div class="text-h3 font-weight-bold text-orange">{{ stats.mediumRisk }}</div>
            <div class="text-caption">listings need attention</div>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col cols="12" md="4">
        <v-card rounded="lg">
          <v-card-title class="text-subtitle-1">
            <v-icon color="red" class="mr-2">mdi-close-circle</v-icon>
            HIGH Risk
          </v-card-title>
          <v-card-text class="text-center">
            <div class="text-h3 font-weight-bold text-red">{{ stats.highRisk }}</div>
            <div class="text-caption">listings flagged urgent</div>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>

    <!-- Recent Activity -->
    <v-row>
      <v-col cols="12" md="6">
        <v-card rounded="lg">
          <v-card-title class="text-subtitle-1">
            <v-icon class="mr-2">mdi-history</v-icon>
            Recent Listings
          </v-card-title>
          <v-list density="compact">
            <v-list-item
              v-for="listing in recentListings"
              :key="listing.listing_id"
              :subtitle="formatDate(listing.listed_at)"
            >
              <template v-slot:title>
                <span class="text-body-2">
                  {{ listing.crop_type }} — {{ listing.quantity }} bags
                </span>
              </template>
              <template v-slot:append>
                <v-chip :color="statusColor(listing.status)" size="x-small">
                  {{ listing.status }}
                </v-chip>
              </template>
            </v-list-item>
            <v-list-item v-if="recentListings.length === 0">
              <v-list-item-title class="text-grey">No listings yet</v-list-item-title>
            </v-list-item>
          </v-list>
        </v-card>
      </v-col>

      <v-col cols="12" md="6">
        <v-card rounded="lg">
          <v-card-title class="text-subtitle-1">
            <v-icon class="mr-2">mdi-account-plus</v-icon>
            Recent Farmers
          </v-card-title>
          <v-list density="compact">
            <v-list-item
              v-for="farmer in recentFarmers"
              :key="farmer.phone_number"
              :subtitle="farmer.phone_number"
            >
              <template v-slot:title>
                <span class="text-body-2">{{ farmer.name || 'Unknown' }}</span>
              </template>
              <template v-slot:append>
                <span class="text-caption text-grey">{{ farmer.location }}</span>
              </template>
            </v-list-item>
            <v-list-item v-if="recentFarmers.length === 0">
              <v-list-item-title class="text-grey">No farmers yet</v-list-item-title>
            </v-list-item>
          </v-list>
        </v-card>
      </v-col>
    </v-row>

    <!-- Refresh button -->
    <v-row class="mt-4">
      <v-col class="text-center">
        <v-btn
          color="green"
          variant="tonal"
          prepend-icon="mdi-refresh"
          @click="fetchAll"
          :loading="loading"
        >
          Refresh Data
        </v-btn>
        <div class="text-caption text-grey mt-2">Last updated: {{ lastUpdated }}</div>
      </v-col>
    </v-row>
  </v-container>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { supabase } from '../supabase'

const loading = ref(false)
const lastUpdated = ref('Never')

const stats = ref({
  totalFarmers: 0,
  totalListings: 0,
  pendingListings: 0,
  totalVerifications: 0,
  lowRisk: 0,
  mediumRisk: 0,
  highRisk: 0
})

const recentListings = ref([])
const recentFarmers = ref([])

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

const formatDate = (date) => {
  if (!date) return 'N/A'
  return new Date(date).toLocaleDateString('en-KE', {
    day: 'numeric',
    month: 'short',
    hour: '2-digit',
    minute: '2-digit'
  })
}

const fetchAll = async () => {
  loading.value = true

  const [farmersRes, listingsRes, verificationsRes] = await Promise.all([
    supabase.from('farmers').select('*').order('registered_at', { ascending: false }),
    supabase.from('produce_listings').select('*').order('listed_at', { ascending: false }),
    supabase.from('iot_verifications').select('*')
  ])

  if (!farmersRes.error) {
    stats.value.totalFarmers = farmersRes.data.length
    recentFarmers.value = farmersRes.data.slice(0, 5)
  }

  if (!listingsRes.error) {
    stats.value.totalListings = listingsRes.data.length
    stats.value.pendingListings = listingsRes.data.filter(l => l.status === 'pending').length
    recentListings.value = listingsRes.data.slice(0, 5)
  }

  if (!verificationsRes.error) {
    stats.value.totalVerifications = verificationsRes.data.length
    stats.value.lowRisk = verificationsRes.data.filter(v => v.risk_level === 'LOW').length
    stats.value.mediumRisk = verificationsRes.data.filter(v => v.risk_level === 'MEDIUM').length
    stats.value.highRisk = verificationsRes.data.filter(v => v.risk_level === 'HIGH').length
  }

  lastUpdated.value = new Date().toLocaleTimeString('en-KE')
  loading.value = false
}

onMounted(fetchAll)
</script>