<template>
  <ion-page>
    <ion-content 
        :fullscreen="true" 
        @click="settings=!settings">

      <div 
        id="container"
        @mouseover="showPanel"
        >
        
        <div 
          class="content">
          <ion-content >
            <iframe 
              v-if="settings.dataReady"
              :src="'https://display24.ru/device?code=' + settings.displayCode"
              :style="{ width: '100%', height: '100%' }" 
              scrolling="yes" 
              :key="iframeKey">
            </iframe>
          </ion-content>
        </div>

        <div class="logo" v-if="!settings.dataReady">
            <ion-img
              :src="getPartnerLogo()"
              :alt="settings.partnerName"
              style="width: 250px; margin-bottom: 30px;"
            ></ion-img>
            <div style="font-size: 32pt; margin-bottom: 10px;">Display24</div>
            <div class="logo-no-code">
              <div style="font-size: 14pt; color: #fff;">Необходимо установить параметры дисплея</div>
            </div>    
        </div>

        <div class="panel" v-if="panel || !settings.dataReady">
          <ion-button @click.prevent="refreshGo" shape="round" fill="solid" color="primary">
            <ion-icon slot="icon-only" :icon="reload"></ion-icon>
          </ion-button>
          <ion-button @click.prevent="settingsGo" shape="round" fill="solid" color="primary">
            <ion-icon slot="icon-only" :icon="settingsOutline"></ion-icon>
          </ion-button>
        </div>

        <ion-modal :is-open="dialog">

          <ion-header>
            <ion-toolbar>
              <ion-title>Настройки приложения<br> <span style="font-size: 80%;">Режим: [{{ nodeEnv }}]</span></ion-title><br>

              <ion-buttons slot="end">
                <ion-button @click.prevent="settingsClear()" shape="round" fill="solid" color="primary" style="margin: 20px 10px 20px 20px;">
                  <ion-icon slot="icon-only" :icon="trashOutline" style="margin: 15px 0 15px 0;"></ion-icon>
                </ion-button>
                <ion-button @click.prevent="settingsSave()" shape="round" fill="solid" color="primary" style="margin: 20px 10px 20px 10px;">
                  <ion-icon slot="icon-only" :icon="saveOutline" style="margin: 15px 0 15px 0;"></ion-icon>
                </ion-button>
                <ion-button @click.prevent="settingsCancel()" shape="round" fill="solid" color="primary" style="margin: 20px 20px 20px 10px;">
                  <ion-icon slot="icon-only" :icon="closeOutline" style="margin: 15px 0 15px 0;"></ion-icon>
                </ion-button>
              </ion-buttons>
            </ion-toolbar>
          </ion-header>

          <ion-content class="ion-padding">

            <div style="width: 100%; display: flex; align-items: center; justify-content: space-between;">
              <div style="color: #fff;"><b>Проверить код дисплея: </b></div>

              <ion-item
                ref="displayCodeBorder">
                <ion-input
                  ref="displayCodeInput" 
                  :value="settings.displayCode" @input="updateInput"
                  placeholder="">
                </ion-input>
              </ion-item>

              <ion-button @click.prevent="settingsCheck()" shape="round" fill="solid" color="primary" style="margin: 0px;">
                <ion-icon slot="icon-only" :icon="checkmarkOutline"></ion-icon>
              </ion-button>

            </div>

            <div class="params">
              <table width="100%">
                <tr><td class="params-item">Партнер:</td><td class="readyData"><span ref="partnerName">{{ settings.partnerName }}</span></td></tr>
                <tr><td class="params-item">Код дисплея:</td><td class="readyData"><span ref="displayCode">{{ settings.displayCode }}</span></td></tr>
                <tr><td class="params-item">Наименование дисплея:</td><td class="readyData"><span ref="displayName">{{ settings.displayName }}</span></td></tr>
                <tr><td class="params-item">Описание дисплея:</td><td class="readyData"><span ref="displayDescription">{{ settings.displayDescription }}</span></td></tr>
                <tr><td class="params-item">Группа дисплея:</td><td class="readyData"><span ref="displayGroup">{{ settings.displayGroup }}</span></td></tr>
                <tr><td class="params-item">Макет:</td><td class="readyData"><span ref="layoutName">{{ settings.layoutName }}</span></td></tr>
              </table>
            </div>

            <div ref="error" style="color: orange; width: 100%; text-align: center; margin-top: 20px;"></div>

          </ion-content>

        </ion-modal>

      </div>
    </ion-content>
  </ion-page>
