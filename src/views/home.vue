<template>
  <div class="home">
    <Header />
    <main>
      <!-- Contenido principal de tu webdoc -->
      <section>
        <h2>Introducción</h2>
        <p>¿La Laguna de Zumpango está seca debido a la acumulación de lirio acuático?</p>
        <!-- Más contenido -->
      </section>
    </main>
  </div>
</template>

<script>
import Header from '@/components/Header.vue';

export default {
  name: 'Home',
  components: {
    Header,
  },
};
</script>

<style scoped>
.home {
  padding: 20px;
}
</style>
