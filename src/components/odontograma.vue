<template>
  <div class="container">
    <!-- Capa superpuesta -->
    <div v-if="piezaSeleccionada" class="overlay" @click="cerrarPanel"></div>

    <!-- Panel de edición -->
    <div v-if="piezaSeleccionada" class="panel-edicion">
      <h3>Editar Pieza {{ piezaSeleccionada.numero_pieza }} - Sección {{ seccionSeleccionada }}</h3>

      <div class="radio-group">
        <label v-for="estado in estadosDentales" :key="estado.value">
          <input type="radio" :value="estado.value" v-model="piezaSeleccionada.secciones[seccionSeleccionada]"
            @change="guardarEstado" name="estado-pieza" />
          {{ estado.label }}
        </label>
      </div>
    </div>

    <div class="odontograma-container">
      <h2>Odontograma (Sistema FDI)</h2>
      <div class="odontograma-grid">
        <!-- Renderizar cada pieza -->
        <svg v-for="pieza in piezas" :key="pieza.id" :width="svgWidth" :height="svgHeight" viewBox="0 0 100 100"
          preserveAspectRatio="xMidYMid meet" :class="{ 'selected': isPiezaSelected(pieza) }"
          @click="seleccionarPieza(pieza)">
          <!-- Grupo principal con rotación -->
          <g :transform="`rotate(45, 50, 50)`">
            <!-- Círculo base del diente -->
            <circle cx="50" cy="50" r="30" fill="white" stroke="black" stroke-width="2" />
            <!-- Líneas divisorias -->
            <line x1="20" y1="50" x2="80" y2="50" stroke="black" />
            <line x1="50" y1="20" x2="50" y2="80" stroke="black" />
            <!-- Círculo central en las líneas divisorias -->
            <circle cx="50" cy="50" r="10" fill="black" />
            <!-- Cuadrantes circulares (secciones) -->
            <path v-for="(estado, index) in pieza.secciones" :key="index" :d="getPath(50, 50, index)"
              :fill="getColor(pieza, estado)" stroke="black" stroke-width="1"
              @click.stop="seleccionarSeccion(pieza, index)"
              :class="{ 'selected-section': isSeccionSelected(pieza, index) }" />
          </g>
          <!-- Número FDI debajo del diente (sin rotación) -->
          <text x="50" y="90" text-anchor="middle" font-size="12" fill="black">
            {{ pieza.numero_pieza }}
          </text>
        </svg>
      </div>

      <div class="leyenda">
        <h3>Leyenda</h3>
        <div class="leyenda-item" v-for="item in estadosDentales" :key="item.value">
          <div class="color-box" :style="{ backgroundColor: item.color }"></div>
          <span>{{ item.label }}</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      svgWidth: 100,
      svgHeight: 100,
      piezaSeleccionada: null,
      seccionSeleccionada: null,
      estadosDentales: [
        { label: "Sana", value: "sana", color: "#4CAF50" },
        { label: "Caries", value: "caries", color: "#FF5252" },
        { label: "Restaurada", value: "restaurada", color: "#2196F3" },
        { label: "Extraída", value: "extraida", color: "#9E9E9E" },
        { label: "Sin Analizar", value: "sin_analizar", color: "white" }
      ],
      piezas: this.inicializarPiezas()
    };
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

    guardarEstado() {
      const pieza = this.piezaSeleccionada;
      const index = this.seccionSeleccionada;
      const estado = pieza.secciones[index];

      if (estado === "extraida") {
        // Marcar todas las secciones como "Extraída"
        pieza.secciones = Array(5).fill("extraida");
      } else if (pieza.secciones.includes("extraida")) {
        // Si se cambia de "Extraída" a otro estado, reiniciar las otras secciones
        pieza.secciones = pieza.secciones.map((_, i) => (i === index ? estado : "sin_analizar"));
      }
      console.log(this.piezas);
    },

    getColor(pieza, estado) {
      // Si alguna sección está marcada como extraída, todo el diente será gris
      if (pieza.secciones.includes("extraida")) {
        return "#9E9E9E"; // Gris para "Extraída"
      }

      // Buscar el color en los estadosDentales
      const estadoDental = this.estadosDentales.find(item => item.value === estado);
      return estadoDental ? estadoDental.color : "white";
    },

    getPath(x, y, index) {
      const r = 30; // Radio del círculo exterior
      const innerR = 10; // Radio del círculo interior (para el centro)

      // Manejar la sección central
      if (index === 4) {
        return `M ${x - innerR} ${y} A ${innerR} ${innerR} 0 1 0 ${x + innerR} ${y} A ${innerR} ${innerR} 0 1 0 ${x - innerR} ${y} Z`;
      }

      // Ángulos para cada cuadrante
      const startAngles = [0, 90, 180, 270];
      const endAngles = [90, 180, 270, 360];

      // Calcular puntos de inicio y fin
      const startAngle = startAngles[index] * Math.PI / 180;
      const endAngle = endAngles[index] * Math.PI / 180;
      const startX = x + r * Math.cos(startAngle);
      const startY = y + r * Math.sin(startAngle);
      const endX = x + r * Math.cos(endAngle);
      const endY = y + r * Math.sin(endAngle);

      // Crear el path para el cuadrante
      return `M ${x} ${y} L ${startX} ${startY} A ${r} ${r} 0 0 1 ${endX} ${endY} Z`;
    },

    isPiezaSelected(pieza) {
      return this.piezaSeleccionada && this.piezaSeleccionada.id === pieza.id;
    },

    isSeccionSelected(pieza, index) {
      return this.isPiezaSelected(pieza) && this.seccionSeleccionada === index;
    },
    cerrarPanel() {
      this.piezaSeleccionada = null;
      this.seccionSeleccionada = null;
    }
  }
};
</script>

<style scoped>
.container {
  display: flex;
}

.panel-edicion {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background: white;
  border: 1px solid #ccc;
  padding: 20px;
  z-index: 1000;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  z-index: 999;
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
}

svg {
  border: 2px solid #e0e0e0;
  margin: 20px auto;
  max-width: 100%;
  height: auto;
  transition: transform 0.3s ease;
}

svg.selected {
  transform: scale(1.2);
}

.radio-group {
  display: flex;
  flex-direction: column;
}

path {
  transition: transform 0.3s ease;
  transform-origin: center;
}

path:hover {
  transform: scale(1.2);
}

path.selected-section {
  transform: scale(1.2);
}

/* Estilos de la leyenda */
.leyenda {
  margin-top: 20px;
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

/* Consultas de medios */
@media (max-width: 1024px) {
  .odontograma-grid {
    grid-template-columns: repeat(4, 1fr);
  }
}

@media (max-width: 768px) {
  .container {
    flex-direction: column;
  }

  .panel-edicion {
    border-right: none;
    border-bottom: 1px solid #ccc;
  }

  .odontograma-grid {
    grid-template-columns: repeat(2, 1fr);
  }

  svg {
    width: 100%;
  }

  text {
    font-size: 10px;
  }

  circle,
  line,
  path {
    stroke-width: 1;
  }
}

@media (max-width: 480px) {
  text {
    font-size: 8px;
  }

  circle,
  line,
  path {
    stroke-width: 0.8;
  }

  .leyenda {
    font-size: 12px;
  }

  .color-box {
    width: 16px;
    height: 16px;
  }
}
</style>
