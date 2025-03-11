<template>
    <div class="odontograma-container">
      <h2>Odontograma (Sistema FDI)</h2>
      <svg :width="svgWidth" :height="svgHeight" viewBox="0 0 800 600" preserveAspectRatio="xMidYMid meet">
        <!-- Dibujar cada pieza dental -->
        <g v-for="pieza in piezas" :key="pieza.id">
          <!-- Grupo principal para aplicar la rotación -->
          <g :transform="`rotate(45, ${pieza.x}, ${pieza.y})`">
            <!-- Círculo base del diente -->
            <circle :cx="pieza.x" :cy="pieza.y" r="30" fill="white" stroke="black" stroke-width="2" />
  
            <!-- Líneas divisorias -->
            <line :x1="pieza.x - 30" :y1="pieza.y" :x2="pieza.x + 30" :y2="pieza.y" stroke="black" />
            <line :x1="pieza.x" :y1="pieza.y - 30" :x2="pieza.x" :y2="pieza.y + 30" stroke="black" />
  
            <!-- Círculo central en las líneas divisorias -->
            <circle :cx="pieza.x" :cy="pieza.y" r="10" fill="black" />
  
            <!-- Cuadrantes circulares (secciones) -->
            <path 
              v-for="(seccion, index) in pieza.secciones" 
              :key="index"
              :d="getPath(pieza.x, pieza.y, index)" 
              :fill="getColor(seccion)"
              stroke="black" 
              stroke-width="1"
              @click.stop="seleccionarSeccion(pieza, index)"
            />
          </g>
  
          <!-- Número FDI debajo del diente (sin rotación) -->
          <text :x="pieza.x" :y="pieza.y + 50" text-anchor="middle" font-size="12" fill="black">
            {{ pieza.numero_pieza }}
          </text>
        </g>
      </svg>
  
      <!-- Panel de edición -->
      <div v-if="piezaSeleccionada" class="panel-edicion">
        <h3>Editar Pieza {{ piezaSeleccionada.numero_pieza }} - Sección {{ seccionSeleccionada }}</h3>
        <select v-model="piezaSeleccionada.secciones[seccionSeleccionada]" @change="guardarEstado">
          <option value="sana">Sana</option>
          <option value="caries">Caries</option>
          <option value="restaurada">Restaurada</option>
          <option value="extraida">Extraída</option>
          <option value="sin_analizar">Sin Analizar</option>
        </select>
      </div>
  
      <!-- Leyenda de colores -->
      <div class="leyenda">
        <h3>Leyenda</h3>
        <div class="leyenda-item" v-for="(item, index) in leyendaColores" :key="index">
          <div class="color-box" :style="{ backgroundColor: item.color }"></div>
          <span>{{ item.label }}</span>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        svgWidth: 800,
        svgHeight: 600,
        piezas: [
          // Cuadrante 1: Superior Derecho (18 al 11)
          { id: 1, numero_pieza: 18, x: 100, y: 100, secciones: Array(5).fill("sin_analizar") },
          { id: 2, numero_pieza: 17, x: 180, y: 100, secciones: Array(5).fill("sin_analizar") },
          { id: 3, numero_pieza: 16, x: 260, y: 100, secciones: Array(5).fill("sin_analizar") },
          { id: 4, numero_pieza: 15, x: 340, y: 100, secciones: Array(5).fill("sin_analizar") },
          { id: 5, numero_pieza: 14, x: 420, y: 100, secciones: Array(5).fill("sin_analizar") },
          { id: 6, numero_pieza: 13, x: 500, y: 100, secciones: Array(5).fill("sin_analizar") },
          { id: 7, numero_pieza: 12, x: 580, y: 100, secciones: Array(5).fill("sin_analizar") },
          { id: 8, numero_pieza: 11, x: 660, y: 100, secciones: Array(5).fill("sin_analizar") },
  
          // Cuadrante 2: Superior Izquierdo (21 al 28)
          { id: 9, numero_pieza: 21, x: 100, y: 200, secciones: Array(5).fill("sin_analizar") },
          { id: 10, numero_pieza: 22, x: 180, y: 200, secciones: Array(5).fill("sin_analizar") },
          { id: 11, numero_pieza: 23, x: 260, y: 200, secciones: Array(5).fill("sin_analizar") },
          { id: 12, numero_pieza: 24, x: 340, y: 200, secciones: Array(5).fill("sin_analizar") },
          { id: 13, numero_pieza: 25, x: 420, y: 200, secciones: Array(5).fill("sin_analizar") },
          { id: 14, numero_pieza: 26, x: 500, y: 200, secciones: Array(5).fill("sin_analizar") },
          { id: 15, numero_pieza: 27, x: 580, y: 200, secciones: Array(5).fill("sin_analizar") },
          { id: 16, numero_pieza: 28, x: 660, y: 200, secciones: Array(5).fill("sin_analizar") },
  
          // Cuadrante 3: Inferior Izquierdo (31 al 38)
          { id: 17, numero_pieza: 31, x: 100, y: 400, secciones: Array(5).fill("sin_analizar") },
          { id: 18, numero_pieza: 32, x: 180, y: 400, secciones: Array(5).fill("sin_analizar") },
          { id: 19, numero_pieza: 33, x: 260, y: 400, secciones: Array(5).fill("sin_analizar") },
          { id: 20, numero_pieza: 34, x: 340, y: 400, secciones: Array(5).fill("sin_analizar") },
          { id: 21, numero_pieza: 35, x: 420, y: 400, secciones: Array(5).fill("sin_analizar") },
          { id: 22, numero_pieza: 36, x: 500, y: 400, secciones: Array(5).fill("sin_analizar") },
          { id: 23, numero_pieza: 37, x: 580, y: 400, secciones: Array(5).fill("sin_analizar") },
          { id: 24, numero_pieza: 38, x: 660, y: 400, secciones: Array(5).fill("sin_analizar") },
  
          // Cuadrante 4: Inferior Derecho (41 al 48)
          { id: 25, numero_pieza: 41, x: 100, y: 500, secciones: Array(5).fill("sin_analizar") },
          { id: 26, numero_pieza: 42, x: 180, y: 500, secciones: Array(5).fill("sin_analizar") },
          { id: 27, numero_pieza: 43, x: 260, y: 500, secciones: Array(5).fill("sin_analizar") },
          { id: 28, numero_pieza: 44, x: 340, y: 500, secciones: Array(5).fill("sin_analizar") },
          { id: 29, numero_pieza: 45, x: 420, y: 500, secciones: Array(5).fill("sin_analizar") },
          { id: 30, numero_pieza: 46, x: 500, y: 500, secciones: Array(5).fill("sin_analizar") },
          { id: 31, numero_pieza: 47, x: 580, y: 500, secciones: Array(5).fill("sin_analizar") },
          { id: 32, numero_pieza: 48, x: 660, y: 500, secciones: Array(5).fill("sin_analizar") }
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
      getColor(estado) {
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
  
  
  <style>
  .odontograma-container {
    text-align: center;
    font-family: Arial, sans-serif;
  }
  
  svg {
    border: 2px solid #e0e0e0;
    margin: 20px auto;
    max-width: 100%;
    height: auto;
  }
  
  .panel-edicion {
    margin-top: 20px;
  }
  
  select {
    padding: 8px;
    font-size: 16px;
  }
  
  /* Media Queries para dispositivos móviles */
  @media (max-width: 768px) {
    svg {
      width: 100%;
      height: auto;
    }
  
    text {
      font-size: 10px;
    }
  
    circle, line, path {
      stroke-width: 1;
    }
  }
  
  @media (max-width: 480px) {
    text {
      font-size: 8px;
    }
  
    circle, line, path {
      stroke-width: 0.8;
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
  </style>