</template>

<script setup>
import { defineComponent, defineModel, onMounted, reactive, ref, watch } from 'vue';
import { IonContent, IonHeader, IonPage, IonModal, IonTitle, IonToolbar, IonButton, IonButtons, IonRippleEffect,
  IonFabList, IonFab, IonFabButton, IonIcon, IonInput, IonItem, IonImg, modalController, useIonRouter } from '@ionic/vue';
import { add, save, reload, saveOutline, closeOutline, settingsOutline, checkmarkOutline, trashOutline } from 'ionicons/icons';

const ionRouter = useIonRouter();
const iframeKey = ref(0)

const nodeEnv = ref(process.env.NODE_ENV)
const serverPath = process.env.NODE_ENV == 'development' ? 'http://localhost:5000' : 'https://device.display24.ru'

var settings = reactive({
  partnerId: null,
  partnerName: null,
  partnerLogo: null,
  
  displayCode: null,
  displayCodeInput: null,
  displayName: null,
  displayDescription: null, 
  displayGroup: null,
  displayToken: null,

  layoutName: null, 
  osVersion: null,
  dataReady: false,
})

const displayCodeInput = ref('')
const displayCodeBorder = ref('')
const displayCode = ref('')
const partnerName = ref('')
const displayName = ref('')
const displayDescription = ref('')
const displayGroup = ref('')
const layoutName = ref('')

const dialog = ref(false)
const panel = ref(false)
const error = ref('')

const getPartnerLogo = () => {
  if (settings.partnerLogo && settings.partnerLogo.length) {
    return 'https://storage.yandexcloud.net/d24/partners/' + settings.partnerId + '/' + settings.partnerLogo
  } else {  
    return 'img/logo_512.png'
  }              
}

const refreshGo = () => {
  iframeKey.value++
}

const settingsGo = () => {
  dialog.value = true
  
  setTimeout(() => {
    const settingsLocalStorage = localStorage.getItem("settings", null)
    settings = settingsLocalStorage ? JSON.parse(localStorage.getItem("settings"), null) : {}

    displayCode.value.innerText = settings.displayCode ? settings.displayCode : '-'
    partnerName.value.innerText = settings.partnerName ? settings.partnerName : '-'
    displayName.value.innerText = settings.displayName ? settings.displayName : '-'
    displayDescription.value.innerText = settings.displayDescription ? settings.displayDescription : '-'
    displayGroup.value.innerText = settings.displayGroup ? settings.displayGroup : '-'
    layoutName.value.innerText = settings.layoutName ? settings.layoutName : '-'
  }, 1000)
}

const loadSettingsFail = (e) => {
    partnerName.value.innerText = '-'
    displayName.value.innerText = '-'
    displayDescription.value.innerText = '-'
    displayGroup.value.innerText = '-'
    layoutName.value.innerText = '-'
    
    displayCodeBorder.value.$el.style.borderRadius = '50px'
    displayCodeBorder.value.$el.style.border = '3px solid orange'

    error.value.innerText = "Ошибка: " + e
 }

const updateInput = ($event) => {
  settings.displayCodeInput = $event.target.value;
  displayCodeInput.value.innerText = $event.target.value;
}

const settingsCheck = () => {
  const val = displayCodeInput.value && displayCodeInput.value.$el && displayCodeInput.value.$el.value ? displayCodeInput.value.$el.value : ''
  loadData(val)
}

const settingsSave = () => {
  if (settings.dataReady) {
    localStorage.setItem("settings", JSON.stringify(settings))
  }
  dialog.value = false
}

const settingsClear = () => {
  localStorage.removeItem("settings")
  settings.partnerId = null
  settings.partnerName = null
  settings.partnerLogo = null
  settings.displayCode = null
  settings.displayCodeInput = null
  settings.displayName = null
  settings.displayDescription = null
  settings.displayGroup = null
  settings.displayToken = null
  settings.layoutName = null
  settings.dataReady = false

  partnerName.value.innerText = '-'
  displayCode.value.innerText = '-'
  displayName.value.innerText = '-'
  displayDescription.value.innerText = '-'
  displayGroup.value.innerText = '-'
  layoutName.value.innerText = '-'
  error.value.innerText = ''
}

const settingsCancel = () => {
  dialog.value = false
}

