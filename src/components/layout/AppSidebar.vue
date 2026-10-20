<script setup>
import { useRoute } from 'vue-router';
const route = useRoute();
</script>

<template>
  <nav aria-label="Menu da PetSchool">
    <div class="nav flex-column nav-pills gap-1">
      <RouterLink class="nav-link" :class="{ active: route.path.startsWith('/pets') && route.path !== '/pets/novo' }" to="/pets"><i class="bi bi-heart me-2" aria-hidden="true"></i>Pets</RouterLink>
      <RouterLink class="nav-link" :class="{ active: route.path === '/pets/novo' }" to="/pets/novo"><i class="bi bi-plus-circle me-2" aria-hidden="true"></i>Novo pet</RouterLink>
      <RouterLink class="nav-link" :class="{ active: route.path === '/tutores' || (route.path.startsWith('/tutores/') && route.path !== '/tutores/novo') }" to="/tutores"><i class="bi bi-people me-2" aria-hidden="true"></i>Tutores</RouterLink>
      <RouterLink class="nav-link" :class="{ active: route.path === '/tutores/novo' }" to="/tutores/novo"><i class="bi bi-person-plus me-2" aria-hidden="true"></i>Novo tutor</RouterLink>
    </div>
  </nav>
</template>
