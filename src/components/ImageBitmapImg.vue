<template>
  <div style="margin: 0 10px 0 10px; display: inline-block">
    <v-hover v-slot="{ hover }">
      <v-list-item-avatar rounded size="80" color="white" style="margin: 5px">
        <v-img v-if="imageBitmap" :src="convertImageBitmap2Base64(imageBitmap)">
          <div v-if="showInfo && hover" class="tex-info unselectable">
            {{ imageBitmap.width }}x{{ imageBitmap.height }}
          </div>
        </v-img>
        <span style="color: black" v-else> none </span>
      </v-list-item-avatar>
    </v-hover>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";
import * as THREE from "three";

interface MaterialInfo {
  name: string;
  material: THREE.Material;
  indices: Array<number>;
}

@Component
export default class ImageBitmapImg extends Vue {
  @Prop()
  public imageBitmap!: ImageBitmap;

  /** マウスカーソルをあわせたときに画像の詳細を表示するか */
  @Prop({ default: true })
  public showInfo!: boolean;

  public convertImageBitmap2Base64(image: ImageBitmap): string {
    const canvas = document.createElement("canvas");
    canvas.width = image.width;
    canvas.height = image.height;
    canvas.getContext("2d")!.drawImage(image, 0, 0);
    return canvas.toDataURL();
  }
}
</script>
<style>
.tex-info {
  font-size: 10px;
  background: grey;
  opacity: 0.75;
  width: 100%;
  height: 100%;
}

.unselectable {
  user-select: none; /* CSS3 */
  -moz-user-select: none; /* Firefox */
  -webkit-user-select: none; /* Safari、Chromeなど */
  -ms-user-select: none; /* IE10以降 */
}
</style>
