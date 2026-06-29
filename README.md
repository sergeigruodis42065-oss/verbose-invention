dolphin-bot/
├── server.js
├── package.json
├── .env
└── public/
    └── index.html
.env:

BOT_TOKEN=твой_токен_от_BotFather
OPENAI_API_KEY=sk-...
WEBHOOK_URL=https://твой-проект.up.railway.app
WEBAPP_URL=https://твой-проект.up.railway.app
PORT=3000
package.json:

{
  "name": "dolphin-bot",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "dotenv": "^16.0.0",
    "express": "^4.18.0",
    "node-telegram-bot-api": "^0.64.0",
    "openai": "^4.0.0"
  }
server.js:

require('dotenv').config();
const express = require('express');
const TelegramBot = require('node-telegram-bot-api');
const OpenAI = require('openai');
const path = require('path');

const app = express();
app.use(express.json());
app.use(express.static('public'));

const bot = new TelegramBot(process.env.BOT_TOKEN);
const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

// Webhook
app.post('/webhook', async (req, res) => {
  const msg = req.body.message;
  if (!msg || !msg.text) return res.sendStatus(200);

  const chatId = msg.chat.id;
  const userText = msg.text;

  try {
    const completion = await openai.chat.completions.create({
      model: 'gpt-4o',
      messages: [
        {
          role: 'system',
          content: `Ты ИИ-ассистент пляжа «Дельфин». 
Помогаешь клиентам с информацией об аттракционах, бронированиях, акциях и событиях.
Отвечай дружелюбно и кратко. Всегда предлагай забронировать через мини-приложение.`
        },
        { role: 'user', content: userText }
      ]
    });

    const reply = completion.choices[0].message.content;
    await bot.sendMessage(chatId, reply, {
      reply_markup: {
        inline_keyboard: [[{
          text: '🐬 Открыть приложение',
          web_app: { url: process.env.WEBAPP_URL }
        }]]
      }
    });
  } catch (err) {
    console.error(err);
    await bot.sendMessage(chatId, 'Произошла ошибка, попробуй ещё раз.');
  }

  res.sendStatus(200);
});

// Установка webhook
app.get('/set-webhook', async (req, res) => {
  try {
    await bot.setWebHook(`${process.env.WEBHOOK_URL}/webhook`);
    res.send('Webhook установлен');
  } catch (err) {
    res.send('Ошибка: ' + err.message);
  }
});

app.listen(process.env.PORT || 3000, () => {
  console.log('Сервер запущен');
});



