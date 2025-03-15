<template>
  <div id="app">
    <section>
      <div class="titulos">
        <h1>Prueba Desarrollador Full Stack</h1>
        <h5>Aarón Hernandez Bueno</h5>
      </div>
      <div class="container">
        <div class="row">
        <div class="col-sm-6 col-12">
          <ListaCategorias />
        </div>
        <div class="col-sm-6 col-12">
          <ListaProducts />
        </div>
      </div>
      </div>
    </section>
  </div>
</template>

<script>
import ListaCategorias from './components/ListaCategorias.vue'; 
import ListaProducts from './components/ListaProducts.vue';  


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

.titulos {
    padding: 20px 0px 15px;
    background-color: cadetblue;
    color: #ffff;
}

table {
    border: solid 1px #000 !important;
}
</style>
