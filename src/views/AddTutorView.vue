<script setup>
import { computed, ref } from 'vue';
import { RouterLink, useRouter } from 'vue-router';

const API_URL = 'http://localhost:3000/tutores';
const router = useRouter();
const routerCancel = '/tutores';
const form = ref({ nome: '', telefone: '', cpfCnpj: '', email: '', dataNascimento: '', rua: '', numero: '', bairro: '', cidade: '', estado: '', cep: '' });
const tentouSalvar = ref(false);
const enviando = ref(false);
const buscandoCep = ref(false);
const erroGeral = ref('');
const mensagemErro = 'Ops, campo obrigatório, por favor preencha-o!';
const digitosTelefone = computed(() => form.value.telefone.replace(/\D/g, ''));
const nomeInvalido = computed(() => !form.value.nome.trim());
const telefoneInvalido = computed(() => digitosTelefone.value.length < 10);
const emailInvalido = computed(() => !!form.value.email && !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(form.value.email));
const formularioValido = computed(() => !nomeInvalido.value && !telefoneInvalido.value && !emailInvalido.value);
const estados = [
  ['AC','Acre'],['AL','Alagoas'],['AP','Amapá'],['AM','Amazonas'],['BA','Bahia'],['CE','Ceará'],['DF','Distrito Federal'],['ES','Espírito Santo'],['GO','Goiás'],['MA','Maranhão'],['MT','Mato Grosso'],['MS','Mato Grosso do Sul'],['MG','Minas Gerais'],['PA','Pará'],['PB','Paraíba'],['PR','Paraná'],['PE','Pernambuco'],['PI','Piauí'],['RJ','Rio de Janeiro'],['RN','Rio Grande do Norte'],['RS','Rio Grande do Sul'],['RO','Rondônia'],['RR','Roraima'],['SC','Santa Catarina'],['SP','São Paulo'],['SE','Sergipe'],['TO','Tocantins']
];

function formatarTelefone(valor) {
  const n = valor.replace(/\D/g, '').slice(0, 11);
  if (!n) return '';
  if (n.length < 3) return `(${n}`;
  if (n.length <= 6) return `(${n.slice(0,2)}) ${n.slice(2)}`;
  if (n.length <= 10) return `(${n.slice(0,2)}) ${n.slice(2,6)}-${n.slice(6)}`;
  return `(${n.slice(0,2)}) ${n.slice(2,7)}-${n.slice(7)}`;
}
function formatarDocumento(valor) {
  const n = valor.replace(/\D/g, '').slice(0, 14);
  if (n.length <= 11) return n.replace(/(\d{3})(\d)/, '$1.$2').replace(/(\d{3})(\d)/, '$1.$2').replace(/(\d{3})(\d{1,2})$/, '$1-$2');
  return n.replace(/^(\d{2})(\d)/, '$1.$2').replace(/^(\d{2})\.(\d{3})(\d)/, '$1.$2.$3').replace(/\.(\d{3})(\d)/, '.$1/$2').replace(/(\d{4})(\d)/, '$1-$2');
}
function formatarCep(valor) {
  const n = valor.replace(/\D/g, '').slice(0, 8);
  return n.length > 5 ? `${n.slice(0,5)}-${n.slice(5)}` : n;
}
async function buscarCep() {
  const cep = form.value.cep.replace(/\D/g, '');
  if (cep.length !== 8) return;
  buscandoCep.value = true;
  erroGeral.value = '';
  try {
    const resposta = await fetch(`https://viacep.com.br/ws/${cep}/json/`);
    if (!resposta.ok) throw new Error();
    const dados = await resposta.json();
    if (dados.erro) {
      erroGeral.value = 'CEP não encontrado. Confira os números digitados.';
      return;
    }
    form.value.rua = dados.logradouro || '';
    form.value.bairro = dados.bairro || '';
    form.value.cidade = dados.localidade || '';
    form.value.estado = dados.uf || '';
  } catch {
    erroGeral.value = 'Não foi possível consultar o CEP. Você pode preencher o endereço manualmente.';
  } finally {
    buscandoCep.value = false;
  }
}
async function salvarTutor() {
  tentouSalvar.value = true;
  erroGeral.value = '';
  if (!formularioValido.value) return;
  enviando.value = true;
  const payload = {
    nome: form.value.nome.trim(),
    telefone: digitosTelefone.value,
    cpfCnpj: form.value.cpfCnpj.replace(/\D/g, ''),
    email: form.value.email.trim(),
    dataNascimento: form.value.dataNascimento,
    rua: form.value.rua.trim(),
    numero: form.value.numero.trim(),
    bairro: form.value.bairro.trim(),
    cidade: form.value.cidade.trim(),
    estado: form.value.estado,
    cep: form.value.cep.replace(/\D/g, ''),
  };
  try {
    const resposta = await fetch(API_URL, { method: 'POST', headers: { 'Content-Type': 'application/json' }, body: JSON.stringify(payload) });
    if (!resposta.ok) throw new Error();
    await router.push('/tutores');
  } catch {
    erroGeral.value = 'Não foi possível cadastrar o tutor. Confira se o JSON Server está rodando na porta 3000.';
  } finally {
    enviando.value = false;
  }
}
</script>

