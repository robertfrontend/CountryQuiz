<script>
import ModalResultado from "./components/ModalResultado.vue";
import axios from "axios";
export default {
  name: "App",
  components: {
    ModalResultado,
  },
  data() {
    return {
      countrys: [],
      // codigo pais ramdom
      county_code_ramdom: null,
      country_selecionado: null,
      three_countrys_ramdom: [],

      objetoPregunta: {
        tipo: null,
        pregunta: null,
        respuesta: null,
        otras_respuestas: null,
      },
      respuesta_selecionada: null,
      selecionado_correcto: null,
      respuestas_correctas: 0,
      mostrar_resultados: false,

      isLoading: false,
      error_system: false,
    };
  },

  created() {
    this.getCountrys();
  },

  methods: {
    // traer todos los paies
    async getCountrys() {
      this.isLoading = true;
      const urlAll = "https://restcountries.eu/rest/v2/all";

      try {
        const respuesta = await axios.get(urlAll);
        this.countrys = respuesta.data;
        this.getCodeRamdon();
      } catch (error) {
        console.error(error, "error al traer todos los paises");
      }
    },

    // buscar y selecionar un pais ramdon
    getCodeRamdon() {
      try {
        let countrys = [];
        this.countrys.map((res) => {
          countrys.push(res.alpha2Code);
        });
        const item = countrys[Math.floor(Math.random() * countrys.length)];
        this.county_code_ramdom = item;
        this.error_system = false;

        this.getCountrySelect();
      } catch (error) {
        console.error(error, "Error al mapear y selecionar el codigo pais ramdon");
        this.error_system = true;
      }
    },

    // buscar pais selecionado ramdom y almacenar pais ramdon y su respuesta correcta
    async getCountrySelect() {
      this.isLoading = true;
      const url = `https://restcountries.eu/rest/v2/alpha/${this.county_code_ramdom}`;

      try {
        const respuesta = await axios.get(url);
        this.country_selecionado = respuesta.data;
        this.buscarRespuestasIncorrectas();
      } catch (error) {
        console.error(error, "error al buscar el pais por codigo");
      }
    },

    // buscar paies 3 paises ramdon (respuestas incorrectas)
    // buscar respuestas incorrectas
    async buscarRespuestasIncorrectas() {
      this.isLoading = true;
      let paises = [];

      // conseguir el codigo de 3 paies
      for (var i = 0; i < 3; i++) {
        let countrys = [];
        this.countrys.map((res) => {
          countrys.push(res.alpha2Code);
        });

        // buscar los codigos ramdom
        const item = countrys[Math.floor(Math.random() * countrys.length)];
        paises.push(item);
      }

      if (paises.length > 0) {
        // buscar los paises ramdoms
        try {
          const url = `https://restcountries.eu/rest/v2/alpha?codes=${paises[0]};${paises[1]};${paises[2]}`;
          const respuesta = await axios.get(url);
          this.three_countrys_ramdom = respuesta.data;
          this.error_system = false;

          this.tipopreguntaRamdom();
        } catch (error) {
          this.error_system = true;
          console.log(error, "error al buscar los paises ");
        }
      }
    },

    // selecionar tipo de pregunta ramdom
    tipopreguntaRamdom() {
      // tipos de preguntas:
      // capitales, bandera, idioma, alphacode

      const tipos = ["capital", "bandera"];
      const item = tipos[Math.floor(Math.random() * tipos.length)];

      this.construirPregunta(item);
    },

    // construir pregunta
    construirPregunta(tipo) {
      this.isLoading = true;
      let respuestas = [];

      const tipo_pregunta = tipo;

      switch (tipo_pregunta) {
        // tipo capital
        case "capital":
          this.three_countrys_ramdom.map((res) => {
            if (res.capital !== "") {
              respuestas.push(res.capital);
            }
          });
          respuestas.push(this.country_selecionado.capital);

          this.objetoPregunta = {
            tipo: tipo_pregunta,
            pregunta: `Select capital of the ${this.country_selecionado.name}`,
            respuesta: this.country_selecionado.capital,
            todas_respustas: respuestas.sort(() => Math.random() - 0.5),
          };

          break;

        // tipo bandera
        case "bandera":
          this.three_countrys_ramdom.map((res) => {
            if (res.name !== "") {
              respuestas.push(res.name);
            }
          });
          respuestas.push(this.country_selecionado.name);

          this.objetoPregunta = {
            tipo: tipo_pregunta,
            pregunta: `Which country does this flag belong to?`,
            respuesta: this.country_selecionado.name,
            todas_respustas: respuestas.sort(() => Math.random() - 0.5),
            bandera: this.country_selecionado.flag,
          };

          break;
        default:
          console.log("default capital");
          break;
      }
      this.isLoading = false;
    },

    // selecionar respuesta
    selecionarRespuesta(dato) {
      if (this.selecionado_correcto === false || this.selecionado_correcto === true)
        return;
      this.respuesta_selecionada = dato;

      // si el usuario seleciono correctamente la respuesta
      if (this.respuesta_selecionada == this.objetoPregunta.respuesta) {
        this.selecionado_correcto = true;
        // incrementar el puntaje
        this.respuestas_correctas++;
      }
      // de lo contrario
      else {
        this.selecionado_correcto = false;
      }
    },

    // validar respuesta
    siguientePaso() {
      // si la respuesta fue correcta, seguir con las demas preguntas
      if (this.selecionado_correcto) {
        this.resetear();
      }
      // si la respuesta fue incorrect, mostrar resultados
      else {
        this.mostrar_resultados = true;
      }
    },

    // resetear
    resetear() {
      this.countrys = [];
      this.county_code_ramdom = null;
      (this.three_countrys_ramdom = []),
        (this.objetoPregunta = {
          tipo: null,
          pregunta: null,
          respuesta: null,
          otras_respuestas: null,
        });

      this.respuesta_selecionada = null;
      this.selecionado_correcto = null;
      this.mostrar_resultados = false;

      this.getCountrys();
    },

    // se ejecuta cuando el usuario halla selecionado algo mal y le de a try again
    tryAain() {
      this.resetear();
      this.respuestas_correctas = 0;
    },

    // selecionar pregunta ramdom
  },
};
</script>
<template>
  <div id="app">
    <div class="container">
      <!--container modall--->
      <div class="container_modal">
        <h1>Country Quiz</h1>
        <!---modal preguntas-->
        <div class="modal" :class="isLoading ? 'flex' : ''" v-if="!mostrar_resultados">
          <template v-if="!isLoading">
            <div class="modal__image">
              <img src="./assets/init.svg" alt="" />
            </div>
            <!--tipo bandera-->
            <div v-if="objetoPregunta.tipo == 'bandera'">
              <img :src="objetoPregunta.bandera" width="100px" alt="" />
            </div>
            <!--fin tipo bandera-->
            <div class="modal__title">
              <h3>{{ objetoPregunta.pregunta }}</h3>
            </div>
            <div class="modal__respuestas">
              <template v-if="true">
                <template v-for="(res, index) in objetoPregunta.todas_respustas">
                  <div
                    class="__respuesta"
                    :key="index"
                    @click="selecionarRespuesta(res)"
                    :class="
                      selecionado_correcto
                        ? respuesta_selecionada == res
                          ? 'correct'
                          : ''
                        : respuesta_selecionada == res
                        ? 'incorrect'
                        : ''
                    "
                  >
                    <span>{{ index + 1 }}</span>
                    {{ res }}
                    <template
                      v-if="
                        respuesta_selecionada
                          ? selecionado_correcto
                            ? res == objetoPregunta.respuesta
                            : ''
                          : false
                      "
                    >
                      <i class="far fa-check-circle"></i>
                    </template>
                    <template
                      v-if="
                        respuesta_selecionada
                          ? !selecionado_correcto
                            ? respuesta_selecionada == res
                            : ''
                          : false
                      "
                    >
                      <i class="far fa-times-circle"></i>
                    </template>
                    <!---->
                    <template
                      v-if="
                        respuesta_selecionada != null
                          ? selecionado_correcto == false
                            ? res == objetoPregunta.respuesta
                            : ''
                          : ''
                      "
                    >
                      todo mal
                      <i class="far fa-check-circle" style="color: #60bf88"></i>
                    </template>
                  </div>
                </template>
              </template>
            </div>
            <br />
            <div class="hit">
              <h6>
                <span class="bien">{{ respuestas_correctas }}</span> HIT !!
              </h6>
            </div>
            <div style="text-align: right" v-if="respuesta_selecionada">
              <button class="boton" @click="siguientePaso">Next</button>
            </div>
            <br />
            <div style="text-align: center">
              <span class="link" @click="resetear">Cambiar pregunta</span>
            </div>
            <br />
          </template>
          <div style="text-align: center" v-if="isLoading">
            <div class="lds-dual-ring"></div>
          </div>
        </div>
        <!--fin modal preguntas-->

        <ModalResultado
          v-if="mostrar_resultados"
          :correctas="respuestas_correctas"
          @tryAain="tryAain"
        />
      </div>
      <!--fin container modal-->
    </div>
  </div>
