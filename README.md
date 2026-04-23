const express = require("express");
const axios = require("axios");

const app = express();
app.use(express.json());

// 🧠 memória simples (gratuita)
let clientes = {};

app.post("/webhook", async (req, res) => {
  const { telefone, mensagem } = req.body;

  if (!telefone || !mensagem) {
    return res.send({ resposta: "Erro na requisição" });
  }

  // cria cliente se não existir
  if (!clientes[telefone]) {
    clientes[telefone] = {
      origem: "",
      destino: "",
      placa: "",
      situacao: "",
      modo: "bot"
    };
  }

  let c = clientes[telefone];
  let msg = mensagem.toLowerCase();

  // 🔕 MODO ATENDENTE
  if (msg.includes("atendente") || msg.includes("assumi")) {
    c.modo = "humano";
    return res.send({ resposta: "" });
  }

  if (msg.includes("liberar bot")) {
    c.modo = "bot";
    return res.send({ resposta: "Atendimento automático retomado." });
  }

  if (c.modo === "humano") {
    return res.send({ resposta: "" });
  }

  // 🚗 DETECTAR PLACA
  if (mensagem.match(/[A-Z]{3}[0-9][A-Z0-9][0-9]{2}/)) {
    c.placa = mensagem.toUpperCase();
  }

  // 📍 DETECTAR ENDEREÇO
  if (
    msg.includes("rua") ||
    msg.includes("av") ||
    msg.includes("br") ||
    msg.includes("km")
  ) {
    if (!c.origem) {
      c.origem = mensagem;
    } else if (!c.destino) {
      c.destino = mensagem;
    }
  }

  // ⚠️ DETECTAR SITUAÇÃO
  if (
    msg.includes("bateu") ||
    msg.includes("pane") ||
    msg.includes("não liga") ||
    msg.includes("capotou")
  ) {
    c.situacao = mensagem;
  }

  // 🚨 EMERGÊNCIA
  if (msg.includes("acidente") || msg.includes("capotamento")) {
    return res.send({ resposta: "Envie sua localização." });
  }

  // 🤖 LÓGICA DE ATENDIMENTO
  let resposta = "";

  if (!c.origem) {
    resposta = "Pode me enviar a localização do veículo?";
  } else if (!c.destino) {
    resposta = "Qual será o destino?";
  } else if (!c.placa) {
    resposta = "Pode me informar a placa?";
  } else if (!c.situacao) {
    resposta = "Qual a situação do veículo?";
  } else {
    resposta = "Perfeito, já estamos verificando.";
  }

  res.send({ resposta });
});

// rota teste
app.get("/", (req, res) => {
  res.send("Servidor rodando!");
});

app.listen(3000, () => console.log("Servidor rodando na porta 3000"));
