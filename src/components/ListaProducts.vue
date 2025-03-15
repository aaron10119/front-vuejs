<template>
    <div class="container mt-4">
      <h2 class="text-center mb-4">Lista de Productos</h2>
      <b-button variant="primary" class="mb-3" @click="mostrarFormularioProducto">Agregar Producto</b-button>
  
      <div>
    <!-- Tabla con paginación -->
    <b-table
      striped
      hover
      bordered
      :items="productos"
      :fields="fields"
      :current-page="currentPage"
      :per-page="perPage"
      :total-rows="totalRows"
    >
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

    <!-- Paginador -->
    <b-pagination
      v-model="currentPage"
      :total-rows="totalRows"
      :per-page="perPage"
      align="center"
      class="mt-3"
    ></b-pagination>
  </div>
  
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
  import Swal from 'sweetalert2';
  
  export default {
    name: "ListaProducts",
    data() {
      return {
        productos: [], 
        productoSeleccionado: {}, 
        nuevoProducto: { name: "", stock: 0, category_id: null },
        mostrarModal: false, 
        mostrarModalAgregar: false,
        categorias: [], 
        fields: [
          { key: 'name', label: 'Nombre del Producto' },
          { key: 'category', label: 'Categoría' },
          { key: 'stock', label: 'Stock' },
          { key: 'actions', label: 'Acciones' }
        ],
        currentPage: 1, 
      perPage: 8, 
      totalRows: 0 
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
    
    if (Array.isArray(data)) {
      // Aquí estás estructurando los datos correctamente según la API que mencionas
      this.productos = data.map(producto => ({
        id: producto.id,
        name: producto.name,
        stock: producto.stock,
        category: producto.category.name,  // Accediendo al nombre de la categoría
      }));
      this.totalRows = data.length;  // O ajusta según lo que necesitas
    } else {
      console.error("La estructura de los datos no es la esperada:", data);
    }
  } catch (error) {
    console.error("Error al obtener productos:", error);
  }
}


,
  
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

      
    onPageChange(newPage) {
      this.currentPage = newPage;
      this.obtenerProductos(); 
    },
  
      editarProducto(producto) {
        this.productoSeleccionado = { ...producto }; 
        this.mostrarModal = true;
      },
  
      async guardarCambios() {
        try {
          const response = await fetch(
            `http://127.0.0.1:8000/api/products/${this.productoSeleccionado.id}`, 
            {
              method: "PUT",
              headers: { "Content-Type": "application/json" },
              body: JSON.stringify({
                name: this.productoSeleccionado.name,
                stock: this.productoSeleccionado.stock
              })
            });
  
          if (response.ok) {
            const index = this.productos.findIndex(p => p.id === this.productoSeleccionado.id);
            this.productos[index] = { ...this.productoSeleccionado };
  
            const categoriaIndex = this.categorias.findIndex(c => c.id === this.categoriaSeleccionada.id);
            if (categoriaIndex !== -1) {
              this.categorias[categoriaIndex] = { ...this.categoriaSeleccionada };
            }
  
            this.mostrarModal = false;
            this.obtenerProductos();
            this.obtenerCategorias();
            Swal.fire({
              icon: 'success',
              title: '¡Producto actualizado!',
              text: 'El producto se ha actualizado correctamente.',
            });
          } else {
            console.error("Error al actualizar producto");
          }
        } catch (error) {
          console.error("Error en la solicitud PUT:", error);
        }
      },
  
      async agregarProducto() {
  try {
    console.log("Ejecutando agregarProducto...");

    const response = await fetch("http://127.0.0.1:8000/api/products", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(this.nuevoProducto)
    });

    if (response.ok) {
      const data = await response.json();
      console.log("Producto agregado correctamente:", data);

      Swal.fire({
        icon: 'success',
        title: '¡Producto agregado!',
        text: 'El producto se ha agregado correctamente.',
      });

      this.obtenerProductos();
      this.mostrarModalAgregar = false;
      this.nuevoProducto = { name: "", stock: 0, category_id: null };
    } else {
      const errorData = await response.json();
      console.error("Error al agregar producto, código:", response.status, errorData.message);

      Swal.fire({
        icon: 'error',
        title: '¡Error!',
        text: errorData.message || "Error al agregar producto.",
      });
    }
  } catch (error) {
    console.error("Error en la solicitud POST:", error);

    Swal.fire({
      icon: 'error',
      title: '¡Error!',
      text: 'No admite caracteres especiales',
    });
  }
},
  
      mostrarFormularioProducto() {
        this.nuevoProducto = { name: "", stock: 0, category_id: null }; 
        this.mostrarModalAgregar = true; 
      },
  
      async eliminarProducto(id) {
  const result = await Swal.fire({
    title: '¿Estás seguro?',
    text: "¡No podrás revertir esta acción!",
    icon: 'warning',
    showCancelButton: true,
    confirmButtonColor: '#3085d6',
    cancelButtonColor: '#d33',
    confirmButtonText: 'Sí, eliminarlo!',
    cancelButtonText: 'Cancelar'
  });

  if (result.isConfirmed) {
    try {
      const response = await fetch(`http://127.0.0.1:8000/api/products/${id}`, {
        method: "DELETE"
      });

      if (response.ok) {
        this.productos = this.productos.filter(producto => producto.id !== id);

        Swal.fire({
          icon: 'success',
          title: '¡Producto eliminado!',
          text: 'El producto ha sido eliminado correctamente.',
        });
      } else {
        console.error("Error al eliminar producto");

        Swal.fire({
          icon: 'error',
          title: '¡Error!',
          text: 'Hubo un problema al eliminar el producto. Inténtalo de nuevo.',
        });
      }
    } catch (error) {
      console.error("Error en la solicitud DELETE:", error);

      Swal.fire({
        icon: 'error',
        title: '¡Error!',
        text: 'Hubo un problema con la solicitud. Intenta de nuevo.',
      });
    }
  }
}
,



    }
  };
  </script>
  
<style scoped>
.container {
  max-width: 900px;
}
</style>