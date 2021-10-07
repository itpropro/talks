<script setup lang="ts">
import { useWebSocket, useFetch, invoke, useTimeout, promiseTimeout } from '@vueuse/core'
import { watch, ref } from 'vue'

const functionUrl = 'https://sessionfeed-message-api.azurewebsites.net/api/GetConnectionUri?code=CFUuVkHwWDUHv8R6X2KXFR5Ys05XEprmzvtNWWh/i9gYiQK76CGWkQ==&userId=slidev'
const { isFetching, error, data, onFetchResponse } = useFetch(functionUrl).get().json()
const currentMessage = ref('')
const currentLink = ref('')
const messageVisible = ref(false)
onFetchResponse(() => {
  const { status, data: webSocketData, send, open, close } = useWebSocket(data.value.url, { autoReconnect: true, protocols: ['json.webpubsub.azure.v1'] })
  watch(webSocketData, (webSocketData, prevData) => {
    const message = JSON.parse(webSocketData)
    if (message.type === 'message') {
      currentMessage.value = `Received new ${message.data.data.category}!`
      currentLink.value = message.data.data.category === 'question' ? `https://sessionfeed.app/talks/${message.data.data.content}` : `https://sessionfeed.app/talks/8cdc86a1-54a6-49be-ad0e-89b95b57db6a/${message.data.data.content}`
      invoke(async () => {
        messageVisible.value = true
        await promiseTimeout(5000)
        messageVisible.value = false
      }
      )
    }
  })
})
</script>

