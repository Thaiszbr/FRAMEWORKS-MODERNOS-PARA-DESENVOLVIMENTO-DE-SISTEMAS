<template>
  <div>
    <form @submit.prevent="submit">
      <!-- Nome do Evento -->
      <v-text-field
        v-model="state.nomeEvento"
        label="Nome do Evento"
        required
        :error-messages="v$.nomeEvento.$errors.map(e => e.$message)"
        @blur="v$.nomeEvento.$touch"
        @input="v$.nomeEvento.$touch"
      ></v-text-field>

      <!-- Data do Evento -->
      <v-text-field
        v-model="state.dataEvento"
        label="Data do Evento (DD/MM/AAAA)"
        required
        type="date"
        :error-messages="v$.dataEvento.$errors.map(e => e.$message)"
        @blur="v$.dataEvento.$touch"
        @input="v$.dataEvento.$touch"
      ></v-text-field>

      <!-- Local do Evento -->
      <v-text-field
        v-model="state.localEvento"
        label="Local do Evento"
        required
        :error-messages="v$.localEvento.$errors.map(e => e.$message)"
        @blur="v$.localEvento.$touch"
        @input="v$.localEvento.$touch"
      ></v-text-field>

      <!-- Descrição do Evento -->
      <v-textarea
        v-model="state.descricaoEvento"
        label="Descrição do Evento"
        rows="3"
        required
        :error-messages="v$.descricaoEvento.$errors.map(e => e.$message)"
        @blur="v$.descricaoEvento.$touch"
        @input="v$.descricaoEvento.$touch"
      ></v-textarea>

      <!-- Adicionar Participantes -->
      <h3>Adicionar Participantes</h3>
      <v-text-field
        v-model="participantName"
        label="Nome do Participante"
      ></v-text-field>

      <v-btn @click="addParticipant">
        Adicionar Participante
      </v-btn>

      <v-list v-if="participants.length" class="participant-list">
        <v-list-item v-for="(participant, index) in participants" :key="index">
          <v-list-item-content>
            {{ participant }}
          </v-list-item-content>
          <v-btn icon @click="removeParticipant(index)">
            <v-icon>mdi-delete</v-icon>
          </v-btn>
        </v-list-item>
      </v-list>
    </form>

    <!-- Botão de Submissão Fixo -->
    <div class="submit-button-container">
      <v-btn class="me-4" type="submit" @click="submit">
        Cadastrar Evento
      </v-btn>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import useVuelidate from '@vuelidate/core'
import { required, minLength } from '@vuelidate/validators'
import { v4 as uuidv4 } from 'uuid' // Usando para gerar IDs únicos

// Estado inicial
const state = ref({
  nomeEvento: '',
  dataEvento: '',
  localEvento: '',
  descricaoEvento: ''
})

const participants = ref([]) // Lista de participantes
const participantName = ref('') // Nome do participante

// Validador de data
const dateValidator = (value) => {
  const regex = /^\d{2}\/\d{2}\/\d{4}$/; // Verifica se a data está no formato DD/MM/AAAA
  return regex.test(value) || 'Formato de data inválido (DD/MM/AAAA)';
}

// Regras de validação
const rules = {
  nomeEvento: { required },
  dataEvento: { required, dateValidator }, // Usando a validação personalizada
  localEvento: { required },
  descricaoEvento: { required }
}

const v$ = useVuelidate(rules, state)

// Função para adicionar participante
const addParticipant = () => {
  if (participantName.value) {
    participants.value.push(participantName.value)
    participantName.value = ''
  }
}

// Função para remover participante
const removeParticipant = (index) => {
  participants.value.splice(index, 1)
}

// Função para formatar a data para o formato BR (DD/MM/AAAA)
const formatDateToBR = (date) => {
  const [year, month, day] = date.split('-');
  return `${day}/${month}/${year}`;
}

// Função para salvar evento e participantes no localStorage
const saveToLocalStorage = (eventDetails) => {
  // Salvando eventos no localStorage
  let events = JSON.parse(localStorage.getItem('events')) || []
  events.push(eventDetails)
  localStorage.setItem('events', JSON.stringify(events))

  // Salvando participantes separados, vinculados ao eventID
  let participantsList = JSON.parse(localStorage.getItem('participants')) || []
  participantsList.push({
    eventId: eventDetails.id,
    participants: participants.value
  })
  localStorage.setItem('participants', JSON.stringify(participantsList))
}

// Função de submissão do formulário
const submit = () => {
  v$.value.$touch()

  if (!v$.value.$invalid) {
    const eventId = uuidv4(); // Gerando um ID único para o evento

    // Formatando a data antes de criar o objeto de evento
    const formattedDate = formatDateToBR(state.value.dataEvento);

    const eventDetails = {
      id: eventId,
      ...state.value,
      dataEvento: formattedDate // Certifique-se de que a data esteja formatada aqui
    };

    saveToLocalStorage(eventDetails); // Salvando evento e participantes no localStorage

    alert('Evento cadastrado com sucesso!');

    // Limpar formulário após cadastro
    state.value.nomeEvento = '';
    state.value.dataEvento = '';
    state.value.localEvento = '';
    state.value.descricaoEvento = '';
    participants.value = [];
  }
}
</script>

<style scoped>
.participant-list {
  margin-top: 10px; /* Margem entre a lista de participantes e o botão */
}

.submit-button-container {
  margin-top: 20px; /* Adicione espaço entre a lista de participantes e o botão de submissão */
}
</style>
