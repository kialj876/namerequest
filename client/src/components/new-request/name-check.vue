<template>
  <div id="name-check-main-div">
    <MainContainer id="name-check-name-input">
      <template v-slot:container-header>
        <v-row no-gutters>
          <v-col cols="auto" class="h6 py-0 mt-1">
            You are searching for a name for a
            {{ entityText === ' BC Corporation' && location.text === ' BC' ? '' : ' ' + location.text }}
            {{ entityText }}
          </v-col>
        </v-row>
      </template>
      <template v-slot:content>
        <v-row no-gutters class="pt-5">
          <v-col cols="6">
            <NameInput/>
          </v-col>
          <v-col cols="3">
            <v-select :items="designationOptions"
                      label="Select a Designation"
                      filled
                      v-model="designation">
            </v-select>
          </v-col>
          <v-col cols="3" class="pl-3 pt-2">
            <v-btn id="search-name-btn" @click="checkAgain()">
              <v-icon left color="white">mdi-magnify</v-icon>
              Check This Name
            </v-btn>
          </v-col>
        </v-row>
      </template>
    </MainContainer>

    <v-container id="name-check-lower-container" class="white rounded copy-normal pa-10 mt-3">
      <v-row justify="center" no-gutters>
        <v-col cols="auto" id="name-check-header">
          <b>Showing Results for:</b>
        </v-col>
      </v-row>
      <v-row justify="center" no-gutters>
        <v-col cols="auto" id="name-check-title">
          <b>{{originalName}}</b>
        </v-col>
      </v-row>
      <v-row no-gutters justify="center">
        <v-col>
          <template>
            <v-row justify="center">
              <v-col cols="12">
                <v-tabs v-model="checks" grow height="7rem" no-gutters>
                  <v-row no-gutters>
                    <v-col cols="6">
                      <v-tab href="#structure-check" class="check-tab">
                        <v-row no-gutters justify="center">
                          <v-col>
                            <v-icon style="border-radius: 5px" size="2.2rem" :color="structureTabIcon.color">
                              {{ structureTabIcon.icon }}
                            </v-icon>
                            <p class="check-tab-title pt-1">Name Structure Check</p>
                            <p class="check-tab-sub-title pt-2 pb-7">{{ structureSubTitle }}</p>
                          </v-col>
                        </v-row>
                      </v-tab>
                    </v-col>
                    <v-col cols="6">
                      <v-tab href="#conflicts-check" class="check-tab">
                        <v-row no-gutters justify="center">
                          <v-col>
                            <v-icon size="2.2rem"  :color="conflictTabIcon.color">
                              {{ conflictTabIcon.icon }}
                            </v-icon>
                            <p class="check-tab-title pt-1">Existing Name Check</p>
                            <p class="check-tab-sub-title pt-2 pb-7">{{ conflictSubTitle }}</p>
                          </v-col>
                        </v-row>
                      </v-tab>
                    </v-col>
                  </v-row>
                </v-tabs>
              </v-col>
            </v-row>
            <v-row>
              <v-col>
                <v-tabs-items v-model="checks">
                  <v-tab-item value="structure-check">
                    <NameCheckConflicts :items="structureItems" />
                  </v-tab-item>
                  <v-tab-item value="conflicts-check">
                    <NameCheckConflicts :items="conflictItems" />
                  </v-tab-item>
                </v-tabs-items>
              </v-col>
            </v-row>
          </template>
        </v-col>
      </v-row>
      <v-row no-gutters justify="center">
        <v-col cols="4">
          <v-row no-gutters justify="center">
            <v-btn id="name-check-submit-btn"
                  @click="submit()">
              Submit this Name for Review
              <v-icon>mdi-chevron-right</v-icon>
            </v-btn>
          </v-row>
          <v-row no-gutters id="buttonInfoText" justify="center" class="pt-2">
            {{ buttonInfoText }}
          </v-row>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script lang="ts">
import MainContainer from '@/components/new-request/main-container.vue'
import NameCheckConflicts from '@/components/new-request/name-check-conflicts.vue'
import NameInput from '@/components/new-request/name-input.vue'
import QuickSearchNames from '@/components/new-request/quick-search-names.vue'
import newReqModule from '@/store/new-request-module'
import { Component, Prop, Vue, Watch } from 'vue-property-decorator'

