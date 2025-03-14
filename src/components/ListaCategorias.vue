<template>
  <div>
    <h2>Lista de Categorías</h2>
    <ul>
      <li v-for="categoria in categorias" :key="categoria.id">
        {{ categoria.name }}
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  name: "ListaCategorias",
  data() {
    return {
      categorias: []
    };
  },
  mounted() {
  const apiUrl = process.env.VUE_APP_API_URL;
  
  fetch(`${apiUrl}/api/categories`)  // Usamos la variable de entorno
    .then(response => response.json())
    .then(data => {
      this.categorias = data;
    })
    .catch(error => console.error("Error al obtener datos:", error));
}
};
</script>
