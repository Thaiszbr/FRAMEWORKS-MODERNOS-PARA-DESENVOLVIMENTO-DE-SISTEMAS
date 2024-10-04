<template>
  <div>
    <h1>Lista de Eventos</h1>
    <v-container>
      <v-row>
        <v-col
          v-for="event in sortedEvents"
          :key="event.id"
          cols="12"
          md="4"
        >
          <v-card @click="openDialog(event)" class="event-card">
            <v-card-title>{{ event.nomeEvento }}</v-card-title>
            <v-card-subtitle>
              Data: {{ formatDate(event.dataEvento) }}<br />
              Local: {{ event.localEvento }}<br />
              Participantes: {{ getParticipantsCount(event.id) }}
            </v-card-subtitle>
          </v-card>
        </v-col>
      </v-row>
      <v-col v-if="!events.length" cols="12">
        <v-card>
          <v-card-title>Nenhum evento encontrado.</v-card-title>
        </v-card>
      </v-col>
    </v-container>

    <!-- Dialog para mostrar detalhes do evento -->
    <v-dialog v-model="dialog" max-width="800px">
      <v-card>
        <v-card-title>
          <span class="headline">{{ selectedEvent.nomeEvento }}</span>
        </v-card-title>
        <v-card-subtitle>
          Data: {{ formatDate(selectedEvent.dataEvento) }}<br />
          Local: {{ selectedEvent.localEvento }}<br />
        </v-card-subtitle>
        <v-card-text>
          <h4>Descrição</h4>
          <v-card>
            <v-card-text>{{ selectedEvent.descricaoEvento }}</v-card-text>
          </v-card>

          <h4>Participantes</h4>
          <v-container>
            <v-row>
              <v-col
                v-for="(participant, index) in getParticipants(selectedEvent.id)"
                :key="index"
                cols="12"
                md="4"
              >
                <v-card>
                  <v-card-text>{{ participant }}</v-card-text>
                </v-card>
              </v-col>
            </v-row>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <v-btn color="primary" @click="dialog = false">Fechar</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'

const events = ref([]) // Lista de eventos
const participantsList = ref([]) // Lista de participantes
const dialog = ref(false) // Controle do dialog
const selectedEvent = ref({}) // Evento selecionado

// Função para carregar eventos e participantes do localStorage
const loadEvents = () => {
  const storedEvents = JSON.parse(localStorage.getItem('events')) || []
  const storedParticipants = JSON.parse(localStorage.getItem('participants')) || []

  events.value = storedEvents
  participantsList.value = storedParticipants
}

// Ordena os eventos pelo número de participantes
const sortedEvents = computed(() => {
  return events.value.sort((a, b) => getParticipantsCount(b.id) - getParticipantsCount(a.id))
})

// Retorna a contagem de participantes para um evento específico
const getParticipantsCount = (eventId) => {
  const participants = participantsList.value.find(participant => participant.eventId === eventId)
  return participants ? participants.participants.length : 0
}

// Retorna a lista de participantes para um evento específico
const getParticipants = (eventId) => {
  const participants = participantsList.value.find(participant => participant.eventId === eventId)
  return participants ? participants.participants : []
}

// Função para formatar a data no formato DD/MM/AAAA
const formatDate = (dateString) => {
  if (!dateString) return 'Data inválida';
  const [year, month, day] = dateString.split('-'); // Converte do formato AAAA-MM-DD para DD/MM/AAAA
  return `${day}/${month}/${year}`;
}

// Função chamada ao montar o componente
onMounted(() => {
  loadEvents()
})

// Abre o dialog com os detalhes do evento
const openDialog = (event) => {
  selectedEvent.value = event // Define o evento selecionado
  dialog.value = true // Abre o dialog
}
</script>

<style scoped>
/* Estilos adicionais para o seu componente, se necessário */
.event-card {
  cursor: pointer; /* Adiciona um cursor de ponteiro ao passar o mouse */
  margin-bottom: 20px; /* Espaçamento entre os cartões */
}
h4 {
  margin-top: 20px;
}
</style>