@Component({
  components: { MainContainer, NameCheckConflicts, NameInput, QuickSearchNames }
})
export default class NameCheck extends Vue {
  config = {
    modules: {
      toolbar: false
    },
    placeholder: '',
    scrollingContainer: false
  }

  private checks: Array<string> = ['structure-check', 'conflicts-check']
  private numCondConflicts: number = 0
  private numExactConflicts: number = 0
  private numRestConflicts: number = 0
  private numSimConflicts: number = 0
  private originalName = ''

  mounted () {
    this.originalName = this.searchName?.toUpperCase()
  }

  @Watch('conditionalConflicts')
  updateCondConflicts (val) {
    this.numCondConflicts = val.length
  }
  @Watch('exactConflicts')
  updateExactConflicts (val) {
    this.numExactConflicts = val.length
  }
  @Watch('restrictedConflicts')
  updateRestConflicts (val) {
    this.numRestConflicts = val.length
  }
  @Watch('similarConflicts')
  updateSimConflicts (val) {
    this.numSimConflicts = val.length
  }

  get designation () {
    return newReqModule.designation
  }
  set designation (value: string) {
    newReqModule.mutateDesignation(value)
  }
  get designationOptions () {
    return newReqModule.designationOptions
  }
  get conditionalConflicts () {
    const conflicts = newReqModule.conditionalConflicts
    this.numCondConflicts = conflicts.length
    console.log('cond ', conflicts) // eslint-disable-line
    return conflicts
  }
  get exactConflicts () {
    const conflicts = newReqModule.exactConflicts
    this.numExactConflicts = conflicts.length
    console.log('exact ', conflicts) // eslint-disable-line
    return conflicts
  }
  get restrictedConflicts () {
    const conflicts = newReqModule.restrictedConflicts
    this.numRestConflicts = conflicts.length
    console.log('rest ', conflicts) // eslint-disable-line
    return conflicts
  }
  get similarConflicts () {
    const conflicts = newReqModule.similarConflicts
    this.numSimConflicts = conflicts.length
    console.log('sim', conflicts) // eslint-disable-line
    return conflicts
  }
  get entityText () {
    return newReqModule.entityTextFromValue
  }
  get location () {
    let value = newReqModule.location
    let options = newReqModule.locationOptions
    return options.find((opt: any) => opt.value === value)
  }
  get searchName () {
    return newReqModule.fullName
  }
  get structureTabIcon (): { color: string, icon: string } {
    if (this.numCondConflicts + this.numRestConflicts > 0) return { color: 'orange darken-2', icon: 'mdi-alert' }
    return { color: 'green', icon: 'mdi-check-circle' }
  }
  get conflictTabIcon (): { color: string, icon: string } {
    if (this.numExactConflicts > 0) return { color: 'red', icon: 'mdi-alert-octagon' }
    else if (this.numSimConflicts > 0) return { color: 'orange darken-2', icon: 'mdi-alert' }
    return { color: 'green', icon: 'mdi-check-circle' }
  }
  get structureSubTitle (): string {
    if (this.numCondConflicts + this.numRestConflicts > 0) {
      return `${this.numCondConflicts + this.numRestConflicts} possible issues found`
    }
    return 'no issues found'
  }
  get conflictSubTitle (): string {
    let message = ''
    if (this.numExactConflicts > 0) message += `${this.numExactConflicts} exact matches`
    if (this.numSimConflicts > 0 && message) message += `, ${this.numSimConflicts} similar names`
    else if (this.numSimConflicts > 0) message += `${this.numSimConflicts} similar names`
    // 0 conflicts
    else message += 'no similar names'
    return `${message} found`
  }
  get conflictItems (): Array<{
    count: number,
    expanded: string,
    expandedList: Array<{ name: string, type: string }>,
    icon: string,
    iconColor: string,
    problem: string,
    words: Array<string>,
  }> {
    let items = []
    if (this.numExactConflicts > 0) {
      items.push({
        icon: 'mdi-alert-octagon',
        iconColor: 'red',
        problem: 'There is an existing BC Corporations using the <b>exact name:</b>',
        words: null,
        expanded: '',
        expandedList: this.exactConflicts,
        expandLabel: {
          closed: 'view',
          open: 'hide'
        },
        count: this.numExactConflicts
      })
    }
    if (this.numSimConflicts > 0) {
      items.push({
        icon: 'mdi-alert',
        iconColor: 'orange darken-2',
        problem: 'There are existing BC Corporations using the <b>same or similar words</b>:',
        words: null,
        expanded: '',
        expandedList: this.similarConflicts,
        expandLabel: {
          closed: 'view',
          open: 'hide'
        },
        count: this.numSimConflicts
      })
    }
    if (items.length === 0) {
      items.push({
        icon: 'mdi-check-circle',
        iconColor: 'green',
        problem: 'No similar names found',
        words: null,
        expanded: null,
        expandedList: null,
        expandLabel: null,
        count: null
      })
    }
    console.log('conf items ', items) // eslint-disable-line
    return items
  }
  get structureItems (): Array<{
    count: number,
    expanded: string,
    expandedList: Array<{ name: string, type: string }>,
    icon: string,
    iconColor: string,
    problem: string,
    words: Array<string>,
  }> {
    let items = []
    if (this.numCondConflicts > 0) {
      items.push({
        icon: 'mdi-alert',
        iconColor: 'orange darken-2',
        problem: 'You may need consent to use the following words:',
        words: this.conditionalConflicts,
        expanded: null,
        expandedList: null,
        expandLabel: {
          closed: 'read more',
          open: 'read less'
        },
        count: null
      })
    }
    if (this.numRestConflicts > 0) {
      items.push({
        icon: 'mdi-alert',
        iconColor: 'orange darken-2',
        problem: 'Your name appears to be using the following words to avoid:',
        words: this.restrictedConflicts,
        expanded: null,
        expandedList: null,
        expandLabel: {
          closed: 'read more',
          open: 'read less'
        },
        count: null
      })
    }
    console.log('struct items ', items) // eslint-disable-line
    if (items.length === 0) {
      items.push({
        icon: 'mdi-check-circle',
        iconColor: 'green',
        problem: 'No issues have been found with your name structure',
        words: null,
        expanded: null,
        expandedList: null,
        expandLabel: null,
        count: null
      })
    }
    return items
  }
  get buttonInfoText () {
    const conflicts = this.numCondConflicts + this.numExactConflicts + this.numRestConflicts + this.numSimConflicts
    if (conflicts) return 'We recommend resolving any issues or name conflicts before submitting.'
    else return "Your name's structure and uniqueness look good, you are ready to send this name for review."
  }
  async checkAgain () {
    newReqModule.mutateQuickSearch(true)
    await newReqModule.startAnalyzeName()
  }
  async submit () {
    console.log('submitted') // eslint-disable-line no-console
  }
}
</script>

<style lang="scss">
@import '@/assets/scss/theme.scss';
#name-check-name-input {
  padding: 2rem !important;
}
#name-check-header {
  font-size: 0.9rem;
  color: $gray9;
  text-align: center;
}
#name-check-title {
  font-size: 1.5rem;
  color: $gray9;
  text-align: center;
  padding-top: 0.3125rem;
}
#name-check-lower-container {
  max-width: none;
}
#name-check-submit-btn {
  height: 2.813rem !important;
  font-size: 1rem !important;
  width: 18rem !important;
}
#buttonInfoText {
  width: 22rem;
  text-align: center;
  font-size: 0.875rem;
  color: $gray7;
}
#search-name-btn {
  font-size: 1rem !important;
  height: 2.813rem !important;
  padding: 0 3.125rem !important;
  width: 13rem;
}
.check-tab {
  max-height: none;
  max-width: none;
  padding: 0;
}
.check-tab :hover {
  background-color: $gray1 !important;
}
.check-tab-title {
  font-size: 1rem !important;
  color: $gray9 !important;
  margin: 0 !important;
}
.check-tab-sub-title {
  font-size: 0.8rem !important;
  color: $gray6 !important;
  margin: 0 !important;
}
</style>
