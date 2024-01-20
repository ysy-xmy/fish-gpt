<template>
  <div class="flex flex-col min-h-screen">
    <div class="flex flex-nowrap fixed w-full items-baseline top-0 px-6 py-4 bg">
      <div class="text-2xl font-bold">GPT-3.5</div>
      <el-tag class="ylj" type="success" size="small">已连接</el-tag>
      <!-- <div class="ml-4 text-sm text-gray-500">2023.11.26</div> -->
    </div>

    <!-- <div class="flex-1 mx-2 mt-20 mb-2" ref="chatListDom"> -->
   
    <div class="flex-1 mx-2 mt-20 mb-2  kd" ref="chatListDom">
      <!-- 颜色 -->
      <!--:class="key % 2 === 0 ? 'bg-blue-400' : 'bg-indigo-400'" -->
      <div v-for="(item, key) of messageList.filter((v) => v.role !== 'system')" :key="key"
        class="group flex flex-col px-4 py-3 rounded-lg">
        <!-- shadow-md -->
        <div class="flex flex-start items-center mb-2">
          <!-- AI头像 -->
          <div v-if="key % 2 !== 0" style="margin-right: 10px">
              <img class="w-8 h-8" src="/chatgpt.svg">
            
          </div>

          <!-- 用户头像 -->
          <div v-if="key % 2 === 0" style="margin-right: 10px">
            <svg t="1700498026576" class="icon" viewBox="0 0 1024 1024" version="1.1" xmlns="http://www.w3.org/2000/svg"
              p-id="2717" width="32" height="32">
              <path d="M632 344m-120 0a120 120 0 1 0 240 0 120 120 0 1 0-240 0Z" fill="#A9D8FF" p-id="2718"></path>
              <path d="M809 800H503c-48.1 0-87-39-87-87 0-111 90-201 201-201h78c111 0 201 90 201 201 0 48-39 87-87 87z"
                fill="#A9D8FF" p-id="2719"></path>
              <path d="M440 296m-168 0a168 168 0 1 0 336 0 168 168 0 1 0-336 0Z" fill="#298DF7" p-id="2720"></path>
              <path
                d="M638.2 896H241.8C179 896 128 845 128 782.2 128 633 249 512 398.2 512h83.6C631 512 752 633 752 782.2 752 845 701 896 638.2 896z"
                fill="#298DF7" p-id="2721"></path>
            </svg>
          </div>

          <!-- 输入内容 -->
          <div class="bt font-bold" style="font-size: 16px">
            {{ roleAlias[item.role] }}
          </div>
        </div>
        <div>
          <div   class="prose  break-words text-sm text-black tracking-wider leading-relaxed" v-if="item.content" v-html="md.render(item.content)">
              
          </div>
          <Loading v-else />
          <Copy style="margin-top: 10px" class="group-hover:visible" :content="item.content" v-if="key % 2 !== 0" />
        </div>
      </div>
    </div>
 
    <div class=" bottom-0 pt-8   w-screen">
      <el-divider style="margin: 0;" class="m-0" />
      
      <div class="flex w-screen p-3 pt-4 flex-nowrap">
        <el-input round size="large" style="width: 95%; margin-right: 30px" v-model="messageContent"
          placeholder="Message..." @keydown.enter="sendChatMessage()" />
        <el-button round type="primary" size="large" @click="sendChatMessage()" :loading="isTalking">发 送</el-button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import type { ChatMessage } from "@/types";
// @ts-ignore
import { ref, watch, nextTick, onMounted } from "vue";
import Loading from "@/components/Loading.vue";
// @ts-ignore
import Copy from "@/components/Copy.vue";
import { md } from "@/libs/markdown";
// @ts-ignore
import CryptoJS from "crypto-js";
// @ts-ignore
import * as base64 from "base-64";
// @ts-ignore
import { ElMessage, ElMessageBox } from "element-plus";
//@ts-ignore
import axios from 'axios'
let isTalking = ref(false);
let messageContent = ref("");
const socket = ref<any>(null);

const apiData = ref({
  APPID: "26e880bc",
  APISecret: "NzgzZWRkOWIyMDgyOGJkNjM1Y2ViNjA5",
  APIKey: "980d8273e9b7f900a6ffbb56f53199b6",
});

const chatListDom = ref<HTMLDivElement>();
const roleAlias = {
  user: "You",
  assistant: "GPT",
  system: "System",
};
const messageList = ref<ChatMessage[]>([
  {
    role: "system",
    content: "我是邵奥建立的模型，尽可能简洁地回答。",
    // content: "我是讯飞星火大模型，尽可能简洁地回答。",
  },
  {
    role: "assistant",
    content: `How can I help you today?\n\n`
      + `我可以回答问题，高效便捷地帮助人们获取信息、知识和灵感。`,
  },
]);

