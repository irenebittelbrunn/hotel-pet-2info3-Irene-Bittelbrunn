<script setup>
import { computed, onMounted, ref } from 'vue';
import { RouterLink, useRoute, useRouter } from 'vue-router';

const API_URL = 'http://localhost:3000';
const route = useRoute();
const router = useRouter();
const tutorId = route.params.id;
const tutor = ref(null);
const pets = ref([]);
const nomePet = ref('');
const especiePet = ref('');
const mensagem = ref('');
const enviando = ref(false);
const nomeInvalido = computed(() => !nomePet.value.trim());
const especieInvalida = computed(() => !especiePet.value);

async function carregarDados() {
  try {
    const [respostaTutor, respostaPets] = await Promise.all([
      fetch(`${API_URL}/tutores/${tutorId}`),
      fetch(`${API_URL}/pets?tutorId=${tutorId}`),
    ]);
    if (!respostaTutor.ok) throw new Error('Tutor não encontrado.');
    tutor.value = await respostaTutor.json();
    pets.value = respostaPets.ok ? await respostaPets.json() : [];
  } catch {
    mensagem.value = 'Não foi possível carregar os dados deste tutor.';
  }
}

async function salvarPet() {
  if (!nomePet.value.trim() || !especiePet.value) return;
  enviando.value = true;
  try {
    const resposta = await fetch(`${API_URL}/pets`, {
      method: 'POST', headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ nome: nomePet.value.trim(), especie: especiePet.value, tutorId: String(tutorId) }),
    });
    if (!resposta.ok) throw new Error();
    nomePet.value = '';
    especiePet.value = '';
    await carregarDados();
  } catch {
    alert('Erro ao cadastrar pet. Confira se o JSON Server está rodando.');
  } finally {
    enviando.value = false;
  }
}

async function excluirTutor() {
  if (!confirm('Deseja realmente remover este tutor?')) return;
  if (pets.value.length > 0) {
    alert('Este tutor possui pets vinculados. Remova ou transfira os pets antes de excluir o tutor.');
    return;
  }
  const resposta = await fetch(`${API_URL}/tutores/${tutorId}`, { method: 'DELETE' });
  if (resposta.ok) router.push('/tutores');
  else alert('Não foi possível remover o tutor.');
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
  <div class="d-flex justify-content-between align-items-start flex-wrap gap-3 mb-4">
    <div><h1 class="h2 fw-bold mb-1">Detalhes do tutor</h1><p class="text-secondary mb-0">Informações e pets vinculados a este tutor.</p></div>
    <RouterLink class="btn btn-outline-secondary" to="/tutores">Voltar para tutores</RouterLink>
  </div>
  <div v-if="mensagem" class="alert alert-warning">{{ mensagem }}</div>
  <section v-if="tutor" class="card card-body p-4 mb-4">
    <h2 class="h5 mb-3">Dados do tutor</h2>
    <p class="mb-2"><strong>ID:</strong> {{ tutor.id }}</p>
    <p class="mb-2"><strong>Nome completo:</strong> {{ tutor.nome }}</p>
    <p class="mb-2"><strong>Telefone:</strong> {{ tutor.telefone }}</p>
    <p v-if="tutor.cpfCnpj" class="mb-2"><strong>CPF / CNPJ:</strong> {{ tutor.cpfCnpj }}</p>
    <p v-if="tutor.email" class="mb-2"><strong>E-mail:</strong> {{ tutor.email }}</p>
    <p v-if="tutor.dataNascimento" class="mb-2"><strong>Data de nascimento:</strong> {{ tutor.dataNascimento }}</p>
    <h3 v-if="tutor.rua || tutor.cep || tutor.cidade || tutor.bairro" class="h6 fw-bold mt-3">Endereço</h3>
    <div v-if="tutor.rua || tutor.cep || tutor.cidade || tutor.bairro" class="row g-2 mb-3">
      <div v-if="tutor.rua" class="col-md-6"><strong>Rua:</strong> {{ tutor.rua }}</div>
      <div v-if="tutor.numero" class="col-md-2"><strong>Número:</strong> {{ tutor.numero }}</div>
      <div v-if="tutor.bairro" class="col-md-4"><strong>Bairro:</strong> {{ tutor.bairro }}</div>
      <div v-if="tutor.cidade" class="col-md-4"><strong>Cidade:</strong> {{ tutor.cidade }}</div>
      <div v-if="tutor.estado" class="col-md-4"><strong>Estado:</strong> {{ tutor.estado }}</div>
      <div v-if="tutor.cep" class="col-md-4"><strong>CEP:</strong> {{ tutor.cep }}</div>
    </div>
    <div><button class="btn btn-outline-danger btn-sm" @click="excluirTutor">Excluir tutor</button></div>
  </section>

  <section class="mb-4">
    <h2 class="h4 fw-bold mb-3">Pets vinculados</h2>
    <div v-if="pets.length === 0" class="alert alert-info">Nenhum pet vinculado a este tutor.</div>
    <div v-else class="table-responsive">
      <table class="table table-striped table-hover align-middle">
        <thead class="table-light"><tr><th>ID</th><th>Nome</th><th>Espécie</th><th>Ação</th></tr></thead>
        <tbody><tr v-for="pet in pets" :key="pet.id"><td>{{ pet.id }}</td><td>{{ pet.nome }}</td><td>{{ pet.especie }}</td><td><button @click="excluirPet(pet.id)">
      Excluir
    </button></td></tr></tbody>
      </table>
    </div>
  </section>

  <section class="card card-body p-4">
    <h2 class="h4 fw-bold mb-3">Cadastrar pet para este tutor</h2>
    <form @submit.prevent="salvarPet" novalidate>
      <div class="mb-3"><label for="nomePet" class="form-label">Nome do pet *</label><input id="nomePet" v-model="nomePet" class="form-control" placeholder="Ex.: Luna" required /></div>
      <div class="mb-3"><label for="especiePet" class="form-label">Espécie *</label><select id="especiePet" v-model="especiePet" class="form-select" required><option value="">Selecione a espécie</option><option value="Cachorro">Cachorro</option><option value="Gato">Gato</option><option value="Pássaro">Pássaro</option><option value="Outro">Outro</option></select></div>
      <button class="btn btn-primary" type="submit" :disabled="enviando || nomeInvalido || especieInvalida">{{ enviando ? 'Salvando...' : 'Salvar pet' }}</button>
    </form>
  </section>
</template>
