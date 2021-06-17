<template lang='html'>
    <div class="container">
        <b-alert variant="success" show>Tu ID es: {{ id }}</b-alert>
        <div class="pantalla">
            <p class="font-weight-bold">.</p>
        </div>

        <div class="asientos">
            <div class="row">
                <b-container class="bv-example-row">
                    <b-row>
                        <b-col cols="3" v-for="asiento in asientos" :key="asiento.id">
                            <p class="asiento col itera" v-text="asiento.id" v-bind:id="asiento.id"
                                v-bind:class="{disponible: asiento.disponible, ocupado: !asiento.disponible, pointer: asientoDisponible(asiento)}"
                                @click="seleccionarAsiento">
                            </p>
                        </b-col>
                    </b-row>
                </b-container>
            </div>
        </div>

        <div class="botones">
            <b-button :disabled="contador == 0" variant="success" @click="guardar">Guardar</b-button>
            <b-button :disabled="contador == 0" style="margin-left: 10px;" variant="danger" @click="cancelar">Cancelar
            </b-button>
            <b-button style="margin-left: 10px;" variant="dark" @click="liberar">Liberar</b-button>
        </div>

        <div style="margin-top: 20px;">
            <strong>Asientos Seleccionados: {{contador}}</strong>
        </div>
    </div>
</template>

<script>
    import firebase from 'firebase';

    const path = 'salas'
    const pathId = '1'

    export default {
        created() {
            //this.actualizarElementos();

            this.id = firebase.database().ref('/users').push().key;
            firebase.database()
                .ref(path)
                .child(pathId)
                .on('value', snapshot => this.cargarElementos(snapshot.val()));
        },

        data() {
            return {
                id: '',
                contador: 0,
                asientos: [],
            }
        },

        methods: {
            seleccionarAsiento: function (event) {
                let asiento = this.asientos.find(a => a.id == event.target.id);
                if (asiento.adquirido || (asiento.user_id != null && asiento.user_id != this.id)) {
                    this.$notify({
                        group: 'foo', type: 'error', title: 'Error', text: 'No es posible seleccionar el asiento ' + asiento.id + ' porque ya esta ocupado.'
                    })
                    this.dismissCountDown = this.dismissSecs
                    return
                }
                asiento.disponible = !asiento.disponible;
                asiento.user_id = this.id;
                this.actualizarElementos();
                this.contador = this.asientosSeleccionados().length
            },

            actualizarElementos: function () {
                firebase.database()
                    .ref(path)
                    .child(pathId)
                    .set(this.asientos);
            },

            validarRespuesta: function (error, committed, snapshot) {
                if (error) {
                    this.$notify({
                        group: 'foo', type: 'error', title: 'Error', text: 'No es posible completar la operación.'
                    })
                }
                if (committed) {
                    this.$notify({
                        group: 'foo', type: 'success', title: 'Exito', text: 'Asientos adquiridos exitosamente.'
                    })
                }
            },

            cargarElementos: function (data) {
                this.asientos = data;
            },

            guardar: function () {
                firebase.database().ref(path).child(pathId).transaction(
                    valoresDB => this.validarCompra(valoresDB),
                    (error, committed, snapshot) => this.validarRespuesta(error, committed, snapshot)
                )
                this.contador = 0;

            },

            validarCompra: function (valoresDB) {
                this.asientosSeleccionados().forEach(function (asiento) {
                    if (valoresDB.find(a => a.id == asiento.id).adquirido) {
                        return
                    }
                    asiento.adquirido = true;
                })
                return this.asientos;
            },

            cancelar: function () {
                this.asientosSeleccionados().forEach(function (asiento) {
                    asiento.user_id = null
                    asiento.disponible = true
                })
                this.actualizarElementos();
                this.contador = 0;
            },

            validarAsientos: function () {
                this.asientosSeleccionados().forEach(function (asiento) {
                    asiento.adquirido = true;
                });
            },

            asientoDisponible: function (asiento) {
                return !asiento.adquirido;
            },

            asientosSeleccionados: function () {
                return this.asientos.filter(a => !a.disponible && !a.adquirido);
            },

            liberar: function () {
                this.asientos.forEach(function (asiento) {
                    asiento.adquirido = false
                    asiento.disponible = true
                    asiento.user_id = null
                })

                this.actualizarElementos();
                this.contador = 0;
            },
        }
    }
</script>

<style lang="css" scoped>
    #app{
        background-color: #1D1717;
    }
    .pantalla p {
        color: #000;
        font-size: 20px;
    }

    .pantalla {
        /* background-color: #2b6282;*/
        background-image: linear-gradient(45deg, #000000 25%, transparent 25%), linear-gradient(-45deg, #000000 25%, transparent 25%), linear-gradient(45deg, transparent 75%, #000000 75%), linear-gradient(-45deg, transparent 75%, #000000 75%);
        background-size: 20px 20px;
        background-position: 0 0, 0 10px, 10px -10px, -10px 0px;
    }

    .asientos {
        margin-top: 60px;
    }

    .asiento {
        color: #ffffff;
        margin: 2px auto;
        width: 50%;
        font-weight: bold;
        padding: 10px 0;
    }

    .col-6:nth-child(2n) {
        padding-top: 15px;
    }

    .pointer {
        cursor: pointer;
    }

    .disponible {
        background-color: #2d4d86;
    }

    .ocupado {
        background-color: #73265f;
    }

    .botones {
        margin-top: 60px;
    }

    @media (max-width: 600px) {
        .asiento {
        font-size: 10px;
    }
}
</style>