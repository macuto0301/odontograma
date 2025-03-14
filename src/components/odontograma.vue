<template>
  <div class="container">
    <!-- Modal de tratamiento -->
    <TratamientoModal :visible="!!piezaSeleccionada" :pieza="piezaSeleccionada || {}" :seccion="seccionSeleccionada"
      :estadosDentales="estadosDentales" :tratamientoExistente="obtenerTratamientoExistente()"
      @guardar-tratamiento="guardarTratamiento" @guardar-estado="guardarEstado" @cerrar="cerrarPanel"
      @seleccionar-seccion="seleccionarSeccion" />

    <div class="odontograma-container">
      <h2>Odontograma (Sistema FDI)</h2>
      <div class="odontograma-grid">
        <!-- Renderizar cada pieza -->
        <Diente v-for="pieza in piezas" :key="pieza.id" :pieza="pieza" :width="svgWidth" :height="svgHeight"
          :piezaSeleccionada="piezaSeleccionada" :seccionSeleccionada="seccionSeleccionada"
          :estadosDentales="estadosDentales" @seleccionar-pieza="seleccionarPieza"
          @seleccionar-seccion="seleccionarSeccion" />

      </div>

      <div class="leyenda">
        <h3>Leyenda</h3>
        <div class="leyenda-item" v-for="item in estadosDentales" :key="item.value">
          <div class="color-box" :style="{ backgroundColor: item.color }"></div>
          <span>{{ item.label }}</span>
        </div>
      </div>

      <!-- Resumen de tratamientos -->
      <div v-if="mostrarResumen" class="resumen-tratamientos">
        <h3>Resumen de Tratamientos</h3>
        <table class="tabla-resumen">
          <thead>
            <tr>
              <th>Pieza</th>
              <th>Sección</th>
              <th>Estado</th>
              <th>Tratamiento</th>
              <th>Fecha</th>
              <th>Profesional</th>
              <th>Acciones</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(tratamiento, index) in tratamientosFiltrados" :key="index">
              <td>{{ tratamiento.pieza.numero_pieza }}</td>
              <td>{{ tratamiento.seccion !== null ? tratamiento.seccion : 'Completa' }}</td>
              <td>
                <span class="estado-dental"
                  :style="{ backgroundColor: getColorPorEstado(tratamiento.tratamiento.estado) }"></span>
                {{ getEstadoLabel(tratamiento.tratamiento.estado) }}
              </td>
              <td>{{ getTipoTratamientoLabel(tratamiento.tratamiento.tipoTratamiento) }}</td>
              <td>{{ formatearFecha(tratamiento.tratamiento.fecha) }}</td>
              <td>{{ tratamiento.tratamiento.profesional }}</td>
              <td>
                <button class="btn-accion" @click="editarTratamiento(tratamiento)">Editar</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
import Diente from './Diente.vue';
import TratamientoModal from './TratamientoModal.vue';

