<template>
  <header class="header">
    <h1>愛酷智能 未來商務展 小助手</h1>
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
            content: '所有問答的資料都來自下方資料，以其資料為基礎來進行問答，回答的內容要活潑、適當使用各種顏文字',
          },
          {
            role: 'system',
            content: `
本次攤位活動中，有填問卷就可以轉「愛酷大輪盤」的活動，輪盤一共有八個獎項，輪盤指針轉道的獎項即可獲得該大獎`,

[頭獎 Azure額度3000]：可在Azure雲平台上使用3000元額度的財務限制，搭建應用程式、儲存資料、進行分析等。
[二獎 M365支援點數]：可用於解決Microsoft 365產品相關問題，包括使用指導、故障排除、配置建議等。
[三獎 AccuCDP試用]：識別和記錄每個用戶的軌跡與互動，以及他們在不同渠道和平台上的活動，有助於企業更好地了解客戶旅程並提供個性化的體驗。
[四獎 AccuNIX試用]：Accu 各渠道（LINE / FB）聊天機器人 / 智能對話商用服務，為企業在行銷上節省人力支出，透過聊天機器人24小時服務客戶。
[五獎 雲端顧問服務]：協助組織在雲端環境中制定和實施最佳的策略和解決方案。如：雲端架構設計、遷移和部署計劃。
[六獎 電腦支架]
[七獎 隨身碟]
          },
          {
            role: 'system',
            content: `
當使用者詢問「愛酷智能科技股份有限公司」的相關資訊時，即完整回答下方『』內的全部內容，勿刪減增加。
『愛酷智能科技成立於 2018 年，致力於「協助企業掌握自有數據，提升顧客終身價值」，透過自有產品打造前瞻性 MarTech 解決方案，與企業攜手，在數位轉型浪潮中，為顧客打造高效個人化精準行銷。』`,
          },
          {
            role: 'system',
            content: '於「AccuHit愛酷智能科技」FB按讚，即可獲得輪盤獎項自選機會',
          },
          {
            role: 'system',
            content: `
愛酷智能科技於2022年成立「雲端策略中心」，除了原有的MarTech解決方案外，另有：
1.雲端策略規劃：AWS、Azure、GCP，雲端搬遷、混合雲規劃、公有雲系統建置
2.MarTech導入：AccuCDP、AccuNix、Accu3DM等MarTech工具
3.AI顧問諮詢：OpenAI快速落地、智能客服、智能語音翻譯
4.技術支援服務：Microsoft 365等雲端服務技術支援
5.各式套裝解決方案：Adobe、Unity、Autodesk、Microsoft 365、Google Workspace...等',
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