onMounted(() => {
  // verifyTokenInfo();
});

localStorage.setItem("tokenInfo", "woaiwangrui");

const checkTokenInfo = () => {
  return new Promise((resolve, reject) => {
    ElMessageBox.prompt("password:woaiwangrui", "提示", {
      confirmButtonText: "确定",
      cancelButtonText: "取消",
    })
      .then(({ value }) => {
        if (value === "woaiwangrui") {
          localStorage.setItem("tokenInfo", "woaiwangrui");
          // @ts-ignore
          resolve();
        } else {
          reject(new Error("校验失败"));
        }
      })
      .catch(() => {
        reject(new Error("校验失败"));
      });
  });
};

const verifyTokenInfo = async () => {
  let tokenInfo = localStorage.getItem("tokenInfo");

  while (tokenInfo !== "woaiwangrui") {
    try {
      await checkTokenInfo();
      tokenInfo = "woaiwangrui";
      ElMessage({
        type: "success",
        message: `校验成功`,
      });
    } catch (error) {
      tokenInfo = "";
      ElMessage({
        type: "error",
        message: `校验失败`,
      });
    }
  }
};

const sendChatMessage = async (content: string = messageContent.value) => {
  //传入默认值，并说明类型
  let tokenInfo = localStorage.getItem("tokenInfo");


  if (tokenInfo === "woaiwangrui") {
    try {
      if (!messageContent.value.length) return;
      //设置在对话的状态

      isTalking.value = true;
      if (messageList.value.length === 2) {
        //删了一开始打招呼内容
        messageList.value.pop();
      }

      messageList.value.push({ role: "user", content });
      //简写了content:content
      messageList.value.push({ role: "assistant", content: "" });

      callSparkModel();

    } catch (error: any) {
      appendLastMessageContent(error);
    }
  } else {
    verifyTokenInfo();
  }
};

// 1.5模型
// const getWebSocketUrl = () => {
//   return new Promise((resolve, reject) => {
//     let url = "wss://spark-api.xf-yun.com/v1.1/chat";
//     let host = "spark-api.xf-yun.com";
//     let apiKeyName = "api_key";
//     // @ts-ignore
//     let date = new Date().toGMTString();
//     let algorithm = "hmac-sha256";
//     let headers = "host date request-line";
//     let signatureOrigin = `host: ${host}\ndate: ${date}\nGET /v1.1/chat HTTP/1.1`;
//     let signatureSha = CryptoJS.HmacSHA256(
//       signatureOrigin,
//       apiData.value.APISecret
//     );
//     let signature = CryptoJS.enc.Base64.stringify(signatureSha);
//     let authorizationOrigin = `${apiKeyName}="${apiData.value.APIKey}", algorithm="${algorithm}", headers="${headers}", signature="${signature}"`;
//     let authorization = base64.encode(authorizationOrigin);
//     url = `${url}?authorization=${authorization}&date=${encodeURI(
//       date
//     )}&host=${host}`;

//     resolve(url);
//   });
// };

// 2.0模型
const getWebSocketUrl = () => {
  return new Promise((resolve, reject) => {
    let url = "wss://spark-api.xf-yun.com/v2.1/chat";
    let host = "spark-api.xf-yun.com";
    let apiKeyName = "api_key";
    // @ts-ignore
    let date = new Date().toGMTString();
    let algorithm = "hmac-sha256";
    let headers = "host date request-line";

    // 注意这里的只变GET后的版本号 比如：1.1 2.1 3.1 后面的HTTP不变
    let signatureOrigin = `host: ${host}\ndate: ${date}\nGET /v2.1/chat HTTP/1.1`;
    let signatureSha = CryptoJS.HmacSHA256(
      signatureOrigin,
      apiData.value.APISecret
    );
    let signature = CryptoJS.enc.Base64.stringify(signatureSha);
    let authorizationOrigin = `${apiKeyName}="${apiData.value.APIKey}", algorithm="${algorithm}", headers="${headers}", signature="${signature}"`;
    let authorization = base64.encode(authorizationOrigin);
    url = `${url}?authorization=${authorization}&date=${encodeURI(
      date
    )}&host=${host}`;

    resolve(url);
  });
};

