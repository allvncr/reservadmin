<template>
  <q-page class="bg-grey-2">
    <!-- Header -->
    <q-toolbar class="bg-primary text-white">
      <q-btn flat icon="arrow_back" color="white" @click="goBack" />
      <q-toolbar-title>Gestion des Réservations</q-toolbar-title>
    </q-toolbar>

    <!-- Tableau des réservations -->
    <q-card-section class="q-pa-md">
      <q-table
        flat
        bordered
        title="Liste des Réservations"
        :rows="reservations"
        :columns="columns"
        row-key="_id"
        :loading="loading"
      >
        <template v-slot:body-cell-status="props">
          <q-td :props="props">
            <q-badge
              :color="getStatusColor(props.row.status)"
              text-color="white"
              align="top"
              rounded
            >
              {{ props.row.status }}
            </q-badge>
          </q-td>
        </template>
        <template v-slot:body-cell-actions="props">
          <q-td align="right">
            <q-btn
              icon="visibility"
              color="primary"
              flat
              dense
              @click="openReservationDetail(props.row)"
            />
          </q-td>
        </template>
      </q-table>
    </q-card-section>

    <!-- Détails de réservation -->
    <q-dialog v-model="showReservationDetail" persistent full-width>
      <q-card style="max-width: 600px; margin: auto">
        <q-card-section>
          <div class="text-h6">Détails de la Réservation</div>
        </q-card-section>

        <q-separator />

        <!-- Infos client -->
        <q-card-section class="q-pb-none">
          <div class="text-subtitle2 text-weight-bold">👤 Client</div>
          <div>
            <strong>Nom :</strong> {{ selectedReservation?.user?.firstname }}
            {{ selectedReservation?.user?.lastname }}
          </div>
          <div><strong>Email :</strong> {{ selectedReservation?.user?.email }}</div>
          <div><strong>Téléphone :</strong> {{ selectedReservation?.user?.tel }}</div>
        </q-card-section>

        <q-separator />

        <!-- Infos réservation -->
        <q-card-section>
          <div>
            <strong>Établissement :</strong> {{ selectedReservation?.items[0].establishment.name }}
          </div>
          <div>
            <strong>Date :</strong>
            {{ formatDate(selectedReservation?.items[0].reservationStartDate) }}
          </div>
          <div v-if="selectedReservation?.items[0].reservationStartTime">
            <strong>Heure :</strong> {{ selectedReservation.items[0].reservationStartTime }}
          </div>
          <div><strong>Personnes :</strong> {{ selectedReservation?.items[0].people }}</div>
          <div><strong>Prix :</strong> {{ selectedReservation?.items[0].price }} FCFA</div>

          <div v-if="selectedReservation?.items[0].menu?.length">
            <strong>Plats commandés :</strong>
            <ul>
              <li v-for="(dish, index) in selectedReservation.items[0].menu" :key="index">
                {{ dish.name }} x{{ dish.quantity }} - {{ dish.price }} FCFA
              </li>
            </ul>
          </div>

          <div><strong>Statut :</strong> {{ selectedReservation.status }}</div>
        </q-card-section>

        <q-separator />

        <!-- Boutons -->
        <q-card-actions align="right">
          <q-btn flat label="Fermer" color="grey" v-close-popup />
          <q-btn
            v-if="selectedReservation.status === 'EnCours'"
            label="Annuler"
            color="negative"
            @click="cancelReservation(selectedReservation._id)"
          />
          <q-btn
            v-if="selectedReservation.status === 'EnCours'"
            label="Confirmer"
            color="positive"
            @click="confirmReservation(selectedReservation._id)"
          />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </q-page>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { useReservationStore } from 'src/stores/reservationStore'
import { date } from 'quasar'

const router = useRouter()
const reservationStore = useReservationStore()
const reservations = ref([])
const loading = ref(false)

const showReservationDetail = ref(false)
const selectedReservation = ref(null)

const goBack = () => {
  router.go(-1)
}

const columns = [
  {
    name: 'user',
    label: 'Client',
    field: (row) => `${row.user.firstname} ${row.user.lastname}`,
    align: 'left',
  },
  {
    name: 'type',
    label: 'Type établissement',
    field: (row) => row.items[0]?.establishment?.type || '—',
    align: 'left',
  },
  {
    name: 'date',
    label: 'Date',
    field: (row) => formatDate(row.items[0]?.reservationStartDate),
    align: 'left',
  },
  { name: 'status', label: 'Statut', field: 'status', align: 'left' },
  { name: 'totalPrice', label: 'Total (FCFA)', field: 'totalPrice', align: 'left' },
  { name: 'actions', label: 'Actions', align: 'center' },
]

const formatDate = (value) => {
  return value ? date.formatDate(value, 'DD/MM/YYYY') : '—'
}

const init = async () => {
  loading.value = true
  await reservationStore.fetchAllReservations()
  reservations.value = reservationStore.reservations
  loading.value = false
}

const openReservationDetail = (reservation) => {
  selectedReservation.value = reservation
  showReservationDetail.value = true
}

const cancelReservation = async (id) => {
  await reservationStore.cancelReservation(id)
  showReservationDetail.value = false
  init()
}

const confirmReservation = async (id) => {
  await reservationStore.confirmReservation(id)
  showReservationDetail.value = false
  init()
}
const getStatusColor = (status) => {
  switch (status) {
    case 'Validée':
      return 'green'
    case 'Annulée':
      return 'red'
    case 'EnCours':
      return 'orange'
    default:
      return 'grey'
  }
}

onMounted(() => {
  init()
})
</script>

<style scoped>
.q-toolbar {
  padding: 0 1rem;
}
</style>
