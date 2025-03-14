<template>
    <div>
        <!-- Capa superpuesta -->
        <div v-if="visible" class="overlay" @click="cancelarCambios"></div>

        <!-- Panel de edición de tratamiento -->
        <transition name="bounce">
            <div v-if="visible" class="panel-tratamiento">

                <!-- Visualización del diente en la parte superior del panel -->
                <div class="diente-preview">
                    <Diente :pieza="pieza" :width="120" :height="120" :piezaSeleccionada="pieza"
                        :seccionSeleccionada="seccion" :estadosDentales="estadosDentales"
                        @seleccionar-seccion="seleccionarSeccionLocal" />
                </div>


                <div class="panel-header">
                    <h3>Tratamiento - Pieza {{ pieza.numero_pieza }} ({{ getNombreDiente(pieza.numero_pieza) }}) {{
                        seccion !== null ? `- Sección ${seccion}` : ''
                        }}</h3>
                    <button class="btn-cerrar" @click="cancelarCambios">&times;</button>
                </div>



                <div class="panel-body">
                    <!-- Estado dental -->
                    <div class="form-group">
                        <label class="form-label">Estado Dental:</label>
                        <div class="radio-grid">
                            <label v-for="estado in estadosDentales" :key="estado.value" class="radio-option">
                                <input type="radio" :value="estado.value" v-model="formData.estado"
                                    name="estado-pieza" />
                                <span class="radio-label">
                                    <span class="color-dot" :style="{ backgroundColor: estado.color }"></span>
                                    {{ estado.label }}
                                </span>
                            </label>
                        </div>
                    </div>

                    <!-- Tipo de tratamiento -->
                    <div class="form-group">
                        <label class="form-label" for="tipo-tratamiento">Tipo de Tratamiento:</label>
                        <select id="tipo-tratamiento" v-model="formData.tipoTratamiento" class="form-control">
                            <option value="">Seleccione un tratamiento</option>
                            <option value="limpieza">Limpieza</option>
                            <option value="empaste">Empaste</option>
                            <option value="endodoncia">Endodoncia</option>
                            <option value="extraccion">Extracción</option>
                            <option value="corona">Corona</option>
                            <option value="puente">Puente</option>
                            <option value="implante">Implante</option>
                            <option value="ortodoncia">Ortodoncia</option>
                            <option value="otro">Otro</option>
                        </select>
                    </div>

                    <!-- Fecha -->
                    <div class="form-group">
                        <label class="form-label" for="fecha-tratamiento">Fecha:</label>
                        <input id="fecha-tratamiento" type="date" v-model="formData.fecha" class="form-control" />
                    </div>

                    <!-- Descripción -->
                    <div class="form-group">
                        <label class="form-label" for="descripcion-tratamiento">Descripción:</label>
                        <textarea id="descripcion-tratamiento" v-model="formData.descripcion"
                            class="form-control text-area" placeholder="Detalles del tratamiento..."></textarea>
                    </div>

                    <!-- Costo estimado -->
                    <div class="form-group">
                        <label class="form-label" for="costo-tratamiento">Costo Estimado:</label>
                        <input id="costo-tratamiento" type="number" v-model="formData.costo" class="form-control"
                            min="0" step="0.01" />
                    </div>

                    <!-- Profesional -->
                    <div class="form-group">
                        <label class="form-label" for="profesional-tratamiento">Profesional:</label>
                        <input id="profesional-tratamiento" type="text" v-model="formData.profesional"
                            class="form-control" />
                    </div>
                </div>

                <div class="panel-footer">
                    <button class="btn btn-cancelar" @click="cancelarCambios">Cancelar</button>
                    <button class="btn btn-guardar" @click="guardarCambios">Guardar Cambios</button>
                </div>
            </div>
        </transition>
    </div>
</template>

<script>
import Diente from './Diente.vue';

