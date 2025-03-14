<template>
    <svg :width="width" :height="height" viewBox="0 0 100 100" preserveAspectRatio="xMidYMid meet"
        :class="{ 'selected': isSelected }" @click="$emit('seleccionar-pieza', pieza)">
        <!-- Grupo principal con rotación -->
        <g :transform="rotate(45, 50, 50)">
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
                @click.stop="$emit('seleccionar-seccion', pieza, index)"
                :class="{ 'selected-section': isSeccionSelected(index) }" />
        </g>
        <!-- Número FDI debajo del diente (sin rotación) -->
        <text x="50" y="90" text-anchor="middle" font-size="12" fill="black">
            {{ pieza.numero_pieza }}
        </text>
    </svg>
</template>

<script>
export default {
    name: 'Diente',
    props: {
        pieza: {
            type: Object,
            required: true
        },
        width: {
            type: Number,
            default: 100
        },
        height: {
            type: Number,
            default: 100
        },
        piezaSeleccionada: {
            type: Object,
            default: null
        },
        seccionSeleccionada: {
            type: Number,
            default: null
        },
        estadosDentales: {
            type: Array,
            required: true
        }
    },
    computed: {
        isSelected() {
            return this.piezaSeleccionada && this.piezaSeleccionada.id === this.pieza.id;
        }
    },
    methods: {
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

        isSeccionSelected(index) {
            return this.isSelected && this.seccionSeleccionada === index;
        },

        rotate(degrees, x, y) {
            return `rotate(${degrees}, ${x}, ${y})`;
        }
    }
}
</script>

<style scoped>
svg {
    border: 2px solid #e0e0e0;
    margin: 2px;
    max-width: 100%;
    height: auto;
    transition: transform 0.3s ease;
}

svg.selected {
    transform: scale(1.2);
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

@media (max-width: 768px) {
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
}
</style>