const showPanel = (event) => {
  if (event.clientY <= 100) {
    panel.value = true

    setTimeout(() => {
      if (!settings.displayCode) {
        panel.value = false
      }  
    }, 10000)

  } else {
    panel.value = false
  }
}

const readSettings = () => {
  if (!settings.displayCode) {
    panel.value = true
  }
}

const loadData = async (val) => {

  displayCodeBorder.value.$el.style.border = '0px solid transparent'
  error.value.innerText = ''
  
  if (!val || (val && val.length < 10)) {
    displayCodeBorder.value.$el.style.borderRadius = '50px'
    displayCodeBorder.value.$el.style.border = '3px solid orange'
    error.value.innerText = 'Указать код дисплея (10 символов)'
    return
  }

  var data = null 
  settings.dataReady = false

  // try {
      const response = await fetch(serverPath + '/api/v1/device/' + val, {
      //const response = await fetch('/api/device?code=' + val, {  
          method: 'GET',
          headers: {
              'Content-Type': 'application/json',
          },
      })

      if (!response.ok) {
        throw new Error('Request failed');
      }
      data = await response.json();
      console.log("Data: ", data)

  // } catch (e) {
  //     loadSettingsFail(e)
  //     settings.dataReady = false
  //   return
  // }

  if (data && data.data && data.data.length == 0) {
    loadSettingsFail("Код не существует")
    return
  }

  if (data && data.code && data.code == 200 && data.data.length > 0) {
    const settingsData = data.data[0]
    settings.partnerId = settingsData.partner_id
    settings.partnerName = settingsData.partner_name
    settings.partnerLogo = settingsData.partner_logo && settingsData.partner_logo.filename ? settingsData.partner_logo.filename : null
    settings.displayCode = settingsData.display_code
    settings.displayName = settingsData.display_name
    settings.displayDescription = settingsData.display_description
    settings.displayGroup = settingsData.display_group_name
    settings.displayToken = settingsData.display_token
    settings.layoutName = settingsData.layout_name

    partnerName.value.innerText = settings.partnerName
    displayCode.value.innerText = settings.displayCode
    displayName.value.innerText = settings.displayName
    displayDescription.value.innerText = settings.displayDescription
    displayGroup.value.innerText = settings.displayGroup
    layoutName.value.innerText = settings.layoutName

    settings.dataReady = true
    displayCodeBorder.value.$el.style.borderRadius = '50px'
    displayCodeBorder.value.$el.style.border = '3px solid green'

    //error.value.innerText = 'Все хорошо'

  } else {
    loadSettingsFail("Ошибка чтения настроек")
    return
  }
}

onMounted(() => {
  readSettings()
})

</script>

<style scoped>
#container {
  text-align: center;
  position: relative;
  background-color: green;
}

#container strong {
  font-size: 20px;
  line-height: 26px;
}

#container p {
  font-size: 16px;
  line-height: 22px;
  color: #8c8c8c;
  margin: 0;
}

#container a {
  text-decoration: none;
}

.content {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%; 
  height: 100vh; 
  padding-top: 0px; 
  color: #fff;
  background-color: black;
}

.logo {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;

  position: absolute;
  top: 0;
  left: 0;
  width: 100%; 
  height: 100vh; 
  padding-top: 0px; 
  color: #fff;
  background-color: black;
}

.logo-no-code {
  border: 0px dashed #999;
  border-radius: 20px;
  padding: 20px;
}

.panel {
  width: 100%; 
  display: flex; 
  justify-content: end;
  padding: 20px 20px 0 0;
  z-index: 99999;
}

ion-button {
  margin-right: 20px;
  margin-bottom: 30px;

  --border-radius: 100px;
  --border-color: #fff;
  --border-style: solid;
  --border-width: 5px;

  --padding-start: 15px;
  --padding-end: 15px;
  --padding-top: 15px;
  --padding-bottom: 15px;
  --box-shadow: 0 2px 6px 0 rgb(0, 0, 0, 0.25);
}

ion-modal {
  --border-radius: 20px;
}

ion-toolbar {
  --background: #428CFF;
}

ion-content {
  --background: #333;
}

ion-item {
  --background: #666;
  --border-radius: 50px;
  --color: #fff;
}

ion-title {
  --color: #fff;
}

.params {
  margin: 20px;
}

.params-item {
  margin-bottom: 10px;
  color: white;
  width: 30%;
  padding: 10px;
  vertical-align: top;
}

.readyData {
  color: rgb(142, 178, 207);
  padding: 10px;
  vertical-align: top;
}

</style>
