<template>
  <v-col id="quick-search-names" class="pl-2 py-2 rounded">
    <v-row no-gutters class="quickSearchNames" v-for="name of namesList" :key="name.name">
      <b v-if="name.type==='exact'">{{name.name}}</b>
      <span v-else>{{name.name}}</span>
    </v-row>
  </v-col>
</template>

<script lang="ts">
import MainContainer from '@/components/new-request/main-container.vue'
import newReqModule from '@/store/new-request-module'
import { Component, Prop, Vue, Watch } from 'vue-property-decorator'

@Component({
  components: { MainContainer }
})
export default class QuickSearchNames extends Vue {
  @Prop() namesList: Array<{ name: string, type: string }>
}

</script>

<style scoped lang="scss">
@import '@/assets/scss/theme.scss';
#quick-search-names {
    max-height: 10.5rem;
    background-color: $gray1;
    display: block;
    overflow: auto;
}
.quickSearchNames {
    font-size: 1rem;
    color: $gray7;
    padding-left: 1.5rem;
    padding-top: 0.3rem;
}
</style>
