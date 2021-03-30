<template>
  <v-container id="name-check-main-container" no-gutters class="white pa-0 rounded">
    <v-row no-gutters class="pl-8 pt-6">
      <v-col cols="auto" class="h6 py-0 mt-1">
        You are searching for a name for a
        {{ entityText === ' BC Corporation' && location.text === ' BC' ? '' : ' ' + location.text }}
        {{ entityText }}
      </v-col>
    </v-row>
    <v-row no-gutters class="pl-8 pt-6">
      <v-col cols="6">
        <NameInput/>
      </v-col>
      <v-col cols="3" class="pl-3">
        <v-select :items="designationOptions"
                  label="Select a Designation"
                  filled
                  v-model="designation">
        </v-select>
      </v-col>
      <v-col cols="3" class="pl-3 pt-2">
        <v-btn id="search-name-btn" class="outlined" outlined @click="checkAgain()">
          <v-icon left>mdi-magnify</v-icon>
          Check This Name
        </v-btn>
      </v-col>
    </v-row>
    <v-container id="name-check-lower-container" class="rounded copy-normal mt-10 pa-0 name-check">
      <v-row no-gutters justify="center" class="pt-12">
        <v-col cols="auto" id="name-check-header">
          <b>Showing Results for:</b>
        </v-col>
      </v-row>
      <v-row no-gutters justify="center">
        <v-col cols="auto" id="name-check-title">
          <b>{{originalName}}</b>
        </v-col>
      </v-row>
      <v-container id="name-check-tabs-container" class="ma-0 pt-10 pl-10 pr-10">
        <v-tabs no-gutters
                id="name-check-tabs"
                active-class="active-tab"
                centered
                grow
                height="7rem"
                hide-slider
                style="border-radius: 4px 4px 0 0;"
                v-model="checks">
          <v-tab href="#structure-check"
                 id="structure-tab"
                 class="upper-border pt-3"
                 :ripple="false">
            <v-row no-gutters justify="center">
              <v-col>
                <v-icon size="2rem" :color="structureTabIcon.color" class="pa-1 check-tab-icon">
                  {{ structureTabIcon.icon }}
                </v-icon>
                <p class="check-tab-title pt-1">Name Structure Check</p>
                <p class="check-tab-sub-title pt-2 pb-7" v-html="structureSubTitle">{{ structureSubTitle }}</p>
              </v-col>
            </v-row>
          </v-tab>
          <v-tab href="#conflicts-check"
                 id="conflicts-tab"
                 class="upper-border pt-3"
                 :ripple="false">
            <v-row no-gutters justify="center">
              <v-col>
                <v-icon size="2rem"  :color="conflictTabIcon.color" class="pa-1 check-tab-icon">
                  {{ conflictTabIcon.icon }}
                </v-icon>
                <p class="check-tab-title pt-1">Similar Name Check</p>
                <p class="check-tab-sub-title pt-2 pb-7" v-html="conflictSubTitle">{{ conflictSubTitle }}</p>
              </v-col>
            </v-row>
          </v-tab>
        </v-tabs>
        <v-tabs-items class="rounded-b tab-items pa-3 pb-0" v-model="checks">
          <v-tab-item value="structure-check">
            <v-row no-gutters class="pa-10 pl-12 pb-7 name-check-info-text">
              <v-col cols="auto">
                <v-icon>mdi-information-outline</v-icon>
              </v-col>
              <v-col class="pl-2">
                <span>{{ tabInfoText }}</span>
              </v-col>
            </v-row>
            <NameCheckConflicts :items="structureItems" />
          </v-tab-item>
          <v-tab-item value="conflicts-check">
            <v-row no-gutters class="pa-10 pl-12 pb-7 name-check-info-text">
              <v-col cols="auto">
                <v-icon>mdi-information-outline</v-icon>
              </v-col>
              <v-col class="pl-2">
                <span>{{ tabInfoText }}</span>
              </v-col>
            </v-row>
            <NameCheckConflicts :items="conflictItems" />
          </v-tab-item>
        </v-tabs-items>
      </v-container>
      <v-row no-gutters justify="end" class="pa-10 pt-7">
        <v-btn id="name-check-submit-btn"
              @click="submit()">
          Submit this Name for Review
          <v-icon>mdi-chevron-right</v-icon>
        </v-btn>
      </v-row>
    </v-container>
  </v-container>
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

  private checks: string = null
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
    if (this.numRestConflicts > 0) return { color: 'red darken-2', icon: 'mdi-alert-octagon' }
    if (this.numCondConflicts > 0) return { color: 'orange darken-2', icon: 'mdi-alert' }
    return { color: 'green darken-2', icon: 'mdi-check-circle' }
  }
  get conflictTabIcon (): { color: string, icon: string } {
    if (this.numExactConflicts > 0) return { color: 'red darken-2', icon: 'mdi-alert-octagon' }
    else if (this.numSimConflicts > 0) return { color: 'orange darken-2', icon: 'mdi-alert' }
    return { color: 'green darken-2', icon: 'mdi-check-circle' }
  }
  get structureSubTitle (): string {
    if (this.numCondConflicts + this.numRestConflicts > 0) {
      return `<b>${this.numCondConflicts + this.numRestConflicts}</b> possible issues found`
    }
    return 'no issues found'
  }
  get conflictSubTitle (): string {
    let message = ''
    if (this.numExactConflicts > 0) message += `<b>${this.numExactConflicts}</b> exact matches`
    if (this.numSimConflicts > 0 && message) message += `, <b>${this.numSimConflicts}</b> similar names`
    else if (this.numSimConflicts > 0) message += `<b>${this.numSimConflicts}</b> similar names`
    // 0 conflicts
    else message += 'no similar names'
    return `${message} found`
  }
  get conflictItems (): Array<{
    count: number,
    expandedInfo: Array<string>,
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
        iconColor: 'red darken-2',
        problem: 'There is an existing BC Corporations using the <b>exact name:</b>',
        words: null,
        expandedExtraInfo: 'View tips on how to create a unique name',
        expandedInfo1: 'Exact name in use. A name will not be approved if there is an existing business using the exact same name. Consider revising your name unless you can obtain consent to use this name.',
        expandedInfo2: null,
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
        expandedExtraInfo: 'View tips on how to create a unique name',
        expandedInfo1: 'If you see a name similar to your choice, check the business category. If the existing name is in the same business category consider revising your name unless you can obtain consent to use your name.',
        expandedInfo2: 'If the name is in a different business category your name may be approved. However, if you want a unique name, consider revising your name. *The above list of names is not exhaustive and does not guarantee your name will be approved for use.',
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
        iconColor: 'green darken-2',
        problem: 'No similar names found',
        words: null,
        expandedExtraInfo: null,
        expandedInfo1: null,
        expandedInfo2: null,
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
    expandedInfo: string,
    expandedList: Array<{ name: string, type: string }>,
    icon: string,
    iconColor: string,
    problem: string,
    words: Array<string>,
  }> {
    let items = []
    if (this.numRestConflicts > 0) {
      items.push({
        icon: 'mdi-alert-octagon',
        iconColor: 'red darken-2',
        problem: 'Your name appears to be using the following words to avoid:',
        words: this.restrictedConflicts,
        expandedExtraInfo: null,
        expandedInfo1: 'Names involving a vulgar expression, obscene words or connotation, racial, physical or sexual slur will be rejected.',
        expandedInfo2: null,
        expandedList: null,
        expandLabel: {
          closed: 'read more',
          open: 'read less'
        },
        count: null
      })
    }
    if (this.numCondConflicts > 0) {
      items.push({
        icon: 'mdi-alert',
        iconColor: 'orange darken-2',
        problem: 'You may need consent to use the following words:',
        words: this.conditionalConflicts,
        expandedExtraInfo: null,
        expandedInfo1: 'Names that include well-known Names (trade name and Trademarks), names of public figures, or reference to the government, BC, or the Crown or Royal Family, will not be allowed without the advance written consent of the holder.',
        expandedInfo2: null,
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
        iconColor: 'green darken-2',
        problem: 'No issues have been found with your name structure',
        words: null,
        expandedInfo: null,
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
  get tabInfoText () {
    if (this.checks === 'structure-check') {
      return 'While not required, we recommend reviewing and resolving any issues ' +
            'for the best chance of your name being approved. You can edit your name ' +
            'above and recheck it. Read helpful tips below on how to resolve each ' +
            'issue.'
    } else {
      return 'Your name will not be approved if an exact match or a very similar name is already in use, unless you can obtain consent. For the best chance of having your name approved, review the existing names below to insure that your name is unique.'
    }
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

<style lang="scss" scoped>
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
  background-color: $gray2;
  max-width: none;
}
#name-check-main-container {
  max-width: none;
}
#name-check-tabs-container {
  background-color: $gray2;
  max-width: none;
}
#name-check-tabs {
  background-color: $gray2 !important;
}
#name-check-submit-btn {
  font-size: 1rem !important;
  font-weight: bold;
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
#structure-tab, #conflicts-tab {
  background-color: $app-blue;
  width: 10rem;
}
.check-tab-icon {
  background-color: white;
  border-radius: 50%;
  height: 2.8rem;
  width: 2.8rem;
}
.check-tab-title {
  color: white;
  font-size: 1.125rem !important;
  font-weight: bold;
  letter-spacing: normal !important;
  margin: 0 !important;
}
.active-tab .check-tab-title {
  color: $gray9;
}
.check-tab-sub-title {
  color: white;
  font-size: 0.875rem !important;
  font-weight: normal;
  letter-spacing: normal !important;
  margin: 0 !important;
  text-decoration: underline;
}
.active-tab .check-tab-sub-title {
  color: $gray7;
  text-decoration: none;
}
.name-check-info-text {
  color: $gray7;
  font-size: 0.875rem;
  line-height: 1.375rem;
  border-bottom: thin solid rgba(0, 0, 0, 0.12);
}
.upper-border {
  border-top-left-radius: 4px;
  border-top-right-radius: 4px;
  margin: 0 2.5px;
}
::v-deep .v-tab:before {
  border-top-left-radius: 4px;
  border-top-right-radius: 4px;
}
::v-deep .theme--light.v-tabs > .v-tabs-bar{
  background-color: transparent !important;
  transition: none !important;
}

::v-deep .theme--light.v-tabs > .v-tabs-bar .v-tab:not(.v-tab--active) > .v-icon {
  color: white;
  transition: none !important;
}
</style>
