<template>
  <div>
    <v-list>
      <v-list-item-group>
        <v-list-item v-for="event in events" :key="event.id">
          <v-list-item-content>
            <v-list-item-title>{{ event.nomeEvento }}</v-list-item-title>
            <v-list-item-subtitle>{{ formatDate(event.dataEvento) }} - {{ event.localEvento }}</v-list-item-subtitle>
          </v-list-item-content>
          <v-list-item-action>
            <v-btn @click="openEditDialog(event)">Editar</v-btn>
          </v-list-item-action>
        </v-list-item>
      </v-list-item-group>
    </v-list>

    <v-dialog v-model="dialog" max-width="600px">
      <v-card>
        <v-card-title>
          <span class="headline">Editar Evento</span>
        </v-card-title>
        <v-card-text>
          <v-text-field v-model="editedEvent.nomeEvento" label="Nome do Evento" required></v-text-field>
          <!-- Campo de data sem formatação adicional -->
          <v-text-field v-model="editedEvent.dataEvento" label="Data do Evento (AAAA-MM-DD)" type="date" required></v-text-field>
          <v-text-field v-model="editedEvent.localEvento" label="Local do Evento" required></v-text-field>
          <v-textarea v-model="editedEvent.descricaoEvento" label="Descrição do Evento" rows="3"></v-textarea>

          <h3>Participantes</h3>
          <v-list>
            <v-list-item v-for="(participant, index) in editedEvent.participants" :key="index">
              <v-list-item-content>{{ participant }}</v-list-item-content>
              <v-list-item-action>
                <v-btn @click="removeParticipant(index)">Remover</v-btn>
              </v-list-item-action>
            </v-list-item>
          </v-list>

          <v-text-field v-model="newParticipant" label="Adicionar Participante"></v-text-field>
          <v-btn @click="addParticipant">Adicionar</v-btn>
        </v-card-text>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="green" @click="saveEdit">Salvar</v-btn>
          <v-btn @click="closeDialog">Cancelar</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const events = ref([]) // Lista de eventos
const dialog = ref(false) // Controle do diálogo
const editedEvent = ref({}) // Evento sendo editado
const newParticipant = ref('') // Nome do novo participante

// Função para carregar eventos e participantes do localStorage
const loadEvents = () => {
  const storedEvents = JSON.parse(localStorage.getItem('events')) || []
  const storedParticipants = JSON.parse(localStorage.getItem('participants')) || []

  // Associando participantes a cada evento
  events.value = storedEvents.map(event => {
    const participantData = storedParticipants.find(p => p.eventId === event.id) || { participants: [] }
    return {
      ...event,
      participants: participantData.participants,
    }
  })
}

// Função para abrir o diálogo de edição
const openEditDialog = (event) => {
  editedEvent.value = { ...event } // Copiando o evento para edição
  dialog.value = true
}

// Função para fechar o diálogo
const closeDialog = () => {
  dialog.value = false
}

// Função para adicionar um participante
const addParticipant = () => {
  if (newParticipant.value) {
    editedEvent.value.participants.push(newParticipant.value)
    newParticipant.value = ''
  }
}

// Função para remover um participante
const removeParticipant = (index) => {
  editedEvent.value.participants.splice(index, 1)
}

// Função para salvar as edições
const saveEdit = () => {
  // Atualizar evento no localStorage
  const storedEvents = JSON.parse(localStorage.getItem('events')) || []
  const storedParticipants = JSON.parse(localStorage.getItem('participants')) || []

  // Validar a data antes de tentar salvar
  const isoDate = editedEvent.value.dataEvento; // A data já está no formato correto (AAAA-MM-DD)

  if (!isoDate) {
    alert("Data inválida! Por favor, selecione uma data válida.");
    return; // Impede o salvamento se a data for inválida
  }

  // Atualizando o evento
  const updatedEvents = storedEvents.map(event => {
    if (event.id === editedEvent.value.id) {
      return {
        ...event,
        nomeEvento: editedEvent.value.nomeEvento,
        dataEvento: isoDate, // Mantém a data no formato AAAA-MM-DD
        localEvento: editedEvent.value.localEvento,
        descricaoEvento: editedEvent.value.descricaoEvento,
      }
    }
    return event
  })

  // Atualizando participantes
  const updatedParticipants = storedParticipants.map(participant => {
    if (participant.eventId === editedEvent.value.id) {
      return {
        eventId: editedEvent.value.id,
        participants: editedEvent.value.participants,
      }
    }
    return participant
  })

  // Salvando de volta no localStorage
  localStorage.setItem('events', JSON.stringify(updatedEvents))
  localStorage.setItem('participants', JSON.stringify(updatedParticipants))

  // Atualiza a lista de eventos na interface sem recarregar a página
  loadEvents()

  // Fechar o diálogo após salvar
  closeDialog()
}

// Função para formatar a data no formato DD/MM/AAAA
const formatDate = (dateString) => {
  if (!dateString) return 'Data inválida';
  const [year, month, day] = dateString.split('-'); // Converte do formato AAAA-MM-DD para DD/MM/AAAA
  return `${day}/${month}/${year}`;
}

// Carregar eventos ao montar o componente
onMounted(loadEvents)
</script>