export default {
    name: 'TratamientoModal',
    components: {
        Diente
    },
    props: {
        visible: {
            type: Boolean,
            default: false
        },
        pieza: {
            type: Object,
            default: () => ({})
        },
        seccion: {
            type: Number,
            default: null
        },
        estadosDentales: {
            type: Array,
            required: true
        },
        piezaSeleccionada: {
            type: Object,
            default: () => ({}),
        },
        tratamientoExistente: {
            type: Object,
            default: () => ({})
        }
    },
    data() {
        return {
            formData: {
                estado: '',
                tipoTratamiento: '',
                fecha: new Date().toISOString().split('T')[0], // Fecha actual en formato YYYY-MM-DD
                descripcion: '',
                costo: 0,
                profesional: ''
            },
            // Mapeo de números FDI a nombres de dientes
            nombresDientes: {
                // Cuadrante superior derecho (1)
                18: "Tercer molar superior derecho",
                17: "Segundo molar superior derecho",
                16: "Primer molar superior derecho",
                15: "Segundo premolar superior derecho",
                14: "Primer premolar superior derecho",
                13: "Canino superior derecho",
                12: "Incisivo lateral superior derecho",
                11: "Incisivo central superior derecho",

                // Cuadrante superior izquierdo (2)
                21: "Incisivo central superior izquierdo",
                22: "Incisivo lateral superior izquierdo",
                23: "Canino superior izquierdo",
                24: "Primer premolar superior izquierdo",
                25: "Segundo premolar superior izquierdo",
                26: "Primer molar superior izquierdo",
                27: "Segundo molar superior izquierdo",
                28: "Tercer molar superior izquierdo",

                // Cuadrante inferior izquierdo (3)
                31: "Incisivo central inferior izquierdo",
                32: "Incisivo lateral inferior izquierdo",
                33: "Canino inferior izquierdo",
                34: "Primer premolar inferior izquierdo",
                35: "Segundo premolar inferior izquierdo",
                36: "Primer molar inferior izquierdo",
                37: "Segundo molar inferior izquierdo",
                38: "Tercer molar inferior izquierdo",

                // Cuadrante inferior derecho (4)
                41: "Incisivo central inferior derecho",
                42: "Incisivo lateral inferior derecho",
                43: "Canino inferior derecho",
                44: "Primer premolar inferior derecho",
                45: "Segundo premolar inferior derecho",
                46: "Primer molar inferior derecho",
                47: "Segundo molar inferior derecho",
                48: "Tercer molar inferior derecho"
            }
        };
    },
    watch: {
        visible(newVal) {
            if (newVal) {
                this.inicializarFormulario();
            }
        },
        seccion() {
            // Reinitialize the form when the section changes
            this.inicializarFormulario();
        }
    },
    methods: {
        // Método para obtener el nombre del diente según su número FDI
        getNombreDiente(numeroPieza) {
            return this.nombresDientes[numeroPieza] || "Desconocido";
        },
        inicializarFormulario() {
            // Si hay un tratamiento existente, cargamos sus datos
            if (this.tratamientoExistente && Object.keys(this.tratamientoExistente).length > 0) {
                this.formData = { ...this.tratamientoExistente };
            } else {
                // Si no hay tratamiento existente, iniciamos con valores por defecto
                this.formData = {
                    estado: this.pieza.secciones ? this.pieza.secciones[this.seccion || 0] : 'sin_analizar',
                    tipoTratamiento: '',
                    fecha: new Date().toISOString().split('T')[0],
                    descripcion: '',
                    costo: 0,
                    profesional: ''
                };
            }
        },
        seleccionarSeccionLocal(pieza, index) {
            // Actualizamos la sección seleccionada localmente
            // y notificamos al componente padre (Odontograma)
            this.$emit('seleccionar-seccion', pieza, index);
        },
        cambiarSeccion(pieza, seccionIndex) {
            // Update the selected section
            this.$emit('seleccionar-seccion', pieza, seccionIndex);
            // Reset the form for the newly selected section
            this.inicializarFormulario();
        },
        guardarCambios() {
            // Emitimos un evento con todos los datos del formulario
            this.$emit('guardar-tratamiento', {
                pieza: this.pieza,
                seccion: this.seccion,
                tratamiento: { ...this.formData }
            });

            // Si solo queremos actualizar el estado dental
            this.$emit('guardar-estado', this.pieza, this.seccion, this.formData.estado);
        },
        cancelarCambios() {
            this.$emit('cerrar');
        }
    }
}
</script>

