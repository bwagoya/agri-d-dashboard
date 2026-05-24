<template>
  <v-app>
    <v-navigation-drawer v-model="drawer" permanent>
      <v-list-item
        title="Agri-D Ledger"
        subtitle="Admin Dashboard"
        prepend-icon="mdi-leaf"
        nav
      ></v-list-item>

      <v-divider></v-divider>

      <v-list density="compact" nav>
        <v-list-item
          v-for="item in navItems"
          :key="item.title"
          :prepend-icon="item.icon"
          :title="item.title"
          :value="item.title"
          :active="currentView === item.component"
          active-color="green"
          @click="currentView = item.component"
        ></v-list-item>
      </v-list>
    </v-navigation-drawer>

    <v-app-bar color="green" flat>
      <v-app-bar-title>
        <span class="text-white">Agri-D Ledger Dashboard</span>
      </v-app-bar-title>
      <template v-slot:append>
        <v-chip color="white" variant="tonal" class="mr-3">
          <v-icon start>mdi-circle</v-icon>
          Live
        </v-chip>
      </template>
    </v-app-bar>

    <v-main>
      <component :is="currentView" />
    </v-main>
  </v-app>
</template>

<script setup>
import { ref } from 'vue'
import OverviewView from './views/OverviewView.vue'
import FarmersView from './views/FarmersView.vue'
import ListingsView from './views/ListingsView.vue'
import VerificationsView from './views/VerificationsView.vue'
import SessionsView from './views/SessionsView.vue'

const drawer = ref(true)
const currentView = ref(OverviewView)

const navItems = [
  { title: 'Overview', icon: 'mdi-view-dashboard', component: OverviewView },
  { title: 'Farmers', icon: 'mdi-account-group', component: FarmersView },
  { title: 'Listings', icon: 'mdi-sprout', component: ListingsView },
  { title: 'IoT Verifications', icon: 'mdi-shield-check', component: VerificationsView },
  { title: 'Sessions', icon: 'mdi-history', component: SessionsView }
]
</script>