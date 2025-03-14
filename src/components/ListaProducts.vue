<template>
    <div>
      <h2>Lista de Productos</h2>
      <ul>
        <li v-for="producto in productos" :key="producto.id">
          <strong>{{ producto.name }}</strong> ({{ producto.category.name }}) - {{ producto.stock }} en stock
        </li>
      </ul>
    </div>
  </template>
  
  <script>
  export default {
    name: "ListaProducts", 
    data() {
      return {
        productos: [] 
      };
    },
    mounted() {
    
      const apiUrl = process.env.VUE_APP_API_URL;
  
      fetch(`${apiUrl}/api/products/categories`)
        .then(response => response.json())
        .then(data => {
          this.productos = data;
        })
        .catch(error => console.error("Error al obtener datos de productos:", error));
    }
  };
  </script>
  