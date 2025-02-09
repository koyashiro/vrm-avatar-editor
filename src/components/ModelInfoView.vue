<template>
  <div v-if="vrmObject">
    <v-simple-table>
      <tbody>
        <tr v-if="vrmObject.children !== null">
          <td>Mesh Count</td>
          <td>{{ getMeshCount(vrmObject.children) }}</td>
          <td></td>
        </tr>
        <tr v-if="materials !== null">
          <td>SubMesh Count</td>
          <td>{{ materials.length }}</td>
          <td></td>
        </tr>
        <tr v-if="vrmObject.children !== null">
          <td>Polygon Count</td>
          <td>{{ getPolygonCount(vrmObject.children) }}</td>
          <td></td>
        </tr>
        <tr v-if="vrmObject.children !== null">
          <td>Bone Count</td>
          <td>{{ getBoneCount(vrmObject.children) }}</td>
          <td><ShowBoneButton /></td>
        </tr>
        <tr v-if="vrmObject.children !== null">
          <td>BlendShape Count</td>
          <td>{{ getBlendShapeCount(vrmObject.children) }}</td>
          <td></td>
        </tr>
      </tbody>
    </v-simple-table>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";
import * as THREE from "three";

import { Arrays, VRMSkinnedMesh, VRMGroup } from "@/scripts/VRMInterface";

import ShowBoneButton from "@/components/ShowBoneButton.vue";

@Component({
  components: {
    ShowBoneButton,
  },
})
export default class ModelInfoView extends Vue {
  @Prop()
  public materials!: THREE.Material[] | null;

  @Prop()
  public vrmObject!: THREE.Scene | THREE.Group | null;

  public getMeshCount(objects: Arrays): Number {
    return objects.filter((object) =>
      ["Group", "SkinnedMesh"].includes(object.type)
    ).length;
  }

  public getPolygonCount(objects: Arrays): Number {
    return objects
      .filter((object) => ["Group", "SkinnedMesh"].includes(object.type))
      .map((object) => {
        if (object.type === "Group") {
          return (object.children[0] as VRMSkinnedMesh).geometry.attributes
            .position.count;
        } else {
          return (object as VRMSkinnedMesh).geometry.attributes.position.count;
        }
      })
      .reduce((sum, value) => sum + value, 0);
  }

  public getBoneCount(objects: Arrays): Number {
    return this.getChildrenCount(
      objects.filter(
        (object) =>
          object.children.length > 0 && object.children[0].type === "Bone"
      )[0]
    );
  }

  public getChildrenCount(parent: THREE.Object3D | THREE.Bone): number {
    if (parent.children.length === 0) return 0;
    else {
      return (
        parent.children
          .map((child) => this.getChildrenCount(child))
          .reduce((sum: number, value: number) => sum + value, 0) +
        parent.children.length
      );
    }
  }

  public getBlendShapeCount(objects: Arrays): Number {
    let count = 0;
    objects.forEach((object) => {
      if (object.type === "Group") {
        const userData = (object as VRMGroup).children[0].geometry.userData;
        count += userData.targetNames ? userData.targetNames.length : 0;
      } else if (object.type === "SkinnedMesh") {
        const userData = (object as VRMSkinnedMesh).geometry.userData;
        count += userData.targetNames ? userData.targetNames.length : 0;
      }
    });
    return count;
  }
}
</script>
<style></style>