</template>
<style>
body {
  margin: 0;
}
#app {
  width: 100%;
  min-height: 100vh;
  font-family: sans-serif !important;
  color: #2c3e50;
  background-image: url("./assets/background.png");
  background-size: cover;
  background-repeat: no-repeat;
}
.container {
  width: 100%;
  min-height: inherit;
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;
}
.container_modal h1 {
  text-align: left;
  color: white;
  margin: 0;
  padding-bottom: 10px;
}
.container .modal {
  width: 350px;
  min-height: 300px;
  background: white;
  border-radius: 20px;
  padding: 1em 2em;
  text-align: left;
}

.flex {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
.modal__image {
  text-align: right;
}
.modal__image img {
  width: 150px;
  margin-top: -5em;
  margin-right: -2em;
}
.__respuesta {
  width: 300px;
  background: white;
  border: 1px solid rgba(96, 102, 208, 0.7);
  padding: 1em;
  margin: 0;
  border-radius: 12px;
  color: rgba(96, 102, 208, 0.7);
  margin: 1em 0;
  font-weight: 500;
  font-size: 16px;
  cursor: pointer;
  position: relative;
}

@media screen and (max-width: 550px) {
  .modal__image img {
    width: 30%;
    margin-top: -5em;
    margin-right: 0em;
  }
  .container .modal {
    width: 100%;
    margin: 5em auto;
    padding: 4px;
  }
  .__respuesta {
    width: 90%;
    padding: 10px 5px;
  }
}

.__respuesta i {
  font-size: 20px;
  position: absolute;
  top: 30%;
  right: 0.5em;
}

.__respuesta:hover {
  background: #f9a826 !important;
  border: 1px solid transparent;
  color: white;
  transition: all 0.2s;
}

/*.__respuesta.select {
  background: #f9a826 !important;
  border: 1px solid transparent;
  color: white;
  transition: all 0.2s;
} */
.__respuesta.correct {
  background: #60bf88;
  border: 2px solid transparent;
  color: white;
}
.__respuesta.incorrect {
  background: #ea8282;
  border: 2px solid transparent;
  color: white;
}

.__respuesta span {
  padding-left: 0.5em;
  padding-right: 0.8em;
}

.boton {
  padding: 1em;
  height: 3em;
  border: none;
  background: #f9a826;
  border-radius: 12px;
  font-weight: bold;
  color: white;
  font-family: sans-serif;
  cursor: pointer;
}
.link {
  color: #6c5ce7;
  cursor: pointer;
}

.lds-dual-ring {
  display: inline-block;
  width: 80px;
  height: 80px;
}
.lds-dual-ring:after {
  content: " ";
  display: block;
  width: 64px;
  height: 64px;
  margin: 8px;
  border-radius: 50%;
  border: 6px solid #6c5ce7;
  border-color: #6c5ce7 transparent #6c5ce7 transparent;
  animation: lds-dual-ring 1.2s linear infinite;
}
@keyframes lds-dual-ring {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.hit {
  text-align: center;
  font-style: normal;
  font-weight: bold;
  font-size: 40px;
  color: #1d355d;
}
.hit h6 {
  margin: 0;
}
.bien {
  color: #6fcf97 !important;
}
</style>
