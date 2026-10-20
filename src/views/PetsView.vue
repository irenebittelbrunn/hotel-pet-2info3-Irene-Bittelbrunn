<script setup>
import { onMounted, ref } from 'vue';
import { RouterLink } from 'vue-router';

const API_URL = 'http://localhost:3000';

const pets = ref([]);
const tutores = ref([]);

async function carregarDados() {
  try {
    const [respostaPets, respostaTutores] = await Promise.all([
      fetch(`${API_URL}/pets`),
      fetch(`${API_URL}/tutores`),
    ]);

    if (!respostaPets.ok || !respostaTutores.ok) {
      throw new Error('Erro ao carregar os dados.');
    }

    pets.value = await respostaPets.json();
    tutores.value = await respostaTutores.json();
  } catch (erro) {
    console.error(erro);
    alert('Não foi possível carregar os pets e tutores.');
  }
}

function nomeDoTutor(tutorId) {
  const tutor = tutores.value.find(
    (tutor) => String(tutor.id) === String(tutorId)
  );

  return tutor ? tutor.nome : 'Não especificado';
}

async function excluirPet(id) {
  const confirmar = window.confirm(
    'Deseja realmente excluir este pet?'
  );

  if (!confirmar) return;

  try {
    const resposta = await fetch(`${API_URL}/pets/${id}`, {
      method: 'DELETE',
    });

    if (!resposta.ok) {
      throw new Error('Erro ao excluir pet.');
    }

    pets.value = pets.value.filter(
      (pet) => String(pet.id) !== String(id)
    );
  } catch (erro) {
    console.error(erro);
    alert('Não foi possível excluir o pet.');
  }
}

onMounted(carregarDados);
</script>

<template>
  <div>
    <header class="mb-4">
      <h1 class="text-2xl fw-bold">Listagem de Pets</h1>
      <p class="text-body-secondary mb-0">
        Listagem dos pets cadastrados no sistema.
      </p>
    </header>

    <RouterLink
      class="btn btn-primary mb-3"
      :to="{ name: 'addPet' }"
    >
      Adicionar Pet
    </RouterLink>

    <div class="table-responsive">
      <table class="table table-striped table-hover align-middle">
        <thead>
          <tr>
            <th>ID</th>
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
            <td>{{ nomeDoTutor(pet.tutorId) }}</td>
            <td>
              <div class="d-flex gap-2">
                <RouterLink
                  class="btn btn-outline-primary btn-sm"
                  :to="`/pets/${pet.id}`"
                >
                  Visualizar
                </RouterLink>

                <button
                  type="button"
                  class="btn btn-outline-danger btn-sm"
                  @click="excluirPet(pet.id)"
                >
                  Excluir
                </button>
              </div>
            </td>
          </tr>

          <tr v-if="pets.length === 0">
            <td colspan="5" class="text-center text-secondary py-4">
              Nenhum pet cadastrado.
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>
```