const callSparkModel = async () => {
 



  // axios.request(config)
  //   .then((response) => {
  //     clearMessageContent();
  //     console.log(response.data)
  //     appendLastMessageContent(response.data);
  //     isTalking.value = false;
  //   })
  //   .catch((error) => {
  //     console.log(error);
  //   });
//修
    // const res = await  axios.request(config)
    // console.log(res.data)
    // // appendLastMessageContent(res.data)
    // return new Response(res.body, {
    //     headers: {
    //         "Access-Control-Allow-Origin": "*",
    //     },
    //     status: 200,
    // });



    const headers = {
        'Content-Type': 'application/json',
        'Host': 'api.binjie.fun',
        'Origin': 'https://chat18.aichatos.xyz'
    };
    const payload = {
      "prompt": messageContent.value,
      "userId": "#/chat/1705550096732",
      "network": true,
      "system": "",
      "withoutContext": false,
      "stream": true
    }
    const res = await fetch(`https://www.oboard.eu.org/api/gpt?prompt=${messageContent.value}&userId=123`, {
        method: 'GET',
    });

    if (!res.body) return;
  const reader = res.body.pipeThrough(new TextDecoderStream()).getReader();
  while (true) {
    var { value, done } = await reader.read();
    if (done){
      setTimeout(() => {
        clearMessageContent();
          isTalking.value = false;
        }, 1000);
      break;
    } 
 
    let outvalue:string = value?.replace('undefined', '') || '';
    // console.log("received data -", value)
    appendLastMessageContent(outvalue);
  }
 
















  // socket.value.onopen = (e: any) => {
  //   let params = {
  //     header: {
  //       app_id: apiData.value.APPID,
  //       uid: "aef9f963-7",
  //     },
  //     parameter: {
  //       // 1.5 参数
  //       // chat: {
  //       //   domain: "general",
  //       //   temperature: 0.5,
  //       //   max_tokens: 1024,
  //       // },

  //       // 2.0 参数
  //       chat: {
  //         domain: "generalv2",
  //         temperature: 0.5,
  //         max_tokens: 3072,
  //       },
  //     },
  //     payload: {
  //       message: {
  //         text: [
  //           {
  //             role: "user",
  //             content: messageContent.value,
  //           },
  //         ],
  //       },
  //     },
  //   };

  //   socket.value.send(JSON.stringify(params));
  //   //清空输入框的内容
  //   
  // };

  // socket.value.onmessage = (e: any) => {
  //   let obj = JSON.parse(e.data);
  //   console.log(obj)

  //   let dataArray = obj.payload.choices.text;

  //   for (let i = 0; i < dataArray.length; i++) {
  //     appendLastMessageContent(dataArray[i].content);
  //   }

  //   let temp = JSON.parse(e.data);
  //   if (temp.header.code !== 0) {
  //     isTalking.value = false;
  //     socket.value.close();
  //   }
  //   if (temp.header.code === 0) {
  //     if (e.data && temp.header.status === 2) {
    
      
  //   }
  // };
  // console.log(socket.value)
};

const appendLastMessageContent = (content: string) => {
  // let outcontent=content.replace(/\n/g, '<br>')
  messageList.value[messageList.value.length - 1].content += content;
};

const clearMessageContent = () => {
  messageContent.value = "";
};

const scrollToBottom = () => {
  if (!chatListDom.value) return;
  scrollTo(0, chatListDom.value.scrollHeight);
};

watch(messageList.value, () => nextTick(() => scrollToBottom()));
</script>

<style scoped>
pre {
  font-family: -apple-system, "Noto Sans", "Helvetica Neue", Helvetica,
    "Nimbus Sans L", Arial, "Liberation Sans", "PingFang SC", "Hiragino Sans GB",
    "Noto Sans CJK SC", "Source Han Sans SC", "Source Han Sans CN",
    "Microsoft YaHei", "Wenquanyi Micro Hei", "WenQuanYi Zen Hei", "ST Heiti",
    SimHei, "WenQuanYi Zen Hei Sharp", sans-serif;
}

.prose :where(ol > li):not( :where([class~="not-prose"], [class~="not-prose"] *))::marker {
  color: white;
}

.bt {
  display: flex;
}

.kd {
  margin: auto;
  margin-top: 60px;
  width: 550px;
  /* margin-left: 300px ;
  margin-right: 300px ; */

}

@media (max-width: 767px) {
  .kd {
    /* margin: auto; */
    width: 390px;
  }
}

.bg {
  backdrop-filter: blur(20px);
  /* 调整模糊程度，单位可以是像素(px)或其他合适的值 */
}

.ylj {
  translate: 2px -10px;
}
</style>
