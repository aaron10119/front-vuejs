<template>
  <div id="app">
    <ListaCategorias />
    <ListaProducts />
  </div>
</template>

<script>
import ListaCategorias from './components/ListaCategorias.vue'; // Importamos el componente
import ListaProducts from './components/ListaProducts.vue';  // Importamos el componente de Productos


export default {
  components: {
    ListaCategorias,
    ListaProducts
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
</style>
