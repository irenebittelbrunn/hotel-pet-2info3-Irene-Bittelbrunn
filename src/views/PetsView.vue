<script setup>
import { onMounted, ref } from 'vue';

const API_URL = 'http://localhost:3000';
const pets = ref([]);
('');
const tutores = ref([]);
async function carregarDados() {
  const respostaPets = await fetch(`${API_URL}/pets`);
  pets.value = await respostaPets.json();

  const respostaTutores = await fetch(`${API_URL}/tutores`);
  tutores.value = await respostaTutores.json();
}

function nomeDoTutor(tutorId) {
  for (const tutor of tutores.value) {
    if (tutor.id === tutorId) {
      return tutor.nome;
    }
  }
  return 'Não Especificado';
}
onMounted(carregarDados);

async function excluirPet(id) {
  const confirmar = window.confirm(
    'Deseja realmente excluir este pet?'
  )

  if (!confirmar) return

  try {
    const resposta = await fetch(
      `http://localhost:3000/pets/${id}`,
      {
        method: 'DELETE'
      }
    )

    if (!resposta.ok) {
      throw new Error('Erro ao excluir pet')
    }

    // Atualiza a lista na tela
    pets.value = pets.value.filter(
      pet => pet.id !== id
    )
  } catch (erro) {
    alert('Não foi possível excluir o pet.')
  }
}
</script>

<template>
  <div>
    <header class="mb-4">
      <h1 class="text-2xl font-bold">Listagem de Pets</h1>
      <p class="text-body-secondary mb-0">
        Listagem dos Pets cadastrados no sistema.
      </p>
    </header>

    <RouterLink
      class="btn btn-primary"
      :to="{ name: 'addPet' }"
    >
      Adicionar Pet
    </RouterLink>
  </div>
  <table class="table table-striped table-hover">
    <thead>
      <tr>
        <th>Id</th>
        <th>Nome</th>
        <th>Espécie</th>
        <th>Tutor</th>
        <th>Ações</th>
      </tr>
    </thead>
    <tbody>
  <tr v-for="pet in pets" :key="pet.id">
  <td>{{ pet.id }}</td>
  <td>{{ pet.nome }}</td>
  <td>{{ pet.especie }}</td>
  <td>
    <button @click="excluirPet(pet.id)">
      Excluir
    </button>
  </td>
</tr>
    </tbody>
  </table>
</template>
<style>
.btn-excluir {
  background-color: #dc2626;
  color: white;
  border: none;
  padding: 8px 14px;
  border-radius: 5px;
  cursor: pointer;
}

.btn-excluir:hover {
  background-color: #b91c1c;
}
</style>