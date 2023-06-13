<template>
  <header class="header">
    <h1>愛酷智能 攤位小助手</h1>
  </header>
  <section class="content" ref="msgContainer">
    <ul id="chat-area" v-for="(message, index) in chatMessages" :key="index">
      <li v-if="message.role == 'Bot'">
        <div class="balloon">
          <img class="img-circle" src="openai.png" alt="image" />
          <p class="talk">{{ message.msg }}</p>
        </div>
      </li>
      <li v-else>
        <div class="balloon balloon-r">
          <p class="talk talk-r">{{ message.msg }}</p>
        </div>
      </li>
    </ul>
  </section>
  <footer class="footer">
    <input
      id="msg-send"
      class="msg-input"
      placeholder="請輸入訊息"
      v-model="prompt"
      v-on:keyup.enter="sendMessage()"
    >
    <button class="btn-submit" type="button" @click="sendMessage()">
      傳送
    </button>
  </footer>
</template>

<script>
import axios from 'axios';
import { Toast } from './utils/helpers';

export default {
  name: 'App',
  data() {
    return {
      openaiEndpoint: process.env.VUE_APP_OPENAI_ENDPOINT,
      openaiModelDeploymentName: process.env.VUE_APP_OPENAI_MODEL_DEPLOYMENT_NAME,
      openaiKey: process.env.VUE_APP_OPENAI_KEY,
      prompt: '',
      chatMessages: [],
      botMessageArrayIndex: 0,
    };
  },
  mounted() {
    this.chatMessages.push({
      msg: '你好~我是愛酷智能攤位小助手 爾森，有什麼可以幫你服務的嗎? ヽ(✿ﾟ▽ﾟ)ノ',
      role: 'Bot',
    });
  },
  watch: {
    chatMessages: {
      handler() {
        window.scrollTo({
          left: 0,
          top: document.body.scrollHeight,
          behavior: 'smooth',
        });
      },
      deep: true,
    },
  },
  methods: {
    sendMessage() {
      if (this.prompt !== '') {
        this.showUserMessage();
        this.postOpenAI(this.prompt);
        this.prompt = '';
      } else {
        Toast.fire({
          icon: 'error',
          title: '請輸入訊息 !',
        });
      }
    },
    showUserMessage() {
      this.chatMessages.push({
        msg: this.prompt,
        role: 'User',
      });
    },
    showBotMessage(botMessage) {
      this.chatMessages.push({
        msg: botMessage,
        role: 'Bot',
      });
      this.botMessageArrayIndex = this.chatMessages.length - 1;
    },
    async postOpenAI(userPrompt = '') {
      this.showBotMessage('思考中 ~');
      const reqBody = {
        messages: [
          {
            role: 'system',
            content: '你的名字叫「爾森」，你是「愛酷智能科技股份有限公司」，在「未來商務展」，這個展場中，放置在攤位協助解決來賓關於攤位內容的問題',
          },
          {
            role: 'system',
            content: '所有問答的資料都來自下方資料，以其資料為基礎來進行問答，回答的內容要活潑、適當使用各種顏文字，但回答不要過長',
          },
          {
            role: 'system',
            content: '本次攤位活動中，有填問卷就可以轉「愛酷大輪盤」的活動，輪盤一共有八個獎項，輪盤指針轉道的獎項即可獲得該大獎',
          },
          {
            role: 'system',
            content: `
當使用者詢問「愛酷智能科技股份有限公司」的相關資訊時，即完整回答下方全部內容，勿刪減增加。
『愛酷智能科技成立於 2018 年，致力於「協助企業掌握自有數據，提升顧客終身價值」，透過自有產品打造前瞻性 MarTech 解決方案，與企業攜手，在數位轉型浪潮中，為顧客打造高效個人化精準行銷。公司創業團隊多為亞洲企業高階經理人，因體會到「雲端科技」和「數據整合」對企業的重要性與未來發展性，認為未來將由數據與雲端驅動，因而將此兩大概念融入愛酷智能科技的產品開發與服務。愛酷智能科技目前提供百人服務團隊、服務市場超過數百家中大型企業客戶，更獲得知名投資人、競賽、專利與多項技術認證的肯定，並持續投入研發與市場擴張。透過海外市場的推廣，愛酷智能科技正邁向更穩健的發展，並深度結合大型企業客戶需求推動各產業數據與科技轉型，參與世界性的變革時代。』`,
          },
          {
            role: 'system',
            content: '請記憶使用者的回覆，以順暢進行一系列的問答。',
          },
          {
            role: 'user',
            content: userPrompt,
          },
          {
            role: 'assistant',
            content: '',
          },
        ],
        temperature: 0.7,
        top_p: 0.95,
        frequency_penalty: 0,
        presence_penalty: 0,
        max_tokens: 800,
        stop: null,
      };
      try {
        const res = await axios.post(
          'https://aoia-demo.openai.azure.com/openai/deployments/DemoWeb/chat/completions?api-version=2023-03-15-preview',
          JSON.stringify(reqBody),
          {
            headers: {
              'content-type': 'application/json',
              'api-key': 'fdaf1146cc084cf6b7384a004c474313',
            },
          },
        );
        this.chatMessages[this.botMessageArrayIndex].msg = res.data.choices[0].message.content;
      } catch (error) {
        console.log(error.response.data.error);
      }
    },
  },
};
</script>

<style scoped>

* {
  margin: 0;
  padding: 0;
}
.header {
  top: 0;
  position: sticky;
  z-index: 1;
  width: 100%;
  height: 50px;
  font-size: 10px;
  text-align: center;
  line-height: 50px;
  background-color: rgb(30, 144, 255);
  color: white;
}

.content {
  width: 100%;
  min-height: calc(100vh - 100px);
  margin-bottom: 50px;
  background-color: lightgray;
}

#chat-area {
  padding: 10px;
  list-style: none;
}

.balloon {
  margin: 20px 0;
  display: flex;
  align-items: flex-start;
}
.balloon-r {
  margin-right: 15px;
  justify-content: flex-end;
}

.img-circle {
  width: 50px;
  height: 50px;
  margin: 0 15px;
  border-radius: 25px;
  background-color: white;
}

.talk {
  max-width: 500px;
  padding: 10px;
  border-radius: 10px;
  background: white;
  white-space: pre-wrap;
}
.talk-r {
  background-color: skyblue;
}

.footer {
  position: fixed;
  z-index: 1;
  bottom: 0;
  width: 100%;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: white;
}

.msg-input {
  width: 70%;
  margin-right: 10px;
  padding: 5px 15px;
  border: 1px solid gray;
  border-radius: 25px;
  background-color: whitesmoke;
}

.btn-submit {
  padding: 6px;
  border: none;
  border-radius: 5px;
  background-color: deepskyblue;
  color: white;
}
</style>
