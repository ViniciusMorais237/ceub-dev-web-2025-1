<template>
  <div>
    <MensagemComponent
      :tipo="'alerta'"
      :texto="mensagemAlerta"
      :visivel="mostrarAlerta"
    />
    <MensagemComponent
      :tipo="'sucesso'"
      :texto="mensagemSucesso"
      :visivel="mostrarSucesso"
    />
    <form id="pedido-form" @submit="criarPedido($event)">
      <div>
        <p id="nome-hotDog-content">
          {{ hotDog && hotDog.nome ? hotDog.nome : "-" }}
        </p>
        <img
          id="foto-content"
          :src="hotDog && hotDog.foto ? hotDog.foto : ''"
        />
      </div>
      <div class="inputs" id="form-pedido">
        <label for="nome_cliente">Nome</label>
        <input
          type="text"
          name="nome-cliente"
          id="nome-cliente"
          v-model="nomeCliente"
          placeholder="Digite seu nome"
        />
      </div>
      <div class="inputs">
        <label for="ponto-carne">tipo da Salsicha</label>
        <select name="ponto-carne" id="ponto-carne" v-model="tipoSalsicha">
          <option value="" selected>Selecione o tipo da salsicha</option>
          <option
            v-for="tipoSalsicha in tiposSalsicha"
            :key="tipoSalsicha.id"
            :value="tipoSalsicha"
          >
            {{ tipoSalsicha.descricao }}
          </option>
        </select>
      </div>
      <div id="opcionais-titulo" class="inputs">
        <label id="opcionais-titulo" for="Opcionais"
          >Selecione os opcionais</label
        >
        <label id="opcionais-subtitulo" for="Complemento"
          >Adicione um complemento</label
        >

        <div
          class="checkbox-container"
          v-for="complemento in listaComplementos"
          :key="complemento.id"
        >
          <input
            type="checkbox"
            :name="complemento.nome"
            :value="complemento"
            v-model="listaComplementosSelecionados"
          />
          <span>{{ complemento.nome }}</span>
        </div>

        <label for="Complemento">Adcione uma bebida</label>

        <div
          class="checkbox-container"
          v-for="bebida in listaBebidas"
          :key="bebida.id"
        >
          <input
            type="checkbox"
            :name="bebida.nome"
            :value="bebida"
            v-model="listaBebidasSelecionadas"
          />
          <span>{{ bebida.nome }}</span>
        </div>
      </div>
      <div class="inputs">
        <input type="submit" class="submit-btn" value="Confirmar Pedido" />
      </div>
    </form>
  </div>
</template>

<script>
import MensagemComponent from "./MensagemComponent.vue";
export default {
  name: "PedidoComponent",
  components: { MensagemComponent },
  data() {
    return {
      nomeCliente: "",
      tipoSalsicha: "",
      tiposSalsicha: [],
      listaComplementosSelecionados: [],
      listaBebidasSelecionadas: [],
      listaComplementos: [],
      listaBebidas: [],
      mostrarAlerta: false,
      mensagemAlerta: "",
      mostrarSucesso: false,
      mensagemSucesso: "",
    };
  },
  props: {
    hotDog: null,
  },
  methods: {
    async getTipoPontos() {
      const response = await fetch("http://localhost:3000/tipos_salsicha");
      const data = await response.json();
      this.tiposSalsicha = data;
    },
    async getOpcionais() {
      const response = await fetch("http://localhost:3000/opcionais");
      const data = await response.json();
      this.listaComplementos = data.complemento;
      this.listaBebidas = data.bebidas;
    },
    async criarPedido(e) {
      e.preventDefault();

      this.mostrarAlerta = false;
      this.mostrarSucesso = false;

      if (!this.nomeCliente || !this.tipoSalsicha) {
        this.mensagemAlerta = "Preencha o nome e selecione o tipo da salsicha!";
        this.mostrarAlerta = true;

        setTimeout(() =>
      {
        this.mostrarAlerta =  false;
      }, 2000)

        return;
      }

      const dadosPedido = {
        nome: this.nomeCliente,
        salsicha: this.tipoSalsicha,
        bebidas: Array.from(this.listaBebidasSelecionadas),
        complemento: Array.from(this.listaComplementosSelecionados),
        statusId: 5,
        hotDog: this.hotDog,
      };
      const dadosJson = JSON.stringify(dadosPedido);
      const req = await fetch("http://localhost:3000/pedidos", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: dadosJson,
      });

      if (req.ok) {
        this.mensagemSucesso = "Pedido de Hot-Dog criado com sucesso!";
        this.mostrarSucesso = true;

        setTimeout(() => {
          this.mostrarSucesso = false;
        }, 2000);

        this.nomeCliente = "";
        this.tipoSalsicha = "";
        this.listaComplementosSelecionados = [];
        this.listaBebidasSelecionadas = [];

        setTimeout(() =>
      {
        this.$router.push("/pedidos");
      }, 2000)

                  
      }
    },
  },
  mounted() {
    this.getTipoPontos();
    this.getOpcionais();
  },
};
</script>

<style scoped>
#foto-content {
  margin-bottom: 16px;
  border-radius: 16px;
  position: relative;
  z-index: -1;
  justify-content: center;
  width: 100%;
  height: 180px;
  object-fit: cover;
}

#nome-hamburguer-content {
  font-size: 43px;
  font-weight: bold;
  text-align: start;
  margin-bottom: -90px;
  margin-left: 40px;
  color: antiquewhite;
  padding: 16px;
}

#pedido-form {
  max-width: 750px;
  margin: 0 auto;
}

.inputs {
  display: flex;
  flex-direction: column;
  margin-bottom: 16px;
}

label {
  font-weight: bold;
  margin-bottom: 16px;
  color: #222;
  padding: 5px 12px;
  flex-direction: start;
  display: flex;
  border-left: 4px solid darkgoldenrod;
}

input,
select {
  padding: 12px;
  width: 300px;
  border: solid #222 1px;
  border-radius: 8px;
  height: 20px;
  font-size: 12px;
}

select {
  height: 50px;
}

#opcionais-titulo {
  width: 100%;
}

#opcionais-subtitulo {
  display: flex;
  align-items: flex-start;
  align-content: center;
  width: 100%;
  margin-bottom: 12px;
}

.checkbox-container span {
  margin-left: 6px;
  font-weight: bold;
}

.checkbox-container span,
.checkbox-container input {
  width: auto;
  height: 20px;
}

.submit-btn {
  background-color: #222;
  color: darkgoldenrod;
  font-weight: bold;
  border: solid 1px darkgoldenrod;
  border-radius: 8px;
  cursor: pointer;
  padding: 12px;
  margin: 0 auto;
  font-size: 16px;
  width: 100%;
  height: auto;
  transition: 0.5s;
}
.submit-btn:hover {
  background-color: darkgoldenrod;
  color: #222;
}
</style>
