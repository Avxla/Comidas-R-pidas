<template>

<div class="contenedor-pagina">
  <div class="titulo">
    <h1>Brawlers</h1>
    <h3>COMBAT FLAVOR</h3>
  </div>

<div class="barra-admin">
 <button class="btn-admin" @click="alternarPanel">
  {{ mostrarFormulario ? '✖ Cerrar Panel' : '➕ Agregar Producto' }}
</button>
</div>

<div v-if="mostrarFormulario" class="panel-admin">
  <h2>{{ editandoIndex !== null ? '✏️ Editar Producto' : 'Panel Administrativo' }}</h2>

  <div class="form-grid">
    <input
      v-model="nuevoProducto.nombre"
      type="text"
      placeholder="Nombre"
    >

    <input
      v-model="nuevoProducto.precio"
      type="number"
      placeholder="Precio"
    >

    <select v-model="nuevoProducto.categoria" :disabled="editandoIndex !== null">
      <option value="hamburguesas">Hamburguesas</option>
      <option value="perros">Perros</option>
      <option value="salchipapas">Salchipapas</option>
      <option value="choripapas">Choripapas</option>
      <option value="infantil">Infantil</option>
      <option value="adicionales">Adicionales</option>
    </select>

    <input
      v-model="nuevoProducto.descripcion"
      type="text"
      placeholder="Descripción"
    >

    <input
      v-model="nuevoProducto.imagen"
      type="text"
      placeholder="URL Imagen"
    >

    <div class="acciones-form">
      <button class="btn-guardar" @click="guardarCambios">
        {{ editandoIndex !== null ? 'Actualizar Producto' : 'Guardar Producto' }}
      </button>
      <button v-if="editandoIndex !== null" class="btn-cancelar" @click="cancelarEdicion">
        Cancelar
      </button>
    </div>
  </div>
</div>

<div class="barra-carrito">
  <button class="btn-carrito" @click="alternarCarrito">
    🛒 Carrito ({{ carritoItems.length }})
  </button>
</div>

  <div
  class="contenedor-principal"
  :class="{
    'con-carrito': carritoItems.length > 0 && !mostrarCarrito
  }"
>
    <div class="panel-comidas">
      <div v-if="vistaActual === 'menu'" class="menu">
        <div class="ficha" @click="abrirCategoria('hamburguesas')">
          <img :src="hamburguesaImg" alt="logo hamburguesas">
        </div>
        <div class="ficha" @click="abrirCategoria('perros')">
          <img :src="Perros" alt="logo perros">
        </div>
        <div class="ficha" @click="abrirCategoria('salchipapas')">
          <img :src="Salchipapas" alt="logo salchipapas">
        </div>
        <div class="ficha" @click="abrirCategoria('choripapas')">
          <img :src="Choripapas" alt="logo choripapas">
        </div>
        <div class="ficha" @click="abrirCategoria('infantil')">
          <img :src="Infantil" alt="Logo infantil">
        </div>
        <div class="ficha" @click="abrirCategoria('adicionales')">
          <img :src="Adicionales" alt="Logo adicionales">
        </div>
      </div>

      <div v-else-if="vistaActual === 'detalle'" class="vista-detalle">
        <div class="cabecera-detalle">
          <button class="btn-volver" @click="volverAlMenu">Volver</button>
          <h2 class="titulo-categoria">{{ categoriaSeleccionada.toUpperCase() }}</h2>
        </div>

        <div class="lista-comidas">
          <div class="item-comida" v-for="(item, index) in menuData[categoriaSeleccionada]" :key="index">
            <div class="imagen-comida">
              <img :src="item.imagen" :alt="item.nombre">
            </div>
            
            <div class="info-comida">
              <h3>{{ item.nombre }}</h3>
              <p>{{ item.descripcion }}</p>
              
              <div class="controles-admin-item">
                <button class="btn-item-editar" @click="prepararEdicion(item, index)">✏️ Editar</button>
                <button class="btn-item-eliminar" @click="eliminarProducto(index)">🗑️ Eliminar</button>
              </div>
            </div>
            
            <div class="precio-comida">
              <span>{{ formatearPrecio(item.precio) }}</span>
              <button class="btn-agregar" @click="agregarAlCarrito(item)">+</button>
            </div>
          </div>
        </div>
      </div>
    </div>

       
<div v-if="mostrarCarrito" class="modal-carrito">
  <div class="panel-carrito-full">

    <div class="cabecera-carrito">
      <h2 class="titulo-carrito">MI PEDIDO</h2>

      <div>
        <button
          class="btn-cerrar-carrito"
          @click="mostrarCarrito = false"
        >
          ✖
        </button>

        <button
          class="btn-cerrar-carrito"
          @click="vaciarCarrito"
        >
          🗑️
        </button>
      </div>
    </div>

    <div class="lista-carrito">
      <div
        class="item-carrito"
        v-for="(item, idx) in carritoItems"
        :key="idx"
      >
        <div class="info-carrito">
         <h4>
  {{ item.nombre }} x {{ item.cantidad }}
</h4>

<p class="precio-item">
  {{ formatearPrecio(item.precio * item.cantidad) }}
</p>
        </div>

        <button
          class="btn-eliminar"
          @click="eliminarDelCarrito(idx)"
        >
          ✖
        </button>
      </div>
    </div>

    <!-- TOTAL DENTRO DEL PANEL -->
    <div class="total-carrito">
      <h3>
        Total: {{ formatearPrecio(calcularTotal()) }}
      </h3>

      <button
        class="btn-confirmar"
        @click="confirmarPedido"
      >
        ✅ CONFIRMAR
      </button>
    </div>
  

  </div>
</div>
</div>


  <div v-if="mostrarFactura" class="modal-factura" @click.self="cerrarFactura">
    <div class="contenido-factura">
      <div class="factura">
        <div class="factura-header">
          <h2>BRAWLERS</h2>
          <p>COMBAT FLAVOR</p>
          <p class="factura-fecha">{{ obtenerFechaHora() }}</p>
        </div>

        <div class="factura-detalle">
          <table class="tabla-factura">
            <thead>
              <tr>
                <th>Cant.</th>
                <th>Producto</th>
                <th>Precio</th>
                <th>Total</th>
              </tr>
            </thead>
            <tbody>
             <tr v-for="(item, idx) in carritoItems" :key="idx">
  <td class="cantidad">{{ item.cantidad }}</td>

  <td class="producto">
    {{ item.nombre }}
  </td>

  <td class="precio">
    {{ formatearPrecio(item.precio) }}
  </td>

  <td class="total">
    {{ formatearPrecio(item.precio * item.cantidad) }}
  </td>
