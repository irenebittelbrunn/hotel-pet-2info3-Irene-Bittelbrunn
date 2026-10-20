<script setup>
import { computed, onMounted, ref } from 'vue';
import { RouterLink } from 'vue-router';

const API_URL = 'http://localhost:3000';
const tutores = ref([]);
const pets = ref([]);
const busca = ref('');
const mensagem = ref('');
const carregando = ref(false);

const tutoresFiltrados = computed(() => {
  const termo = busca.value.trim().toLocaleLowerCase('pt-BR');
  if (!termo) return tutores.value;
  return tutores.value.filter((tutor) =>
    [tutor.nome, tutor.telefone, tutor.email, tutor.cpfCnpj]
      .some((valor) => String(valor ?? '').toLocaleLowerCase('pt-BR').includes(termo)),
  );
});

async function carregarDados() {
  carregando.value = true;
  mensagem.value = '';
  try {
    const [respostaTutores, respostaPets] = await Promise.all([
      fetch(`${API_URL}/tutores`),
      fetch(`${API_URL}/pets`),
    ]);
    if (!respostaTutores.ok || !respostaPets.ok) throw new Error();
    tutores.value = await respostaTutores.json();
    pets.value = await respostaPets.json();
  } catch {
    mensagem.value = 'Não foi possível carregar os dados. Confira se o JSON Server está rodando na porta 3000.';
  } finally {
    carregando.value = false;
  }
}

function petsDoTutor(tutorId) {
  return pets.value.filter((pet) => String(pet.tutorId) === String(tutorId));
}

function formatarTelefone(valor) {
  const numeros = String(valor ?? '').replace(/\D/g, '');
  if (numeros.length === 11) return `(${numeros.slice(0, 2)}) ${numeros.slice(2, 7)}-${numeros.slice(7)}`;
  if (numeros.length === 10) return `(${numeros.slice(0, 2)}) ${numeros.slice(2, 6)}-${numeros.slice(6)}`;
  return valor || '—';
}

async function excluirTutor(tutor) {
  if (!confirm('Deseja realmente remover este tutor?')) return;
  if (petsDoTutor(tutor.id).length > 0) {
    alert('Este tutor possui pets vinculados. Remova ou transfira os pets antes de excluir o tutor.');
    return;
  }
  try {
    const resposta = await fetch(`${API_URL}/tutores/${tutor.id}`, { method: 'DELETE' });
    if (!resposta.ok) throw new Error();
    await carregarDados();
  } catch {
    alert('Não foi possível remover o tutor. Tente novamente.');
  }
}

onMounted(carregarDados);
</script>

<template>
  <div class="d-flex flex-wrap justify-content-between align-items-start gap-3 mb-4">
    <div>
      <h1 class="h2 fw-bold mb-1">Listagem de Tutores</h1>
      <p class="text-secondary mb-0">Consulte os tutores cadastrados e seus pets.</p>
    </div>
    <RouterLink class="btn btn-primary" to="/tutores/novo"><i class="bi bi-plus-lg me-2"></i>Cadastrar Tutor</RouterLink>
  </div>

  <div class="mb-3" style="max-width: 680px">
    <label for="buscaTutor" class="visually-hidden">Pesquisar tutor</label>
    <div class="input-group">
      <span class="input-group-text bg-white"><i class="bi bi-search"></i></span>
      <input id="buscaTutor" v-model="busca" class="form-control" placeholder="Pesquisar tutor por nome, CPF/CNPJ, telefone ou e-mail" />
    </div>
  </div>

  <div v-if="mensagem" class="alert alert-warning">{{ mensagem }}</div>
  <div class="card border-0 shadow-sm overflow-hidden">
    <div class="table-responsive">
      <table class="table table-striped table-hover align-middle mb-0">
        <thead class="table-light"><tr><th>ID</th><th>Nome completo</th><th>CPF / CNPJ</th><th>Telefone</th><th>E-mail</th><th class="text-center">Pets (quantidade)</th><th>Nomes dos pets</th><th>Ações</th></tr></thead>
        <tbody>
          <tr v-for="tutor in tutoresFiltrados" :key="tutor.id">
            <td>{{ tutor.id }}</td>
            <td class="fw-semibold">{{ tutor.nome }}</td>
            <td>{{ tutor.cpfCnpj || '—' }}</td>
            <td>{{ formatarTelefone(tutor.telefone) }}</td>
            <td>{{ tutor.email || '—' }}</td>
            <td class="text-center"><span class="badge rounded-pill text-bg-primary">{{ petsDoTutor(tutor.id).length }}</span></td>
            <td>{{ petsDoTutor(tutor.id).map((pet) => pet.nome).join(' / ') || 'Nenhum pet vinculado' }}</td>
            <td class="text-nowrap">
              <RouterLink class="btn btn-sm btn-outline-primary me-2" :to="`/tutores/${tutor.id}`"><i class="bi bi-eye me-1"></i>Visualizar</RouterLink>
              <button class="btn btn-sm btn-outline-danger" @click="excluirTutor(tutor)"><i class="bi bi-trash me-1"></i>Remover</button>
            </td>
          </tr>
          <tr v-if="!carregando && tutoresFiltrados.length === 0"><td colspan="8" class="text-center text-secondary py-4">{{ busca ? 'Nenhum tutor corresponde à pesquisa.' : 'Nenhum tutor cadastrado.' }}</td></tr>
          <tr v-if="carregando"><td colspan="8" class="text-center text-secondary py-4">Carregando tutores...</td></tr>
        </tbody>
      </table>
    </div>
  </div>
  <p class="text-secondary small mt-3 mb-0">Exibindo {{ tutoresFiltrados.length }} de {{ tutores.length }} tutor(es).</p>
</template>
