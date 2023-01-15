<template>

  <div class="page">

      <div :translucent="true">
        <div class="header">
          <p class="title" >Camera Application</p>
        </div>
      </div>
      
      <div class="content" :fullscreen="true">
        <div class="container">
          <button @click="ClickImage()">Click Image</button>
        </div>
        <div class="container">
          <img v-if="render" class="img" :src="imageUrl"/>
        </div>
        <div class="container">
          <button v-if="render" @click="Save()">Save</button>
          <button v-if="render" @click="Clear()">Clear</button>
        </div>
      </div>

  </div>

</template>
  
  <script lang="ts">
  //import { IonContent, IonHeader, IonPage, IonTitle, IonToolbar, IonImg, IonButton, IonRouter, } from '@ionic/vue';
  import { defineComponent } from 'vue';
  //import { useRouter } from 'vue-router';
  import { Camera, CameraResultType } from '@capacitor/camera';
  import { Filesystem, Directory, Encoding } from '@capacitor/filesystem';

  
  export default defineComponent({
    
    name: 'Home',

    data(){ 
      return {
        imageUrl : '',
        render: false
      }
    },
    
    methods: {
    
      async Clear() {
        this.imageUrl = '';
        this.render = false;
      },
      
      async ClickImage() {
        await Camera.getPhoto({
          quality: 90,
          allowEditing: false,
          resultType: CameraResultType.Uri
          }).then((image) => {
            this.render = true;
            this.imageUrl = String(image.webPath)
        });
      },
      
      convertBlobToBase64 (blob: Blob) {
        return new Promise((resolve, reject) =>{
          const reader = new FileReader;
          reader.onerror = reject;
          reader.onload = () => {
            resolve(reader.result);
          };
          reader.readAsDataURL(blob);
        });
      },
      
      async Save() {
          const response = await fetch(this.imageUrl);
          const blob = await response.blob();
          const base64Data = await this.convertBlobToBase64(blob) as string;
          console.log('url', base64Data);
          const savedFile = await Filesystem.writeFile({
              path: new Date().getTime() + '.jpeg',
              data: base64Data,
              directory: Directory.Documents
        });
      }
    }
  });
  </script>
  
  <style scoped>
  .container {
    flex: 1;
    display: flex;
    width: 100%;
    justify-content: center;
    align-content: center;
    margin-top: 12px;
  }
  .img {
    height: 300px;
    width: 300px;
  }
  </style>