</tr>
            </tbody>
          </table>
        </div>

        <div class="factura-totales">
          <div class="linea-total">
            <span>SUBTOTAL:</span>
            <span>{{ formatearPrecio(calcularTotal()) }}</span>
          </div>
          <div class="linea-total">
            <span>IVA (0%):</span>
            <span>$0</span>
          </div>
          <div class="linea-total total-final">
            <span>TOTAL:</span>
            <span>{{ formatearPrecio(calcularTotal()) }}</span>
          </div>
        </div>

        <div class="factura-footer">
          <p>¡Gracias por tu compra!</p>
          <p class="factura-legal">Este documento es una factura válida</p>
        </div>
      </div>

      <div class="botones-factura">
        <button class="btn-cerrar-factura" @click="cerrarFactura">✖ CERRAR</button>
        <button class="btn-imprimir-factura" @click="imprimirFactura">🖨️ IMPRIMIR</button>
      </div>
    </div>
  </div>
</div>
</template>



<script setup>
import { ref, reactive } from 'vue'
import Swal from 'https://cdn.jsdelivr.net/npm/sweetalert2@11/+esm'

// Imports de imágenes de categorías
import hamburguesaImg from './assets/Hamburguesas.png';
import Perros from './assets/Perros.png';
import Salchipapas from './assets/Salchipapas.png';
import Choripapas from './assets/Choripapas.png';
import Infantil from './assets/Infantil.png';
import Adicionales from './assets/Adicionales.png';

const vistaActual = ref('menu'); 
const categoriaSeleccionada = ref(''); 
const carritoItems = ref([]);
const mostrarCarrito = ref(false);
const mostrarFactura = ref(false);
const mostrarFormulario = ref(false);
const alternarCarrito = () => {
  mostrarCarrito.value = !mostrarCarrito.value;
};
const agregarAlCarrito = (item) => {

  const productoExistente = carritoItems.value.find(
    p => p.nombre === item.nombre
  );

  if (productoExistente) {
    productoExistente.cantidad++;
  } else {
    carritoItems.value.push({
      ...item,
      cantidad: 1
    });
  }

  Swal.fire({
    position: "top-end",
    icon: "success",
    title: `${item.nombre} agregado al carrito`,
    showConfirmButton: false,
    timer: 1500
  });
};

// Índice auxiliar para saber qué producto editamos (null significa que estamos creando uno nuevo)
const editandoIndex = ref(null);

const nuevoProducto = ref({
  nombre: '',
  descripcion: '',
  precio: '',
  imagen: '',
  categoria: 'hamburguesas'
});

