<template>
  <div class="container">
    <!-- Panel de edición -->
    <div v-if="piezaSeleccionada" class="panel-edicion">
      <h3>Editar Pieza {{ piezaSeleccionada.numero_pieza }} - Sección {{ seccionSeleccionada }}</h3>

      <div class="radio-group">
        <label>
          <input type="radio" value="sana" v-model="piezaSeleccionada.secciones[seccionSeleccionada]"
            @change="guardarEstado" name="estado-pieza" />
          Sana
        </label>

        <label>
          <input type="radio" value="caries" v-model="piezaSeleccionada.secciones[seccionSeleccionada]"
            @change="guardarEstado" name="estado-pieza" />
          Caries
        </label>

        <label>
          <input type="radio" value="restaurada" v-model="piezaSeleccionada.secciones[seccionSeleccionada]"
            @change="guardarEstado" name="estado-pieza" />
          Restaurada
        </label>

        <label>
          <input type="radio" value="extraida" v-model="piezaSeleccionada.secciones[seccionSeleccionada]"
            @change="guardarEstado" name="estado-pieza" />
          Extraída
        </label>

        <label>
          <input type="radio" value="sin_analizar" v-model="piezaSeleccionada.secciones[seccionSeleccionada]"
            @change="guardarEstado" name="estado-pieza" />
          Sin Analizar
        </label>
      </div>
    </div>

    <div class="odontograma-container">
      <h2>Odontograma (Sistema FDI)</h2>
      <div class="odontograma-grid">
        <!-- Dibujar cada pieza dental -->
        <svg v-for="pieza in piezas" :key="pieza.id" :width="svgWidth" :height="svgHeight" viewBox="0 0 100 100"
          preserveAspectRatio="xMidYMid meet"
          :class="{ 'selected': piezaSeleccionada && piezaSeleccionada.id === pieza.id }"
          @click="seleccionarPieza(pieza)">
          <!-- Grupo principal para aplicar la rotación -->
          <g :transform="`rotate(45, 50, 50)`">
            <!-- Círculo base del diente -->
            <circle cx="50" cy="50" r="30" fill="white" stroke="black" stroke-width="2" />
            <!-- Líneas divisorias -->
            <line x1="20" y1="50" x2="80" y2="50" stroke="black" />
            <line x1="50" y1="20" x2="50" y2="80" stroke="black" />
            <!-- Círculo central en las líneas divisorias -->
            <circle cx="50" cy="50" r="10" fill="black" />
            <!-- Cuadrantes circulares (secciones) -->
            <path v-for="(seccion, index) in pieza.secciones" :key="index" :d="getPath(50, 50, index)"
              :fill="getColor(pieza, seccion)" stroke="black" stroke-width="1"
              @click.stop="seleccionarSeccion(pieza, index)"
              :class="{ 'selected-section': piezaSeleccionada && piezaSeleccionada.id === pieza.id && seccionSeleccionada === index }" />
          </g>
          <!-- Número FDI debajo del diente (sin rotación) -->
          <text x="50" y="90" text-anchor="middle" font-size="12" fill="black">
            {{ pieza.numero_pieza }}
          </text>
        </svg>

      </div>
      <div class="leyenda">
        <h3>Leyenda</h3>
        <div class="leyenda-item" v-for="(item, index) in leyendaColores" :key="index">
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
      piezas: [
        // Cuadrante 1: Superior Derecho (18 al 11)
        { id: 1, numero_pieza: 18, secciones: Array(5).fill("sin_analizar") },
        { id: 2, numero_pieza: 17, secciones: Array(5).fill("sin_analizar") },
        { id: 3, numero_pieza: 16, secciones: Array(5).fill("sin_analizar") },
        { id: 4, numero_pieza: 15, secciones: Array(5).fill("sin_analizar") },
        { id: 5, numero_pieza: 14, secciones: Array(5).fill("sin_analizar") },
        { id: 6, numero_pieza: 13, secciones: Array(5).fill("sin_analizar") },
        { id: 7, numero_pieza: 12, secciones: Array(5).fill("sin_analizar") },
        { id: 8, numero_pieza: 11, secciones: Array(5).fill("sin_analizar") },
        // Cuadrante 2: Superior Izquierdo (21 al 28)
        { id: 9, numero_pieza: 21, secciones: Array(5).fill("sin_analizar") },
        { id: 10, numero_pieza: 22, secciones: Array(5).fill("sin_analizar") },
        { id: 11, numero_pieza: 23, secciones: Array(5).fill("sin_analizar") },
        { id: 12, numero_pieza: 24, secciones: Array(5).fill("sin_analizar") },
        { id: 13, numero_pieza: 25, secciones: Array(5).fill("sin_analizar") },
        { id: 14, numero_pieza: 26, secciones: Array(5).fill("sin_analizar") },
        { id: 15, numero_pieza: 27, secciones: Array(5).fill("sin_analizar") },
        { id: 16, numero_pieza: 28, secciones: Array(5).fill("sin_analizar") },
        // Cuadrante 3: Inferior Izquierdo (31 al 38)
        { id: 17, numero_pieza: 31, secciones: Array(5).fill("sin_analizar") },
        { id: 18, numero_pieza: 32, secciones: Array(5).fill("sin_analizar") },
        { id: 19, numero_pieza: 33, secciones: Array(5).fill("sin_analizar") },
        { id: 20, numero_pieza: 34, secciones: Array(5).fill("sin_analizar") },
        { id: 21, numero_pieza: 35, secciones: Array(5).fill("sin_analizar") },
        { id: 22, numero_pieza: 36, secciones: Array(5).fill("sin_analizar") },
        { id: 23, numero_pieza: 37, secciones: Array(5).fill("sin_analizar") },
        { id: 24, numero_pieza: 38, secciones: Array(5).fill("sin_analizar") },
        // Cuadrante 4: Inferior Derecho (41 al 48)
        { id: 25, numero_pieza: 41, secciones: Array(5).fill("sin_analizar") },
        { id: 26, numero_pieza: 42, secciones: Array(5).fill("sin_analizar") },
        { id: 27, numero_pieza: 43, secciones: Array(5).fill("sin_analizar") },
        { id: 28, numero_pieza: 44, secciones: Array(5).fill("sin_analizar") },
        { id: 29, numero_pieza: 45, secciones: Array(5).fill("sin_analizar") },
        { id: 30, numero_pieza: 46, secciones: Array(5).fill("sin_analizar") },
        { id: 31, numero_pieza: 47, secciones: Array(5).fill("sin_analizar") },
        { id: 32, numero_pieza: 48, secciones: Array(5).fill("sin_analizar") }
      ],
      piezaSeleccionada: null,
      seccionSeleccionada: null,
      leyendaColores: [
        { label: "Sana", color: "#4CAF50" },
        { label: "Caries", color: "#FF5252" },
        { label: "Restaurada", color: "#2196F3" },
        { label: "Extraída", color: "#9E9E9E" },
        { label: "Sin Analizar", color: "white" } // White
      ]
    };
  },
  methods: {
    seleccionarPieza(pieza) {
      this.piezaSeleccionada = pieza;
      this.seccionSeleccionada = null;
    },
    getColor(pieza, estado) {
      if (pieza.secciones.includes("extraida")) {
        return "#9E9E9E"; // Gris para "Extraída"
      }
      return {
        sana: "#4CAF50",
        caries: "#FF5252",
        restaurada: "#2196F3",
        extraida: "#9E9E9E",
        sin_analizar: "White" // Transparente para "Sin Analizar"
      }[estado] || "White";
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
        // Si la sección seleccionada es "Extraída", marcar todas las secciones como "Extraída"
        pieza.secciones = pieza.secciones.map(() => "extraida");
      } else {
        // Si se cambia el estado de una sección que está en "Extraída" a otro estado,
        // marcar todas las demás secciones como "Sin Analizar"
        if (pieza.secciones.includes("extraida")) {
          pieza.secciones = pieza.secciones.map((seccion, i) => (i === index ? estado : "sin_analizar"));
        }
      }
      console.log("Estado guardado:", this.piezas);
    },
    getPath(x, y, index) {
      const r = 30; // Radio del círculo externo
      const innerR = 10; // Radio del círculo interno (para el centro)

      // Ángulos para cada cuadrante
      const startAngles = [0, 90, 180, 270];
      const endAngles = [90, 180, 270, 360];

      if (index === 4) {
        // Centro (círculo pequeño)
        return `M ${x - innerR} ${y} A ${innerR} ${innerR} 0 1 0 ${x + innerR} ${y} A ${innerR} ${innerR} 0 1 0 ${x - innerR} ${y} Z`;
      }

      // Calcular puntos de inicio y fin del arco
      const startX = x + r * Math.cos((startAngles[index] * Math.PI) / 180);
      const startY = y + r * Math.sin((startAngles[index] * Math.PI) / 180);
      const endX = x + r * Math.cos((endAngles[index] * Math.PI) / 180);
      const endY = y + r * Math.sin((endAngles[index] * Math.PI) / 180);

      // Crear el path para el cuadrante
      return `
      M ${x} ${y}
      L ${startX} ${startY}
      A ${r} ${r} 0 0 1 ${endX} ${endY}
      Z
      `;
    }
  }
};
</script>