<template>
  <div class="mb-4">
    <h1 class="h2 fw-bold mb-1">Cadastrar Tutor</h1>
    <p class="text-secondary mb-0">Preencha os dados para cadastrar um novo tutor.</p>
  </div>
  <form class="card border-0 shadow-sm p-3 p-md-4" @submit.prevent="salvarTutor" novalidate>
    <h2 class="h5 fw-bold border-bottom pb-3 mb-3">Dados do Tutor</h2>
    <div class="row g-3">
      <div class="col-md-4"><label class="form-label" for="idTutor">ID</label><input id="idTutor" class="form-control bg-body-secondary" value="Gerado automaticamente" disabled /></div>
      <div class="col-md-8"><label class="form-label" for="nome">Nome completo <span class="text-danger">*</span></label><input id="nome" v-model="form.nome" class="form-control" :class="{'is-invalid': tentouSalvar && nomeInvalido}" placeholder="Digite o nome completo" autocomplete="name"/><div v-if="tentouSalvar && nomeInvalido" class="invalid-feedback">{{ mensagemErro }}</div></div>
      <div class="col-md-6"><label class="form-label" for="cpfCnpj">CPF / CNPJ</label><input id="cpfCnpj" :value="form.cpfCnpj" class="form-control" placeholder="Digite o CPF ou CNPJ" inputmode="numeric" @input="form.cpfCnpj = formatarDocumento($event.target.value)"/><div class="form-text">A máscara muda conforme a quantidade de números.</div></div>
      <div class="col-md-6"><label class="form-label" for="telefone">Telefone / Celular / WhatsApp <span class="text-danger">*</span></label><input id="telefone" :value="form.telefone" class="form-control" :class="{'is-invalid': tentouSalvar && telefoneInvalido}" placeholder="(00) 00000-0000" inputmode="tel" autocomplete="tel" @input="form.telefone = formatarTelefone($event.target.value)"/><div v-if="tentouSalvar && telefoneInvalido" class="invalid-feedback">{{ mensagemErro }}</div><div class="form-text">Formato: (00) 00000-0000</div></div>
      <div class="col-md-6"><label class="form-label" for="email">E-mail</label><input id="email" v-model.trim="form.email" type="email" class="form-control" :class="{'is-invalid': emailInvalido}" placeholder="nome@exemplo.com" autocomplete="email"/><div v-if="emailInvalido" class="invalid-feedback">Informe um e-mail válido.</div></div>
      <div class="col-md-6"><label class="form-label" for="dataNascimento">Data de nascimento</label><input id="dataNascimento" v-model="form.dataNascimento" type="date" class="form-control" autocomplete="bday"/></div>
    </div>
    <h2 class="h5 fw-bold border-bottom pb-3 mt-4 mb-3">Endereço</h2>
    <div class="row g-3">
      <div class="col-md-6"><label class="form-label" for="cep">CEP</label><input id="cep" :value="form.cep" class="form-control" placeholder="00000-000" inputmode="numeric" maxlength="9" autocomplete="postal-code" @input="form.cep = formatarCep($event.target.value)" @blur="buscarCep"/><div class="form-text">{{ buscandoCep ? 'Buscando endereço...' : 'Digite o CEP para preencher rua, bairro, cidade e estado automaticamente.' }}</div></div>
      <div class="col-md-4"><label class="form-label" for="rua">Rua</label><input id="rua" v-model="form.rua" class="form-control" placeholder="Digite o nome da rua" autocomplete="address-line1"/></div>
      <div class="col-md-2"><label class="form-label" for="numero">Número</label><input id="numero" v-model="form.numero" class="form-control" placeholder="Número"/></div>
      <div class="col-md-4"><label class="form-label" for="bairro">Bairro</label><input id="bairro" v-model="form.bairro" class="form-control" placeholder="Digite o bairro"/></div>
      <div class="col-md-4"><label class="form-label" for="cidade">Cidade</label><input id="cidade" v-model="form.cidade" class="form-control" placeholder="Digite a cidade" autocomplete="address-level2"/></div>
      <div class="col-md-4"><label class="form-label" for="estado">Estado</label><select id="estado" v-model="form.estado" class="form-select"><option value="">Selecione</option><option v-for="estado in estados" :key="estado[0]" :value="estado[0]">{{ estado[0] }} — {{ estado[1] }}</option></select></div>
    </div>
    <div v-if="erroGeral" class="alert alert-danger mt-4 mb-0">{{ erroGeral }}</div>
    <div class="alert alert-primary py-2 mt-4 mb-0 small"><i class="bi bi-info-circle-fill me-2"></i>Somente Nome e Telefone são obrigatórios para cadastrar o tutor.</div>
    <div class="d-flex flex-wrap justify-content-end gap-2 mt-3 pt-3 border-top"><RouterLink class="btn btn-outline-secondary" :to="routerCancel">Cancelar</RouterLink><button class="btn btn-primary" type="submit" :disabled="enviando || !formularioValido">{{ enviando ? 'Cadastrando...' : 'Cadastrar Tutor' }}</button></div>
  </form>
</template>