// Convertimos menuData en un objeto reactivo con reactive() para asegurar que Vue detecte los cambios internos de edición y eliminación.
const menuData = reactive({
  hamburguesas: [
    { 
      nombre: "The Pit Burger", 
      descripcion: "Carne de res 150g, queso cheddar derretido, tocineta crispy, salsa Riot Zone.", 
      precio: 25000,
      imagen: "https://images.unsplash.com/photo-1571091718767-18b5b1457add?w=400&h=300&fit=crop"
    },
    { 
      nombre: "Knockout Sencilla", 
      descripcion: "Carne de res 150g, queso mozzarella, lechuga fresca, tomate y salsa de ajo.", 
      precio: 18000,
      imagen: "https://images.pexels.com/photos/1639557/pexels-photo-1639557.jpeg?w=400&h=300&fit=crop"
    },
    { 
      nombre: "Doble Impacto", 
      descripcion: "Doble carne, doble queso cheddar, aros de cebolla y salsa BBQ picante.", 
      precio: 32000,
      imagen: "https://images.unsplash.com/photo-1553979459-d2229ba7433b?w=400&h=300&fit=crop"
    },
    { 
      nombre: "La Callejera", 
      descripcion: "Carne, huevo frito, ripio de papa, queso, tocineta y salsa rosada.", 
      precio: 22000,
      imagen: "https://www.carniceriademadrid.es/wp-content/uploads/2022/09/smash-burger-que-es.jpg"
    },
    { 
      nombre: "Chicken Uppercut", 
      descripcion: "Pechuga apanada extra crujiente, ensalada coleslaw, pepinillos y mayonesa.", 
      precio: 20000,
      imagen: "https://media.istockphoto.com/id/521207406/es/foto/pa%C3%ADs-de-pollo-frito-s%C3%A1ndwich-del-sur.jpg?s=612x612&w=0&k=20&c=pE3vvzFCtdXMa_IsDAa-n7vUE6mFf_sYXnqe9VI7FxQ="
    }
  ],
  perros: [
    { 
      nombre: "Perro Callejero", 
      descripcion: "Salchicha americana, ripio, queso costeño, salsas tradicionales.", 
      precio: 12000,
      imagen: "https://ranchera.com.co/wp-content/uploads/2022/11/perro-colombiano-1.jpg"
    },
    { 
      nombre: "El Suizo Brawler", 
      descripcion: "Salchicha suiza, extra queso fundido, tocineta picada y salsa piña.", 
      precio: 16000,
      imagen: "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRpySyVh9EalDnNLU9u8PxafOPLeRBM1us0ow&s"
    },
    { 
      nombre: "Perro Mexicano", 
      descripcion: "Salchicha, guacamole, jalapeños, pico de gallo y nachos triturados.", 
      precio: 18000,
      imagen: "https://hamburguesaspecadocapital.com/wp-content/uploads/2023/10/Perro-caliente-optimizada-web.jpg"
    },
    { 
      nombre: "El Salvaje", 
      descripcion: "Doble salchicha, carne desmechada, queso mozzarella y maíz tierno.", 
      precio: 20000,
      imagen: "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSPkTOhi2JjJs2tstiwSsudL869p46su5fJAw&s"
    },
    { 
      nombre: "Perro Chori", 
      descripcion: "Chorizo de ternera en pan artesanal, chimichurri y queso fundido.", 
      precio: 17000,
      imagen: "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRqZiWdcRYT_GS-y2A7fZUCCtBNNi_0IWVZsw&s"
    }
  ],
  salchipapas: [
    { 
      nombre: "Salchipapa Sencilla", 
      descripcion: "Papas a la francesa, salchicha tradicional, queso rallado y salsa rosada.", 
      precio: 14000,
      imagen: "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcSwC6GGQhB_1d0wljn9UpnzoL8DzL5obi4JYA&s"
    },
    { 
      nombre: "La Especial", 
      descripcion: "Papas, salchicha premium, pollo desmechado, tocineta y queso fundido.", 
      precio: 22000,
      imagen: "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcTJfr2m_Yhvy-rYQ-6FxvNmcKVuV6q6DbTAZw&s"
    },
    { 
      nombre: "Brawlers Salvaje", 
      descripcion: "Papas, salchicha, carne desmechada, maíz, maduritos y triple queso.", 
      precio: 28000,
      imagen: "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQROcvqsD9ATOFl9HJWMQiHdwAnMv9Inl7XGA&s"
    },
    { 
      nombre: "La Costeña", 
      descripcion: "Papas, salchicha, butifarra, suero costeño y queso costeño rallado.", 
      precio: 20000,
      imagen: "https://cloudfront-us-east-1.images.arcpublishing.com/infobae/EJLJ24LK3JAIRENLK63SULNRE4.jpg"
    },
    { 
      nombre: "Salchipapa BBQ", 
      descripcion: "Papas, salchicha bañada en salsa BBQ, cebolla caramelizada y tocineta.", 
      precio: 18000,
      imagen: "https://cloudfront-us-east-1.images.arcpublishing.com/infobae/CRU7IWBQBVBWTDZT2AZFIXREVI.jpg"
    }
  ],
  choripapas: [
    { 
      nombre: "Choripapa Clásica", 
      descripcion: "Papas crujientes, chorizo antioqueño picado, queso y salsas.", 
      precio: 16000,
      imagen: "https://images.rappi.com/products/1689265490533_09c37fe2-51d3-48d2-a180-b4630acb8d35_salchipapaespecial.jpg"
    },
    { 
      nombre: "Choripapa Picante", 
      descripcion: "Papas, chorizo santandereano, jalapeños y salsa de ají casero.", 
      precio: 18000,
      imagen: "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQB_U_8hu-aAaAeM6sM0Uy8GOQfAbZvW-WGwQ&s"
    },
    { 
      nombre: "La Quesuda", 
      descripcion: "Papas, chorizo, bañada en una piscina de queso cheddar y mozzarella.", 
      precio: 20000,
      imagen: "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRoWQIMxaHrzmxruzQIzx2EJar2MUZhYGtVdA&s"
    },
    { 
      nombre: "Chori-Maíz", 
      descripcion: "Papas, chorizo premium, extra maíz dulce, queso y salsa tártara.", 
      precio: 19000,
      imagen: "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRlifsnsicv4dcSV44PXCToYrCh59mH_u-Vrw&s"
    },
    { 
      nombre: "Mix Brawler", 
      descripcion: "Papas, chorizo, morcilla picada, arepita frita y hogao.", 
      precio: 24000,
      imagen: "https://images.rappi.com/restaurants_background/480d989a-6804-4a8c-b9f5-6705c2eee766-1686176390144.png"
    }
  ],
  infantil: [
    { 
      nombre: "Mini Brawler", 
      descripcion: "Mini hamburguesa con queso y papitas francesas carita.", 
      precio: 15000,
      imagen: "https://tse3.mm.bing.net/th/id/OIP.PviqYWk3K39oAQb03Ax64QHaE7?rs=1&pid=ImgDetMain&o=7&rm=3"
    },
    { 
      nombre: "Nuggets de Combate", 
      descripcion: "6 Nuggets de pollo crujientes acompañados de papas francesas.", 
      precio: 14000,
      imagen: "https://media.a24.com/p/1c69800fa693ed7b3cf2b251644cd8f2/adjuntos/296/imagenes/008/951/0008951716/1200x675/smart/papas-fritas_nuggetsjpg.jpg"
    },
    { 
      nombre: "Salchi-Kids", 
      descripcion: "Porción pequeña de papas con salchicha en rodajas sin salsas fuertes.", 
      precio: 12000,
      imagen: "https://www.seriouseats.com/thmb/mrIC6LF4KWf5dltOBEVBQoMqR04=/1500x0/filters:no_upscale():max_bytes(150000):strip_icc()/20220131-salchipapas-vicky-wasik-15-f01dcb79a9f84bcd909845a1a101a962.jpg"
    },
    { 
      nombre: "Mini Perrito", 
      descripcion: "Perrito caliente sencillo con salsa de tomate y papitas.", 
      precio: 10000,
      imagen: "https://tse2.mm.bing.net/th/id/OIP.4dOSoNv4m3g2IZsNMG2HUAHaE7?rs=1&pid=ImgDetMain&o=7&rm=3"
    },
    { 
      nombre: "Nachos Quesudos", 
      descripcion: "Porción de nachos bañados en queso cheddar suave (sin picante).", 
      precio: 12000,
      imagen: "https://img.taste.com.au/O_5e5BxC/taste/2016/11/tray-baked-nachos-102903-1.jpeg"
    }
  ],
  adicionales: [
    { 
      nombre: "Porción de Papas", 
      descripcion: "Papas a la francesa extra crujientes condimentadas estilo Brawlers.", 
      precio: 6000,
      imagen: "https://tse3.mm.bing.net/th/id/OIP.Jdss4SOTKHXFpShDsBNEngHaE7?rs=1&pid=ImgDetMain&o=7&rm=3"
    },
    { 
      nombre: "Porción de Carne", 
      descripcion: "Porción de carne de res o pollo desmechado para sumar a tus platos.", 
      precio: 8000,
      imagen: "https://tse2.mm.bing.net/th/id/OIP.8_3U1vH5LhvNmRpmemQG2AHaEk?rs=1&pid=ImgDetMain&o=7&rm=3"
    },
    { 
      nombre: "Nachos con Cheddar", 
      descripcion: "Porción personal de nachos crujientes con dip de queso cheddar.", 
      precio: 7000,
      imagen: "https://tse2.mm.bing.net/th/id/OIP.dpzMkrEs6mahq1-Fxd5BLAHaE8?rs=1&pid=ImgDetMain&o=7&rm=3"
    },
    { 
      nombre: "Adición de Tocineta", 
      descripcion: "Trocitos de tocineta ahumada crujiente.", 
      precio: 4000,
      imagen: "https://tse3.mm.bing.net/th/id/OIP.5mOXkXujI_cKGt8eSWAG9AHaEA?rs=1&pid=ImgDetMain&o=7&rm=3"
    },
    { 
      nombre: "Baño de Queso", 
      descripcion: "Extra de queso cheddar fundido para bañar cualquiera de tus platos.", 
      precio: 5000,
      imagen: "https://tse4.mm.bing.net/th/id/OIP.SLj1H4UdGTrtLZccwuB59wHaHQ?rs=1&pid=ImgDetMain&o=7&rm=3"
    }
  ]
});

