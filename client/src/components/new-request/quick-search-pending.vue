<template>
  <MainContainer id="quick-search-pending-container">
    <template v-slot:container-header>
      <v-col cols="auto" class="font-weight-bold py-0 mt-1">
        You are searching for a name for a
        {{ entityText === ' BC Corporation' && location.text === ' BC' ? '' : ' ' + location.text }} {{ entityText }}
      </v-col>
    </template>
    <template v-slot:content>
      <v-row no-gutters class="pt-3">
        <NameInput :is-read-only="true" class="mb-n2"/>
      </v-row>
      <v-row no-gutters justify="center" class="quick-search-info pt-10">
        <b>Please wait...</b>
      </v-row>
      <v-row no-gutters justify="center" class="quick-search-title pt-3">
        <b>Checking Your Name</b>
      </v-row>
      <v-row no-gutters justify="center" class="normal-text pt-3">
        (This could take a few minutes)
      </v-row>
      <v-row no-gutters justify="center" class="spinner-row pt-10">
        <v-col cols="auto" class="pa-0 ma-0">
          <v-progress-circular id="spinner"
                               color="primary"
                               size="50"
                               indeterminate />
        </v-col>
      </v-row>
      <v-row no-gutters justify="center" class="pt-10">
        <v-btn width="8rem" @click="back">
          Cancel
        </v-btn>
      </v-row>
      <v-row no-gutters justify="center" class="pt-2">
        <v-btn class="outlined tips-btn" :ripple="false" @click="skip">
          Submit this name without checking
          <v-icon>mdi-chevron-right</v-icon>
        </v-btn>
      </v-row>
    </template>
  </MainContainer>
</template>

<script lang="ts">
import MainContainer from '@/components/new-request/main-container.vue'
import newReqModule from '@/store/new-request-module'
import NameInput from '@/components/new-request/name-input.vue'
import { Component, Vue } from 'vue-property-decorator'

@Component({
  components: { MainContainer, NameInput }
})
export default class QuickSearchPending extends Vue {
  get entityObject () {
    return newReqModule.entityTypeOptions.find((ent: any) => ent.value === this.entity_type_cd)
  }
  get entityText () {
    return newReqModule.entityTextFromValue
  }
  get entity_type_cd () {
    return newReqModule.entity_type_cd
  }
  get location () {
    let value = newReqModule.location
    let options = newReqModule.locationOptions
    return options.find((opt: any) => opt.value === value)
  }
  get request_action_cd () {
    switch (newReqModule.request_action_cd) {
      case 'new':
        return 'a new'
      case 'existing':
        return 'an existing'
      default:
        return 'a new'
    }
  }
  back () {
    newReqModule.mutateDisplayedComponent('tabs')
  }
  async skip () {
    newReqModule.mutateQuickSearch(false)
    await newReqModule.startAnalyzeName()
  }
}

</script>
<style scoped lang="scss">
@import '@/assets/scss/theme.scss';
.normal-text {
  color: $gray7;
  font-size: 0.875rem;
}
.quick-search-title {
  font-size: 1.5rem;
  color: $gray9;
  text-align: center;
  padding: 0;
  margin: 0;
  padding-top: 0.3125rem;
}
.quick-search-info {
  font-size: 1rem;
  color: $gray9;
  text-align: center;
  padding: 0;
  margin: 0;
  padding-top: 0.3125rem;
}
.spinner-row {
  padding: 0;
  margin: 0;
  padding-top: 1.5625rem;
}
</style>