<template>
  <div
    v-if="messageVisible"
    class="absolute top-5 right-5 text-sm from-[#FEAC5E] via-[#C779D0] to-[#4BC0C8] bg-gradient-to-r rounded-md animate-slide-in-right animate-duration-500 p-2 text-white"
  >
    <ion-chatbubbles class="inline" />
    <span class="pl-2">{{ currentMessage }}</span>
    <div class="text-right">
      <ion-link class="inline" />
      <a :href="currentLink" target="_blank" class="pl-2">Open</a>
    </div>
  </div>
  <header v-if="$slidev.nav.currentPage !== 1" class="absolute top-6 left-6 right-0 p-2 w-40">
    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 337.6 72">
      <path
        class="st0"
        d="M140.4,14.4v43.2h-7.5V23.7h-0.1l-13.4,33.9h-5l-13.7-33.9h-0.1v33.9h-6.9V14.4h10.8l12.4,32h0.2l13.1-32H140.4
	z M146.6,17.7c0-1.2,0.4-2.2,1.3-3c0.9-0.8,1.9-1.2,3.1-1.2c1.3,0,2.4,0.4,3.2,1.2s1.3,1.8,1.3,3c0,1.2-0.4,2.2-1.3,3
	c-0.9,0.8-1.9,1.2-3.2,1.2s-2.3-0.4-3.1-1.2C147.1,19.8,146.6,18.8,146.6,17.7z M154.7,26.6v31h-7.3v-31H154.7z M176.8,52.3
	c1.1,0,2.3-0.2,3.6-0.8c1.3-0.5,2.5-1.2,3.6-2v6.8c-1.2,0.7-2.5,1.2-4,1.5c-1.5,0.3-3.1,0.5-4.9,0.5c-4.6,0-8.3-1.4-11.1-4.3
	c-2.9-2.9-4.3-6.6-4.3-11c0-5,1.5-9.1,4.4-12.3c2.9-3.2,7-4.8,12.4-4.8c1.4,0,2.8,0.2,4.1,0.5c1.4,0.3,2.5,0.8,3.3,1.2v7
	c-1.1-0.8-2.3-1.5-3.4-1.9c-1.2-0.4-2.4-0.7-3.6-0.7c-2.9,0-5.2,0.9-7,2.8s-2.6,4.4-2.6,7.6c0,3.1,0.9,5.6,2.6,7.3
	C171.6,51.4,173.9,52.3,176.8,52.3z M204.7,26.1c0.6,0,1.1,0,1.6,0.1s0.9,0.2,1.2,0.3v7.4c-0.4-0.3-0.9-0.6-1.7-0.8
	s-1.6-0.4-2.7-0.4c-1.8,0-3.3,0.8-4.5,2.3s-1.9,3.8-1.9,7v15.6h-7.3v-31h7.3v4.9h0.1c0.7-1.7,1.7-3,3-4
	C201.2,26.6,202.8,26.1,204.7,26.1z M207.9,42.6c0-5.1,1.5-9.2,4.3-12.2c2.9-3,6.9-4.5,12-4.5c4.8,0,8.6,1.4,11.3,4.3
	s4.1,6.8,4.1,11.7c0,5-1.5,9-4.3,12c-2.9,3-6.8,4.5-11.8,4.5c-4.8,0-8.6-1.4-11.4-4.2C209.3,51.3,207.9,47.4,207.9,42.6z
	 M215.5,42.3c0,3.2,0.7,5.7,2.2,7.4s3.6,2.6,6.3,2.6c2.6,0,4.7-0.8,6.1-2.6c1.4-1.7,2.1-4.2,2.1-7.6c0-3.3-0.7-5.8-2.1-7.6
	c-1.4-1.7-3.5-2.6-6-2.6c-2.7,0-4.7,0.9-6.2,2.7C216.2,36.5,215.5,39,215.5,42.3z M250.5,34.8c0,1,0.3,1.9,1,2.5
	c0.7,0.6,2.1,1.3,4.4,2.2c2.9,1.2,5,2.5,6.1,3.9c1.2,1.5,1.8,3.2,1.8,5.3c0,2.9-1.1,5.2-3.4,7c-2.2,1.8-5.3,2.6-9.1,2.6
	c-1.3,0-2.7-0.2-4.3-0.5c-1.6-0.3-2.9-0.7-4-1.2v-7.2c1.3,0.9,2.8,1.7,4.3,2.2c1.5,0.5,2.9,0.8,4.2,0.8c1.6,0,2.9-0.2,3.6-0.7
	c0.8-0.5,1.2-1.2,1.2-2.3c0-1-0.4-1.8-1.2-2.6c-0.8-0.7-2.4-1.5-4.6-2.4c-2.7-1.1-4.6-2.4-5.7-3.8s-1.7-3.2-1.7-5.4
	c0-2.8,1.1-5.1,3.3-6.9c2.2-1.8,5.1-2.7,8.6-2.7c1.1,0,2.3,0.1,3.6,0.4s2.5,0.6,3.4,0.9V34c-1-0.6-2.1-1.2-3.4-1.7
	c-1.3-0.5-2.6-0.7-3.8-0.7c-1.4,0-2.5,0.3-3.2,0.8C250.9,33.1,250.5,33.8,250.5,34.8z M266.9,42.6c0-5.1,1.5-9.2,4.3-12.2
	c2.9-3,6.9-4.5,12-4.5c4.8,0,8.6,1.4,11.3,4.3s4.1,6.8,4.1,11.7c0,5-1.5,9-4.3,12c-2.9,3-6.8,4.5-11.8,4.5c-4.8,0-8.6-1.4-11.4-4.2
	C268.4,51.3,266.9,47.4,266.9,42.6z M274.5,42.3c0,3.2,0.7,5.7,2.2,7.4s3.6,2.6,6.3,2.6c2.6,0,4.7-0.8,6.1-2.6
	c1.4-1.7,2.1-4.2,2.1-7.6c0-3.3-0.7-5.8-2.1-7.6c-1.4-1.7-3.5-2.6-6-2.6c-2.7,0-4.7,0.9-6.2,2.7C275.3,36.5,274.5,39,274.5,42.3z
	 M322.9,32.6h-10.9v25h-7.4v-25h-5.2v-6h5.2v-4.3c0-3.2,1.1-5.9,3.2-8s4.8-3.1,8.1-3.1c0.9,0,1.7,0.1,2.4,0.1s1.3,0.2,1.8,0.4v6.3
	c-0.2-0.1-0.7-0.3-1.3-0.5c-0.6-0.2-1.3-0.3-2.1-0.3c-1.5,0-2.7,0.5-3.5,1.4c-0.8,0.9-1.2,2.4-1.2,4.2v3.7h10.9v-7l7.3-2.2v9.2h7.4
	v6h-7.4v14.5c0,1.9,0.4,3.2,1,4c0.7,0.8,1.8,1.2,3.3,1.2c0.4,0,0.9-0.1,1.5-0.3c0.6-0.2,1.1-0.4,1.5-0.7v6c-0.5,0.3-1.2,0.5-2.3,0.7
	c-1.1,0.2-2.1,0.3-3.2,0.3c-3.1,0-5.4-0.8-6.9-2.4c-1.5-1.6-2.3-4.1-2.3-7.4L322.9,32.6L322.9,32.6z"
      />
      <g>
        <rect class="st1" width="34.2" height="34.2" />
        <rect x="37.8" class="st2" width="34.2" height="34.2" />
        <rect y="37.8" class="st3" width="34.2" height="34.2" />
        <rect x="37.8" y="37.8" class="st4" width="34.2" height="34.2" />
      </g>
    </svg>
  </header>
  <footer
    v-if="$slidev.nav.currentPage !== 1"
    class="absolute bottom-0 left-0 right-0 pl-8 pb-4 flex"
  >
    <img src="/azureserverlessconfheader.png" class="h-15" />
    <div class="ml-2">
      <div class="text-3xl text-black font-bold font-sans">Azure Serverless</div>
      <div class="text-xl text-gray-500 font-semibold">Conf</div>
    </div>
  </footer>
</template>

<style>
.st0 {
  fill: #737373;
}
.st1 {
  fill: #f25022;
}
.st2 {
  fill: #7fba00;
}
.st3 {
  fill: #00a4ef;
}
.st4 {
  fill: #ffb900;
}
</style>
