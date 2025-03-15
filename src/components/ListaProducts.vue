<template>
    <div class="container mt-4">
      <h2 class="text-center mb-4">Lista de Productos</h2>
      <b-button variant="primary" class="mb-3" @click="mostrarFormularioProducto">Agregar Producto</b-button>
  
      <b-table striped hover bordered :items="productos" :fields="fields">
        <template #cell(name)="data">
          <b>{{ data.value }}</b>
        </template>
  
        <template #cell(category)="data">
          <span>{{ data.value }}</span>
        </template>
  
        <template #cell(stock)="data">
          <span :class="{'text-danger': data.value < 3, 'text-success': data.value >= 3}">
            {{ data.value }}
          </span>
        </template>
  
        <template #cell(actions)="data">
          <b-button variant="warning" size="sm" @click="editarProducto(data.item)">
            ✏️
          </b-button>
          <b-button variant="danger" size="sm" class="ml-2" @click="eliminarProducto(data.item.id)">
            🗑
          </b-button>
        </template>
      </b-table>
  
      <b-modal v-model="mostrarModalAgregar" title="Agregar Producto" @ok="agregarProducto">
        <b-form>
          <b-form-group label="Nombre">
            <b-form-input v-model="nuevoProducto.name"></b-form-input>
          </b-form-group>
  
          <b-form-group label="Stock">
            <b-form-input type="number" v-model="nuevoProducto.stock"></b-form-input>
          </b-form-group>
  
          <b-form-group label="Categoría">
            <b-form-select v-model="nuevoProducto.category_id" :options="categorias"></b-form-select>
          </b-form-group>
        </b-form>
      </b-modal>
  
      <b-modal v-model="mostrarModal" title="Editar Producto" @ok="guardarCambios">
        <b-form>
          <b-form-group label="Nombre">
            <b-form-input v-model="productoSeleccionado.name"></b-form-input>
          </b-form-group>
  
          <b-form-group label="Stock">
            <b-form-input type="number" v-model="productoSeleccionado.stock"></b-form-input>
          </b-form-group>
        </b-form>
      </b-modal>
    </div>
  </template>
  

  
  <script>
  export default {
    name: "ListaProducts",
    data() {
      return {
        productos: [], // Productos de la API
        productoSeleccionado: {}, // Producto en edición
        nuevoProducto: { name: "", stock: 0, category_id: null }, // Datos del nuevo producto
        mostrarModal: false, // Controla el modal de edición
        mostrarModalAgregar: false, // Controla el modal de agregar producto
        categorias: [], // Categorías disponibles
        fields: [
          { key: 'name', label: 'Nombre del Producto' },
          { key: 'category', label: 'Categoría' },
          { key: 'stock', label: 'Stock' },
          { key: 'actions', label: 'Acciones' } // Nueva columna para editar/eliminar
        ]
      };
    },
    mounted() {
      this.obtenerProductos();
      this.obtenerCategorias();
    },
    methods: {
      async obtenerProductos() {
        try {
          const response = await fetch("http://127.0.0.1:8000/api/products/categories");
          const data = await response.json();
          this.productos = data.map(producto => ({
            id: producto.id,
            name: producto.name,
            category: producto.category.name,
            stock: producto.stock
          }));
        } catch (error) {
          console.error("Error al obtener productos:", error);
        }
      },
  
      async obtenerCategorias() {
        try {
          const response = await fetch("http://127.0.0.1:8000/api/categories");
          const data = await response.json();
          this.categorias = data.map(category => ({
            value: category.id,
            text: category.name
          }));
        } catch (error) {
          console.error("Error al obtener categorías:", error);
        }
      },
  
      editarProducto(producto) {
        this.productoSeleccionado = { ...producto }; // Copia del producto
        this.mostrarModal = true;
      },
  
      async guardarCambios() {
        try {
          const response = await fetch(`http://127.0.0.1:8000/api/products/${this.productoSeleccionado.id}`, {
            method: "PUT",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({
              name: this.productoSeleccionado.name,
              stock: this.productoSeleccionado.stock
            })
          });
  
          if (response.ok) {
            this.obtenerProductos();
          } else {
            console.error("Error al actualizar producto");
          }
        } catch (error) {
          console.error("Error en la solicitud PUT:", error);
        }
      },
  
      async eliminarProducto(id) {
        if (!confirm("¿Estás seguro de eliminar este producto?")) return;
  
        try {
          const response = await fetch(`http://127.0.0.1:8000/api/products/${id}`, {
            method: "DELETE"
          });
  
          if (response.ok) {
            this.productos = this.productos.filter(producto => producto.id !== id);
          } else {
            console.error("Error al eliminar producto");
          }
        } catch (error) {
          console.error("Error en la solicitud DELETE:", error);
        }
      },
  
      mostrarFormularioProducto() {
        this.nuevoProducto = { name: "", stock: 0, category_id: null }; // Resetear los campos
        this.mostrarModalAgregar = true; // Mostrar el modal
      },
  


      async agregarProducto() {
  try {
    console.log("Ejecutando agregarProducto...");  // 🟢 Verifica si se ejecuta más de una vez

    const response = await fetch("http://127.0.0.1:8000/api/products", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(this.nuevoProducto)
    });

    if (response.ok) {
      console.log("Producto agregado correctamente");
      
      // Opcional: Obtener los datos de nuevo para asegurarte de que no haya duplicados
      this.obtenerProductos();

      // Cerrar modal y resetear formulario
      this.mostrarModalAgregar = false;
      this.nuevoProducto = { name: "", stock: 0, category_id: null };
    } else {
      console.error("Error al agregar producto, código:", response.status);
    }
  } catch (error) {
    console.error("Error en la solicitud POST:", error);
  }
}




    }
  };
  </script>
  
<style scoped>
.container {
  max-width: 900px;
}
</style>