// Nueva función para el botón principal del panel
const alternarPanel = () => {
  if (mostrarFormulario.value) {
    // Si está abierto, aprovechamos y limpiamos los campos al cerrar
    cancelarEdicion();
  } else {
    // Si está cerrado, simplemente lo abrimos en modo creación
    mostrarFormulario.value = true;
  }
};

// Tu función cancelarEdicion optimizada
const cancelarEdicion = () => {
  editandoIndex.value = null;
  nuevoProducto.value = {
    nombre: '',
    descripcion: '',
    precio: '',
    imagen: '',
    categoria: categoriaSeleccionada.value || 'hamburguesas'
  };
  // Aseguramos el cierre definitivo
  mostrarFormulario.value = false; 
};

// Función centralizada para Guardar o Actualizar
const guardarCambios = () => {
  if (
    !nuevoProducto.value.nombre ||
    !nuevoProducto.value.precio ||
    !nuevoProducto.value.descripcion ||
    !nuevoProducto.value.imagen
  ) {
    Swal.fire({
      icon: "warning",
      title: "Campos incompletos",
      text: "Debes completar todos los campos."
    });
    return;
  }

  if (editandoIndex.value !== null) {
    menuData[nuevoProducto.value.categoria][editandoIndex.value] = {
      nombre: nuevoProducto.value.nombre,
      descripcion: nuevoProducto.value.descripcion,
      precio: Number(nuevoProducto.value.precio),
      imagen: nuevoProducto.value.imagen
    };

    Swal.fire({
      icon: "success",
      title: "Producto actualizado",
      text: "Los cambios fueron guardados correctamente.",
      timer: 2000,
      showConfirmButton: false
    });

  } else {
    menuData[nuevoProducto.value.categoria].push({
      nombre: nuevoProducto.value.nombre,
      descripcion: nuevoProducto.value.descripcion,
      precio: Number(nuevoProducto.value.precio),
      imagen: nuevoProducto.value.imagen
    });

    Swal.fire({
      icon: "success",
      title: "Producto agregado",
      text: "El producto fue agregado al menú.",
      timer: 2000,
      showConfirmButton: false
    });
  }

  cancelarEdicion();
};

// Carga el producto seleccionado en el formulario
const prepararEdicion = (item, index) => {
  editandoIndex.value = index;
  nuevoProducto.value = {
    nombre: item.nombre,
    descripcion: item.descripcion,
    precio: item.precio,
    imagen: item.imagen,
    categoria: categoriaSeleccionada.value // Esto mantiene el select sincronizado
  };
  mostrarFormulario.value = true;
  window.scrollTo({ top: 0, behavior: 'smooth' });
};

// Elimina el producto usando el índice de la categoría activa
const eliminarProducto = async (index) => {
  const productoNombre =
    menuData[categoriaSeleccionada.value][index].nombre;

  const result = await Swal.fire({
    title: "¿Eliminar producto?",
    text: `Se eliminará "${productoNombre}"`,
    icon: "warning",
    showCancelButton: true,
    confirmButtonColor: "#d33",
    cancelButtonColor: "#3085d6",
    confirmButtonText: "Sí, eliminar",
    cancelButtonText: "Cancelar"
  });

  if (result.isConfirmed) {
    menuData[categoriaSeleccionada.value].splice(index, 1);

    if (editandoIndex.value === index) {
      cancelarEdicion();
    }

    Swal.fire({
      icon: "success",
      title: "Producto eliminado",
      text: "El producto fue eliminado correctamente."
    });
  }
};



const formatearPrecio = (precio) => {
  return `$${precio.toLocaleString()}`;
};

const abrirCategoria = (categoria) => {
  categoriaSeleccionada.value = categoria;
  vistaActual.value = 'detalle';
};

const volverAlMenu = () => {
  vistaActual.value = 'menu';
  categoriaSeleccionada.value = '';
};



const eliminarDelCarrito = (index) => {

  const producto = carritoItems.value[index];

  if (producto.cantidad > 1) {
    producto.cantidad--;
  } else {
    carritoItems.value.splice(index, 1);
  }

  Swal.fire({
    icon: "info",
    title: "Producto eliminado",
    text: `${producto.nombre} fue retirado del carrito.`,
    timer: 1500,
    showConfirmButton: false
  });
};

const vaciarCarrito = async () => {

  if (carritoItems.value.length === 0) {
    Swal.fire({
      icon: "info",
      title: "Carrito vacío",
      text: "No hay productos para eliminar."
    });
    return;
  }

  const result = await Swal.fire({
    title: "¿Vaciar carrito?",
    text: "Se eliminarán todos los productos.",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "Sí, vaciar",
    cancelButtonText: "Cancelar"
  });

  if (result.isConfirmed) {
    carritoItems.value = [];

    Swal.fire({
      icon: "success",
      title: "Carrito vacío",
      text: "Todos los productos fueron eliminados."
    });
  }
};

const calcularTotal = () => {
  return carritoItems.value.reduce(
    (sum, item) => sum + (item.precio * item.cantidad),
    0
  );
};

const confirmarPedido = async () => {

  if (carritoItems.value.length === 0) {
    Swal.fire({
      icon: "error",
      title: "Carrito vacío",
      text: "Debes agregar productos antes de confirmar."
    });
    return;
  }

  const result = await Swal.fire({
    title: "Confirmar pedido",
    text: `Total a pagar: ${formatearPrecio(calcularTotal())}`,
    icon: "question",
    showCancelButton: true,
    confirmButtonColor: "#28a745",
    cancelButtonColor: "#d33",
    confirmButtonText: "Confirmar",
    cancelButtonText: "Cancelar"
  });

  if (result.isConfirmed) {
    mostrarCarrito.value = false;
    mostrarFactura.value = true;

    Swal.fire({
      icon: "success",
      title: "Pedido confirmado",
      text: "La factura fue generada correctamente."
    });
  }
};
const obtenerFechaHora = () => {
  const ahora = new Date();
  return ahora.toLocaleString('es-CO', {
    dateStyle: 'full',
    timeStyle: 'medium'
  });
};

const cerrarFactura = () => {
  mostrarFactura.value = false;
  carritoItems.value = [];
  volverAlMenu();
};

