# SS-Guincho-const express = require("express");
const axios = require("axios");

const app = express();
app.use(express.json());

let clientes = {};

app.post("/webhook", async (req, res) => {
  const { telefone, mensagem } = req.body;

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

  // 🔕 MODO HUMANO
  if (mensagem.toLowerCase().includes("atendente")) {
    c.modo = "humano";
    return res.send({ resposta: "" });
  }

  if (c.modo === "humano") {
    return res.send({ resposta: "" });
  }

  // 🧠 PROMPT INTELIGENTE
  const prompt = `
Você é atendente da SS Guincho Plataforma.

Dados do cliente:
Origem: ${c.origem}
Destino: ${c.destino}
Placa: ${c.placa}
Situação: ${c.situacao}

Mensagem:
${mensagem}

REGRAS:
- resposta curta
- linguagem humana
- não repetir perguntas
- perguntar apenas o que falta
`;

  try {
    const response = await axios.post(
      "https://openrouter.ai/api/v1/chat/completions",
      {
        model: "mistralai/mistral-7b-instruct",
        messages: [{ role: "user", content: prompt }]
      },
      {
        headers: {
          Authorization: "Bearer SUA_API_KEY_AQUI",
          "Content-Type": "application/json"
        }
      }
    );

    const resposta = response.data.choices[0].message.content;

    res.send({ resposta });

  } catch (erro) {
    res.send({ resposta: "Pode me enviar a localização do veículo?" });
  }
});

app.listen(3000);