<style scoped>
.overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.5);
    z-index: 999;
}

.panel-tratamiento {
    display: flex;
    flex-direction: column;
    position: fixed;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background: white;
    border-radius: 8px;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
    width: 90%;
    max-width: 500px;
    max-height: 90vh;
    z-index: 1000;
    overflow: auto;
}

/* Nuevo estilo para la visualización del diente */
.diente-preview {
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 15px;
    background-color: #f9f9f9;
    border-bottom: 1px solid #eaeaea;
}

/* Add these media queries for responsive sizing */
@media (max-width: 768px) {
    .diente-preview {
        padding: 10px;
    }

    .diente-preview>>>svg {
        width: 100px;
        height: 100px;
    }
}

@media (max-width: 480px) {
    .diente-preview {
        padding: 8px;
    }

    .diente-preview>>>svg {
        width: 120px;
        height: 120px;
    }
}

.panel-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 15px 20px;
    border-bottom: 1px solid #eaeaea;
    background-color: #f8f9fa;
    border-radius: 8px 8px 0 0;
}

.panel-header h3 {
    margin: 0;
    font-size: 18px;
    color: #333;
}

.btn-cerrar {
    background: none;
    border: none;
    font-size: 24px;
    cursor: pointer;
    color: #666;
}

.btn-cerrar:hover {
    color: #333;
}

.panel-body {
    padding: 20px;
    overflow-y: auto;
}

.panel-footer {
    display: flex;
    justify-content: flex-end;
    gap: 10px;
    padding: 15px 20px;
    border-top: 1px solid #eaeaea;
    background-color: #f8f9fa;
    border-radius: 0 0 8px 8px;
}

.form-group {
    margin-bottom: 15px;
}

.form-label {
    display: block;
    margin-bottom: 6px;
    font-weight: 600;
    color: #555;
}

.form-control {
    width: 100%;
    padding: 8px 12px;
    border: 1px solid #ddd;
    border-radius: 4px;
    font-size: 14px;
}

.text-area {
    min-height: 80px;
    resize: vertical;
}

.radio-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(120px, 1fr));
    gap: 10px;
    margin-top: 5px;
}

.radio-option {
    display: flex;
    align-items: center;
    cursor: pointer;
}

.radio-label {
    display: flex;
    align-items: center;
    margin-left: 5px;
}

.color-dot {
    display: inline-block;
    width: 12px;
    height: 12px;
    border-radius: 50%;
    margin-right: 6px;
    border: 1px solid #ccc;
}

.btn {
    padding: 8px 16px;
    border-radius: 4px;
    font-size: 14px;
    cursor: pointer;
    transition: all 0.2s;
}

.btn-cancelar {
    background-color: #f8f9fa;
    border: 1px solid #ddd;
    color: #666;
}

.btn-cancelar:hover {
    background-color: #eaeaea;
}

.btn-guardar {
    background-color: #4CAF50;
    border: 1px solid #4CAF50;
    color: white;
}

.btn-guardar:hover {
    background-color: #3d8b40;
}

/* Estilos de transición */
.bounce-enter-active {
    animation: bounce-in 0.3s;
}

.bounce-leave-active {
    animation: bounce-in 0.3s reverse;
}

@keyframes bounce-in {
    0% {
        opacity: 0;
        transform: scale(0.8) translate(-50%, -50%);
        transform-origin: top left;
    }

    50% {
        opacity: 1;
        transform: scale(1.05) translate(-48%, -48%);
        transform-origin: top left;
    }

    100% {
        transform: scale(1) translate(-50%, -50%);
        transform-origin: top left;
    }
}

@media (max-width: 768px) {
    .panel-tratamiento {
        width: 95%;
    }

    .radio-grid {
        grid-template-columns: repeat(auto-fill, minmax(100px, 1fr));
    }
}

@media (max-width: 480px) {
    .panel-tratamiento {
        width: 100%;
        max-height: 100vh;
        border-radius: 0;
        top: 0;
        left: 0;
        transform: none;
    }

    .radio-grid {
        grid-template-columns: repeat(2, 1fr);
    }

    .panel-body {
        padding: 15px;
    }
}
</style>