const imprimirFactura = () => {
    Swal.fire({
  icon: "success",
  title: "Factura generada",
  text: "Se abrirá la ventana de impresión.",
  timer: 1500,
  showConfirmButton: false
});
  const contenido = document.querySelector('.factura').cloneNode(true);
  const ventana = window.open('', '_blank');

  
  ventana.document.write(`
    <!DOCTYPE html>
    <html>
    <head>
      <title>Factura Brawlers</title>
      <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body { 
          font-family: 'Courier New', 'Segoe UI', monospace;
          background: white;
          padding: 20px;
          display: flex;
          justify-content: center;
        }
        .factura {
          max-width: 400px;
          width: 100%;
          background: white;
          padding: 20px;
          border: 1px solid #ddd;
          box-shadow: 0 0 10px rgba(0,0,0,0.1);
        }
        .factura-header {
          text-align: center;
          border-bottom: 2px solid #333;
          padding-bottom: 15px;
          margin-bottom: 20px;
        }
        .factura-header h2 {
          font-size: 24px;
          letter-spacing: 3px;
          margin-bottom: 5px;
        }
        .factura-fecha {
          font-size: 11px;
          color: #666;
          margin-top: 8px;
        }
        .tabla-factura {
          width: 100%;
          border-collapse: collapse;
          margin-bottom: 20px;
        }
        .tabla-factura th {
          text-align: left;
          border-bottom: 1px solid #ddd;
          padding: 8px 0;
          font-size: 12px;
        }
        .tabla-factura td {
          padding: 8px 0;
          font-size: 12px;
          border-bottom: 1px solid #eee;
        }
        .cantidad {
          width: 50px;
          text-align: center;
        }
        .producto {
          text-align: left;
        }
        .precio, .total {
          text-align: right;
          width: 80px;
        }
        .factura-totales {
          text-align: right;
          border-top: 2px solid #333;
          padding-top: 15px;
          margin-bottom: 20px;
        }
        .linea-total {
          display: flex;
          justify-content: flex-end;
          margin-bottom: 5px;
          font-size: 12px;
        }
        .linea-total span:first-child {
          width: 100px;
          text-align: left;
        }
        .linea-total span:last-child {
          width: 100px;
          text-align: right;
        }
        .total-final {
          font-size: 16px;
          font-weight: bold;
          margin-top: 10px;
          padding-top: 5px;
          border-top: 1px solid #ddd;
        }
        .factura-footer {
          text-align: center;
          border-top: 1px solid #ddd;
          padding-top: 15px;
          font-size: 11px;
          color: #666;
        }
        .factura-legal {
          font-size: 9px;
          margin-top: 5px;
        }
        @media print {
          body {
            padding: 0;
          }
          .factura {
            box-shadow: none;
            border: none;
          }
        }
      </style>
    </head>
    <body>
      ${contenido.outerHTML}
      <script>
        window.onload = () => {
          window.print();
          setTimeout(() => window.close(), 1000);
        };
      <\/script>
    </body>
    </html>
  `);
  
  ventana.document.close();
};
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Creepster&family=Rubik+Distressed&display=swap');

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html, body, #app {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
  overflow: hidden;
  background-color: #000000;
  font-family: 'Creepster', cursive;
}

.contenedor-pagina {
  display: flex;
  flex-direction: column;
  width: 100vw;
  height: 100vh;
  padding: 10px; 
  box-sizing: border-box; 
  overflow: hidden;
}

.titulo {
  display: flex;
  flex-direction: column;
  justify-content: space-evenly;
  align-items: center;
  height: 15vh; 
  margin-bottom: 10px; 
  flex-shrink: 0;
}

.titulo h1 {
  color: #ffffffd5;
  letter-spacing: 20px;
  font-family: 'Creepster', cursive;
  text-transform: uppercase;
  font-size: 80px;
  text-shadow: 5px 5px 5px #ff4500;
  margin: 0;
}

.titulo h3 {
  color: #d3703b;
  font-size: 1.5rem;
  letter-spacing: 5px;
  text-transform: uppercase;
  margin: 0;
}

/* ESTILOS DEL PANEL ADMIN ACTUALIZADO */
.panel-admin {
  background-color: #1a1a1a;
  border: 2px dashed #ff4500;
  border-radius: 8px;
  padding: 15px;
  margin-bottom: 15px;
}
.panel-admin h2 {
  color: #fff;
  font-size: 1.5rem;
  margin-bottom: 10px;
  text-shadow: 0 0 5px #ff4500;
}
.form-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 10px;
  align-items: center;
}
.form-grid input, .form-grid select {
  background-color: #333;
  color: #fff;
  border: 1px solid #ff4500;
  padding: 8px;
  border-radius: 4px;
  font-family: sans-serif;
}
.acciones-form {
  display: flex;
  gap: 10px;
}
.btn-guardar {
  background-color: #ff4500;
  color: white;
  border: none;
  padding: 8px 12px;
  border-radius: 4px;
  cursor: pointer;
  font-weight: bold;
}
.btn-cancelar {
  background-color: #6c757d;
  color: white;
  border: none;
  padding: 8px 12px;
  border-radius: 4px;
  cursor: pointer;
  font-weight: bold;
}
.barra-admin {
  margin-bottom: 10px;
}
.btn-admin {
  background-color: #111;
  color: #ff4500;
  border: 2px solid #ff4500;
  padding: 8px 15px;
  border-radius: 5px;
  cursor: pointer;
  font-weight: bold;
  font-size: 1rem;
}

/* NUEVOS CONTROLES PARA LOS PLATOS INDIVIDUALES */
.controles-admin-item {
  display: flex;
  gap: 10px;
  margin-top: 10px;
}
.btn-item-editar {
  background-color: #ffc107;
  color: #000;
  border: none;
  padding: 5px 10px;
  border-radius: 4px;
  cursor: pointer;
  font-weight: bold;
  font-family: sans-serif;
  font-size: 0.8rem;
}
.btn-item-eliminar {
  background-color: #dc3545;
  color: #fff;
  border: none;
  padding: 5px 10px;
  border-radius: 4px;
  cursor: pointer;
  font-weight: bold;
  font-family: sans-serif;
  font-size: 0.8rem;
}

.contenedor-principal {
  display: flex;
  flex: 1;
  gap: 15px;
  min-height: 0;
  overflow: hidden;
}

.panel-comidas {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow-y: auto;
  overflow-x: hidden;
  min-height: 0;
}

.con-carrito .panel-comidas {
  flex: 8;
}