export default {
  name: 'Odontograma',
  components: {
    Diente,
    TratamientoModal
  },
  data() {
    return {
      svgWidth: 100,
      svgHeight: 100,
      piezaSeleccionada: null,
      seccionSeleccionada: null,
      mostrarResumen: true,
      tratamientos: [], // Almacena todos los tratamientos registrados
      estadosDentales: [
        { label: "Sana", value: "sana", color: "#4CAF50" },
        { label: "Caries", value: "caries", color: "#FF5252" },
        { label: "Restaurada", value: "restaurada", color: "#2196F3" },
        { label: "Extraída", value: "extraida", color: "#9E9E9E" },
        { label: "Sin Analizar", value: "sin_analizar", color: "white" }
      ],
      tiposTratamiento: [
        { label: "Seleccione un tratamiento", value: "" },
        { label: "Limpieza", value: "limpieza" },
        { label: "Empaste", value: "empaste" },
        { label: "Endodoncia", value: "endodoncia" },
        { label: "Extracción", value: "extraccion" },
        { label: "Corona", value: "corona" },
        { label: "Puente", value: "puente" },
        { label: "Implante", value: "implante" },
        { label: "Ortodoncia", value: "ortodoncia" },
        { label: "Otro", value: "otro" }
      ],
      piezas: this.inicializarPiezas()
    };
  },
  computed: {
    tratamientosFiltrados() {
      // Aquí podrías agregar filtros para el resumen de tratamientos
      return this.tratamientos;
    }
  },
  methods: {
    inicializarPiezas() {
      const piezas = [];
      const cuadrantes = [
        // Superior derecho (18-11)
        { start: 18, end: 11, direction: -1 },
        // Superior izquierdo (21-28)
        { start: 21, end: 28, direction: 1 },
        // Inferior izquierdo (31-38)
        { start: 31, end: 38, direction: 1 },
        // Inferior derecho (41-48)
        { start: 41, end: 48, direction: 1 }
      ];

      let id = 1;

      cuadrantes.forEach(cuadrante => {
        const { start, end, direction } = cuadrante;
        let current = start;

        while (direction > 0 ? current <= end : current >= end) {
          piezas.push({
            id,
            numero_pieza: current,
            secciones: Array(5).fill("sin_analizar")
          });

          id++;
          current += direction;
        }
      });

      return piezas;
    },

    seleccionarPieza(pieza) {
      this.piezaSeleccionada = pieza;
      this.seccionSeleccionada = null;
    },

    seleccionarSeccion(pieza, index) {
      this.piezaSeleccionada = pieza;
      this.seccionSeleccionada = index;
    },

    guardarEstado(pieza, index, estado) {
      // Clonar pieza para mantener reactividad
      const piezaIndex = this.piezas.findIndex(p => p.id === pieza.id);
      if (piezaIndex !== -1) {
        const nuevasPiezas = [...this.piezas];
        const nuevaPieza = { ...nuevasPiezas[piezaIndex] };
        nuevaPieza.secciones = [...nuevaPieza.secciones];

        nuevaPieza.secciones[index !== null ? index : 0] = estado;

        if (estado === "extraida") {
          // Marcar todas las secciones como "Extraída"
          nuevaPieza.secciones = Array(5).fill("extraida");
        } else if (nuevaPieza.secciones.includes("extraida") && index !== null) {
          // Si se cambia de "Extraída" a otro estado, reiniciar las otras secciones
          nuevaPieza.secciones = nuevaPieza.secciones.map((_, i) => (i === index ? estado : "sin_analizar"));
        }

        nuevasPiezas[piezaIndex] = nuevaPieza;
        this.piezas = nuevasPiezas;
        this.piezaSeleccionada = nuevaPieza;
      }
    },

    guardarTratamiento(data) {
      // Primero actualizamos el estado visual del diente
      this.guardarEstado(data.pieza, data.seccion, data.tratamiento.estado);

      // Buscar si ya existe un tratamiento para esta pieza y sección
      const index = this.tratamientos.findIndex(t =>
        t.pieza.id === data.pieza.id && t.seccion === data.seccion
      );

      if (index !== -1) {
        // Actualizar tratamiento existente
        this.tratamientos[index] = data;
      } else {
        // Agregar nuevo tratamiento
        this.tratamientos.push(data);
      }

      // Cerrar el modal
      this.cerrarPanel();
    },

    obtenerTratamientoExistente() {
      if (!this.piezaSeleccionada) return {};

      // Buscar tratamiento existente
      const tratamiento = this.tratamientos.find(t =>
        t.pieza.id === this.piezaSeleccionada.id && t.seccion === this.seccionSeleccionada
      );

      return tratamiento ? tratamiento.tratamiento : {};
    },

    editarTratamiento(tratamiento) {
      // Seleccionar la pieza y sección correspondiente
      this.piezaSeleccionada = this.piezas.find(p => p.id === tratamiento.pieza.id);
      this.seccionSeleccionada = tratamiento.seccion;
    },

    cerrarPanel() {
      this.piezaSeleccionada = null;
      this.seccionSeleccionada = null;
    },

    getColorPorEstado(estado) {
      const estadoDental = this.estadosDentales.find(item => item.value === estado);
      return estadoDental ? estadoDental.color : "white";
    },

    getEstadoLabel(valor) {
      const estado = this.estadosDentales.find(item => item.value === valor);
      return estado ? estado.label : "Sin Analizar";
    },

    getTipoTratamientoLabel(valor) {
      const tipo = this.tiposTratamiento.find(item => item.value === valor);
      return tipo ? tipo.label : "";
    },

    formatearFecha(fecha) {
      if (!fecha) return "";

      const options = { year: 'numeric', month: 'short', day: 'numeric' };
      return new Date(fecha).toLocaleDateString(undefined, options);
    }
  }
};
</script>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
}

.odontograma-container {
  flex: 2;
  text-align: center;
  font-family: Arial, sans-serif;
  padding: 10px;
}

.odontograma-grid {
  display: grid;
  grid-template-columns: repeat(8, 1fr);
  gap: 10px;
  margin-bottom: 30px;
}

/* Estilos de la leyenda */
.leyenda {
  margin: 20px 0;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 20px;
  font-size: 14px;
}

.leyenda-item {
  display: flex;
  align-items: center;
  gap: 8px;
}

.color-box {
  width: 20px;
  height: 20px;
  border: 1px solid black;
}

/* Estilos para la tabla de resumen */
.resumen-tratamientos {
  margin-top: 30px;
  padding: 0 15px;
}

.tabla-resumen {
  width: 100%;
  border-collapse: collapse;
  margin-top: 15px;
  font-size: 14px;
}

.tabla-resumen th,
.tabla-resumen td {
  padding: 8px 10px;
  text-align: left;
  border-bottom: 1px solid #ddd;
}

.tabla-resumen th {
  background-color: #f5f5f5;
  font-weight: 600;
}

.tabla-resumen tr:hover {
  background-color: #f9f9f9;
}

.estado-dental {
  display: inline-block;
  width: 12px;
  height: 12px;
  border-radius: 50%;
  margin-right: 6px;
  border: 1px solid #ccc;
}

.btn-accion {
  padding: 4px 8px;
  background-color: #2196F3;
  color: white;
  border: none;
  border-radius: 3px;
  cursor: pointer;
  font-size: 12px;
}

.btn-accion:hover {
  background-color: #0b7dda;
}

/* Consultas de medios */
@media (max-width: 1024px) {
  .odontograma-grid {
    grid-template-columns: repeat(4, 1fr);
  }

  .tabla-resumen {
    font-size: 13px;
  }
}

@media (max-width: 768px) {
  .container {
    flex-direction: column;
  }

  .odontograma-grid {
    grid-template-columns: repeat(2, 1fr);
  }

  .leyenda {
    font-size: 12px;
  }

  .color-box {
    width: 16px;
    height: 16px;
  }

  .tabla-resumen th,
  .tabla-resumen td {
    padding: 6px 8px;
  }

  .tabla-resumen {
    display: block;
    overflow-x: auto;
    white-space: nowrap;
  }
}

@media (max-width: 480px) {
  .tabla-resumen {
    font-size: 12px;
  }

  .btn-accion {
    padding: 3px 6px;
    font-size: 11px;
  }
}
</style>