<style scoped>
.container {
  display: flex;
}

.panel-edicion {
  flex: 1;
  border-right: 1px solid #ccc;
  padding: 10px;
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
  /* Añadir transición */
}

svg.selected {
  transform: scale(1.2);
  /* Efecto de agrandamiento */
}

select {
  padding: 8px;
  font-size: 16px;
  transform: scale(1.5);
  /* Efecto de agrandamiento */
}

/* Media Queries para tablets */
@media (max-width: 1024px) {
  .odontograma-grid {
    grid-template-columns: repeat(4, 1fr);
    /* 4 columnas en tablets */
  }
}

/* Media Queries para dispositivos móviles */
@media (max-width: 768px) {
  .odontograma-grid {
    grid-template-columns: repeat(2, 1fr);
  }

  /* Media Queries para tablet */


  svg {
    width: 100%;
    height: auto;
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
}

path:hover {
  transition: transform 0.9s ease;
  /* Añadir transición */
  transform: scale(1.2);
  /* Efecto de agrandamiento */
  transform-origin: center;
  /* Asegurar que la transformación sea desde el centro */
}

path.selected-section {
  transform: scale(1.2);
  /* Efecto de agrandamiento */
  transform-origin: center;
  /* Asegurar que la transformación sea desde el centro */
}





/* Estilos para la leyenda */
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

@media (max-width: 480px) {
  .leyenda {
    font-size: 12px;
  }

  .color-box {
    width: 16px;
    height: 16px;
  }
}


/* Estilos para la leyenda */
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

@media (max-width: 480px) {
  .leyenda {
    font-size: 12px;
  }

  .color-box {
    width: 16px;
    height: 16px;
  }
}

/* Cambia la dirección a columna */
.radio-group {
  display: flex;
  flex-direction: column;
  /* Cambia la dirección a columna */
}
</style>