.panel-carrito {
  flex: 2;
  background: linear-gradient(135deg, #1a1a1a 0%, #0a0a0a 100%);
  border-radius: 12px;
  border: 1px solid #ff4500;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  box-shadow: 0 0 15px rgba(255, 69, 0, 0.3);
  animation: slideIn 0.3s ease;
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateX(20px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

.cabecera-carrito {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px;
  background-color: #ff4500;
  border-bottom: 2px solid #cc3700;
}

.titulo-carrito {
  color: white;
  font-size: 1.2rem;
  letter-spacing: 2px;
  margin: 0;
  text-shadow: 2px 2px 2px rgba(0,0,0,0.5);
}

.btn-cerrar-carrito {
  background-color: rgba(0,0,0,0.5);
  color: white;
  border: none;
  width: 32px;
  height: 32px;
  border-radius: 50%;
  cursor: pointer;
  font-size: 1rem;
  transition: all 0.2s;
}

.btn-cerrar-carrito:active {
  background-color: rgba(0,0,0,0.8);
  transform: scale(0.95);
}

.menu {
  display: grid;
  gap: 10px;
  flex: 1;
}

.ficha {
  position: relative; 
  display: flex;
  justify-content: center; 
  align-items: center;
  border: 2px solid #ff4500;
  border-radius: 12px; 
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
  height: 300px;
  width: 100%;
}

.ficha:active {
  transform: scale(0.95);
  box-shadow: 0 0 15px rgba(255, 69, 0, 0.7);
}

.ficha img {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  object-fit: cover; 
  object-position: center;
  align-items: center;
  z-index: 1; 
}

.vista-detalle {
  flex: 1;
  display: flex;
  flex-direction: column;
  background-color: #000000;
  border-radius: 8px;
  border: 1px solid #333;
  padding: 15px;
  overflow-y: auto;
  overflow-x: hidden;
  min-height: 0;
}

.cabecera-detalle {
  display: flex;
  align-items: center;
  margin-bottom: 20px;
  border-bottom: 2px solid #ff4500;
  padding-bottom: 10px;
  flex-shrink: 0;
}

.cabecera-detalle h2 {
  font-family: 'Creepster', cursive;
}

.btn-volver {
  background-color: #ff4500;
  color: white;
  border: none;
  padding: 8px 15px;
  border-radius: 5px;
  font-weight: bold;
  cursor: pointer;
  margin-right: 15px;
  font-size: 1rem;
  transition: background-color 0.2s;
  font-family: 'Creepster', cursive;
  font-size: 20px;
  letter-spacing: 3px;
}

.btn-volver:active {
  background-color: #cc3700;
}

.titulo-categoria {
  color: #ffffff;
  margin: 0;
  font-size: 1.5rem;
  letter-spacing: 2px;
  text-shadow: 0px 0px 5px rgba(255, 69, 0, 0.8);
}

.lista-comidas {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.imagen-comida {
  flex-shrink: 0;
  width: 100px;
  height: 100px;
  border-radius: 8px;
  overflow: hidden;
  background: linear-gradient(135deg, #1a1a1a 0%, #0a0a0a 100%);
  border: 2px solid #ff4500;
  box-shadow: 0 0 10px rgba(255, 69, 0, 0.3);
}

.imagen-comida img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.3s ease;
}

.item-comida {
  display: flex;
  align-items: center;
  gap: 15px;
  background-color: #262626;
  padding: 15px;
  border-radius: 8px;
  border-left: 4px solid #ff4500;
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.item-comida:hover {
  transform: translateX(5px);
  box-shadow: 0 0 15px rgba(255, 69, 0, 0.2);
}

.item-comida:hover .imagen-comida img {
  transform: scale(1.1);
}

.info-comida {
  display: flex;
  flex-direction: column;
  flex: 1;
}

.info-comida h3 {
  color: #ffb84d;
  margin: 0 0 5px 0;
  font-size: 1.2rem;
}

.info-comida p {
  color: #bfbfbf;
  margin: 0;
  font-size: 0.9rem;
  line-height: 1.3;
}

.precio-comida {
  display: flex;
  align-items: center;
  gap: 10px;
}

.precio-comida span {
  color: #ff4500;
  font-weight: bold;
  font-size: 1.3rem;
  background-color: #000000;
  padding: 5px 12px;
  border-radius: 5px;
  border: 2px solid #ff4500;
  white-space: nowrap;
  font-family: monospace;
}

.btn-agregar {
  background-color: #28a745;
  color: white;
  border: none;
  width: 40px;
  height: 40px;
  border-radius: 50%;
  font-size: 1.5rem;
  font-weight: bold;
  cursor: pointer;
  transition: transform 0.1s;
  flex-shrink: 0;
}

.btn-agregar:active {
  transform: scale(0.9);
}

.lista-carrito {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 10px;
  padding: 15px;
  overflow-y: auto;
  overflow-x: hidden;
}

.item-carrito {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background-color: #262626;
  padding: 10px;
  border-radius: 8px;
  border-left: 3px solid #ff4500;
}

.info-carrito {
  flex: 1;
}

.info-carrito h4 {
  margin: 0 0 5px 0;
  color: #ffb84d;
  font-size: 0.9rem;
}

.precio-item {
  margin: 0;
  color: #fff;
  font-weight: bold;
  font-size: 0.85rem;
}

.btn-eliminar {
  background-color: #dc3545;
  color: white;
  border: none;
  width: 28px;
  height: 28px;
  border-radius: 50%;
  cursor: pointer;
  font-size: 0.8rem;
  flex-shrink: 0;
  transition: all 0.2s;
}

.btn-eliminar:active {
  background-color: #c82333;
  transform: scale(0.95);
}

.total-carrito {
  padding: 15px;
  background-color: #1a1a1a;
  border-top: 2px solid #ff4500;
  text-align: center;
}

.total-carrito h3 {
  color: #ffb84d;
  margin-bottom: 12px;
  font-size: 1.1rem;
}

.btn-confirmar {
  background-color: #28a745;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 5px;
  font-weight: bold;
  cursor: pointer;
  font-size: 0.9rem;
  width: 100%;
  transition: all 0.2s;
}

.btn-confirmar:active {
  background-color: #218838;
  transform: scale(0.98);
}

/* Modal factura */
.modal-factura {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.8);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 10000;
  animation: fadeIn 0.3s ease;
}

@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

.contenido-factura {
  background: white;
  border-radius: 12px;
  padding: 20px;
  max-width: 500px;
  width: 90%;
  max-height: 90vh;
  overflow-y: auto;
  animation: slideUp 0.3s ease;
}

@keyframes slideUp {
  from {
    transform: translateY(50px);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}

.factura {
  background: white;
  color: #000000;
  font-family: 'Courier New', 'Segoe UI', monospace;
  max-width: 400px;
  margin: 0 auto;
}

.factura-header {
  text-align: center;
  border-bottom: 2px solid #333333;
  padding-bottom: 15px;
  margin-bottom: 20px;
}

.factura-header h2 {
  font-size: 24px;
  letter-spacing: 3px;
  margin-bottom: 5px;
  color: #000000;
  font-weight: bold;
}

.factura-header p {
  color: #555555;
  font-size: 12px;
  margin-bottom: 5px;
}

.factura-fecha {
  font-size: 11px;
  color: #666666;
  margin-top: 8px;
}

.tabla-factura {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 20px;
}

.tabla-factura th {
  text-align: left;
  border-bottom: 1px solid #dddddd;
  padding: 8px 0;
  font-size: 12px;
  color: #333333;
}

.tabla-factura td {
  padding: 8px 0;
  font-size: 12px;
  border-bottom: 1px solid #eeeeee;
  color: #000000;
}

.tabla-factura .cantidad {
  width: 50px;
  text-align: center;
}

.tabla-factura .producto {
  text-align: left;
}

.tabla-factura .precio,
.tabla-factura .total {
  text-align: right;
  width: 80px;
}

.factura-totales {
  text-align: right;
  border-top: 2px solid #333333;
  padding-top: 15px;
  margin-bottom: 20px;
}

.linea-total {
  display: flex;
  justify-content: flex-end;
  margin-bottom: 5px;
  font-size: 12px;
  color: #333333;
}

.linea-total span:first-child {
  width: 100px;
  text-align: left;
}

.linea-total span:last-child {
  width: 100px;
  text-align: right;
}

.total-final {
  font-size: 16px;
  font-weight: bold;
  margin-top: 10px;
  padding-top: 5px;
  border-top: 1px solid #dddddd;
  color: #000000;
}

.factura-footer {
  text-align: center;
  border-top: 1px solid #dddddd;
  padding-top: 15px;
  font-size: 11px;
  color: #666666;
}

.factura-legal {
  font-size: 9px;
  margin-top: 5px;
}

.botones-factura {
  display: flex;
  gap: 10px;
  justify-content: center;
  margin-top: 20px;
  padding-top: 15px;
  border-top: 1px solid #eeeeee;
}

.btn-cerrar-factura,
.btn-imprimir-factura {
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 14px;
  font-weight: bold;
  transition: all 0.2s;
}

.btn-cerrar-factura {
  background-color: #6c757d;
  color: white;
}

.btn-cerrar-factura:hover {
  background-color: #5a6268;
}

.btn-imprimir-factura {
  background-color: #28a745;
  color: white;
}

.btn-imprimir-factura:hover {
  background-color: #218838;
}

@media print {
  .modal-factura {
    position: static;
    background: white;
  }
  
  .contenido-factura {
    padding: 0;
    max-width: 100%;
  }
  
  .botones-factura {
    display: none;
  }
  
  .factura {
    box-shadow: none;
    border: none;
  }
}

.panel-comidas::-webkit-scrollbar,
.lista-carrito::-webkit-scrollbar,
.vista-detalle::-webkit-scrollbar {
  width: 6px;
}

.panel-comidas::-webkit-scrollbar-track,
.lista-carrito::-webkit-scrollbar-track,
.vista-detalle::-webkit-scrollbar-track {
  background: #1a1a1a;
  border-radius: 3px;
}

.panel-comidas::-webkit-scrollbar-thumb,
.lista-carrito::-webkit-scrollbar-thumb,
.vista-detalle::-webkit-scrollbar-thumb {
  background: #ff4500;
  border-radius: 3px;
}

@media (max-width: 768px) {
  .titulo h1 {
    font-size: 40px;
    letter-spacing: 10px;
  }
  
  .titulo h3 {
    font-size: 1rem;
    letter-spacing: 3px;
  }
  
  .con-carrito .panel-comidas {
    flex: 6;
  }
  
  .panel-carrito {
    flex: 4;
  }
  
  .info-carrito h4 {
    font-size: 0.75rem;
  }
  
  .precio-item {
    font-size: 0.7rem;
  }
  
  .imagen-comida {
    width: 70px;
    height: 70px;
  }
  
  .item-comida {
    flex-wrap: wrap;
  }
  
  .precio-comida {
    margin-left: auto;
  }
}
 /* =========================================
   Responsividad: Celulares Pequeños (300px a 450px)
   ========================================= */
@media (min-width: 300px) and (max-width: 500px) {
  .titulo {
    height: 10vh;
  }

  .titulo h1 {
    font-size: 32px;
    letter-spacing: 5px;
  }

  .titulo h3 {
    font-size: 0.8rem;
    letter-spacing: 2px;
  }

  /* Cambiamos la disposición a vertical */
  .contenedor-principal {
    flex-direction: column;
  }

  /* Dividimos la pantalla: Menú arriba, Carrito abajo */
  .con-carrito .panel-comidas {
    flex: 1;
  }

  .panel-carrito {
    flex: 1;
    max-height: 45%;
    border-radius: 12px 12px 0 0; /* Borde redondeado arriba */
  }

  .menu {
    
    gap: 8px;
  }

  /* Reducimos dramáticamente la altura para que se vean */
  .ficha {
    height: 160px;
  }

  .cabecera-detalle h2 {
    font-size: 1.2rem;
  }

  .btn-volver {
    font-size: 14px;
    padding: 6px 10px;
    letter-spacing: 1px;
  }

  /* Reducimos los items de comida */
  .imagen-comida {
    width: 65px;
    height: 65px;
  }

  .item-comida {
    padding: 10px;
    gap: 10px;
  }

  .info-comida h3 {
    font-size: 1rem;
  }

  .info-comida p {
    font-size: 0.75rem;
  }

  .precio-comida span {
    font-size: 0.9rem;
    padding: 3px 8px;
  }

  .btn-agregar {
    width: 32px;
    height: 32px;
    font-size: 1.2rem;
  }

  /* Ajustes del carrito */
  .info-carrito h4 {
    font-size: 0.8rem;
  }

  .precio-item {
    font-size: 0.75rem;
  }
}

 @media (min-width: 1000px) and (max-width: 2000px) {  
  .menu {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  grid-template-rows: repeat(3, 1fr);
  gap: 10px;
  flex: 1;
  min-height: 0;
 }
}

@media (min-width: 501px) and (max-width: 999px) {


  .titulo h1 {
    font-size: 45px;
    letter-spacing: 8px;
  }

  .titulo h3 {
    font-size: 1rem;
    letter-spacing: 3px;
  }

  /* También mantenemos la disposición vertical aquí para mejor lectura */
  .contenedor-principal {
    flex-direction: column;
  }

  .con-carrito .panel-comidas {
    flex: 1;
  
  }

  .panel-carrito {
    flex: 1;
 
    border-radius: 12px 12px 0 0;
  }

  .menu {
  display: grid;
  gap: 10px;
  flex: 1;
  min-height: 0;
  }

  .ficha {
  position: relative; 
  display: flex;
  justify-content: center; 
  align-items: center;
  border: 2px solid #ff4500;
  border-radius: 12px; 
  cursor: pointer;
  transition: transform 0.2s, box-shadow 0.2s;
  height: 300px;
  }

  .cabecera-detalle h2 {
    font-size: 1.4rem;
  }

  .btn-volver {
    font-size: 16px;
    padding: 8px 12px;
    letter-spacing: 2px;
  }

  .imagen-comida {
    width: 80px;
    height: 80px;
  }

  .item-comida {
    padding: 12px;
    gap: 12px;
  }

  .info-comida h3 {
    font-size: 1.1rem;
  }

  .info-comida p {
    font-size: 0.85rem;
  }

  .precio-comida span {
    font-size: 1.1rem;
  }
  
  .info-carrito h4 {
    font-size: 0.9rem;
  }

  .precio-item {
    font-size: 0.85rem;
  }
}
/* =========================
   PANEL ADMIN
========================= */

.barra-admin {
  width: 100%;
  margin-bottom: 15px;
}

.btn-admin {
  width: 100%;
  padding: 16px;
  background: #ff4500;
  border: none;
  border-radius: 15px;
  color: white;
  font-size: 1.1rem;
  font-weight: bold;
  cursor: pointer;
  transition: 0.3s;
}

.btn-admin:hover {
  background: #ff4500;
}

.panel-admin {
  width: 100%;
  background: #050505;
  border: 1px solid #ff4500;
  border-radius: 20px;
  padding: 20px;
  margin-bottom: 15px;
}

.panel-admin h2 {
  color: white;
  margin-bottom: 20px;
}

.form-grid {
  display: grid;
  grid-template-columns: repeat(6, 1fr);
  gap: 15px;
  align-items: center;
}

.form-grid input,
.form-grid select {
  background: #111;
  border: 1px solid #333;
  border-radius: 12px;
  padding: 14px;
  color: white;
  outline: none;
}

.form-grid input:focus,
.form-grid select:focus {
  border-color: #ff4500;
}

.btn-guardar {
  background: #ff4500;
  border: none;
  color: white;
  border-radius: 14px;
  padding: 14px;
  cursor: pointer;
  font-weight: bold;
  transition: 0.3s;
}

.btn-guardar:hover {
  background: #ff4500;
}

/* RESPONSIVE */
@media (max-width: 1200px) {
  .form-grid {
    grid-template-columns: 1fr;
  }
}
.barra-carrito {
  margin-bottom: 15px;
}

.btn-carrito {
  width: 100%;
  padding: 15px;
  background: #28a745;
  color: white;
  border: none;
  border-radius: 15px;
  font-size: 1rem;
  font-weight: bold;
  cursor: pointer;
}

.btn-carrito:hover {
  background: #218838;
}
.carrito-fullscreen {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(0, 0, 0, 0.95);
  z-index: 9999;
  display: flex;
  justify-content: center;
  align-items: center;
}

.carrito-fullscreen .panel-carrito {
  width: 95%;
  height: 95%;
  max-width: none;
  max-height: none;
  flex: none;
  border-radius: 20px;
  border: 2px solid #ff4500;
}
/* =========================
   MODAL CARRITO FULLSCREEN
========================= */

.modal-carrito {
  position: fixed;
  inset: 0;
  z-index: 9999;
  background: rgba(0,0,0,0.95);
  backdrop-filter: blur(6px);

  display: flex;
  justify-content: center;
  align-items: center;

  animation: fadeCarrito .25s ease;
}

.panel-carrito-full {
  width: 100%;
  height: 100%;
  border-radius: 0;

  background: linear-gradient(
    135deg,
    #111 0%,
    #050505 100%
  );

  border: 2px solid #ff4500;
  border-radius: 25px;

  display: flex;
  flex-direction: column;

  overflow: hidden;

  box-shadow:
    0 0 40px rgba(255,69,0,.5),
    0 0 80px rgba(255,69,0,.2);
}
.modal-carrito {
  padding: 0;
}

@keyframes fadeCarrito {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}

/* CABECERA */

.panel-carrito-full .cabecera-carrito {
  padding: 20px;
  background: #ff4500;
}

.panel-carrito-full .titulo-carrito {
  font-size: 2rem;
  letter-spacing: 4px;
}

/* LISTA */

.panel-carrito-full .lista-carrito {
  flex: 1;
  overflow-y: auto;
  padding: 20px;
}

.panel-carrito-full .item-carrito {
  padding: 18px;
  margin-bottom: 12px;
  border-radius: 15px;
  background: #1d1d1d;
}

.panel-carrito-full .info-carrito h4 {
  font-size: 1.2rem;
}

.panel-carrito-full .precio-item {
  font-size: 1rem;
}

/* TOTAL */

.panel-carrito-full .total-carrito {
  padding: 25px;
}

.panel-carrito-full .total-carrito h3 {
  font-size: 1.8rem;
}

.panel-carrito-full .btn-confirmar {
  height: 60px;
  font-size: 1.1rem;
}

/* RESPONSIVE TABLET */

@media (max-width: 900px) {

  .panel-carrito-full {
    width: 100%;
    height: 100%;
    border-radius: 0;
  }

  .panel-carrito-full .titulo-carrito {
    font-size: 1.5rem;
  }

  .panel-carrito-full .total-carrito h3 {
    font-size: 1.4rem;
  }
}

/* RESPONSIVE CELULAR */

@media (max-width: 600px) {

  .panel-carrito-full .cabecera-carrito {
    padding: 15px;
  }

  .panel-carrito-full .titulo-carrito {
    font-size: 1.2rem;
    letter-spacing: 2px;
  }

  .panel-carrito-full .item-carrito {
    padding: 12px;
  }

  .panel-carrito-full .info-carrito h4 {
    font-size: .95rem;
  }

  .panel-carrito-full .precio-item {
    font-size: .85rem;
  }

  .panel-carrito-full .btn-confirmar {
    height: 55px;
    font-size: 1rem;
  }
}
.panel-carrito-full {
  display: flex;
  flex-direction: column;
}

.lista-carrito {
  flex: 1;
  overflow-y: auto;
}

.total-carrito {
  flex-shrink: 0;
}
</style>
