<template>
  <v-data-table :headers="headers"
                fixed
                hide-default-header hide-default-footer
                :items="items"
                item-key="problem"
                :expanded.sync="expanded"
                show-expand>
    <template v-slot:item="{ item, headers, expand, isExpanded }">
      <tr :colspan="headers.length" @click="expand(!isExpanded)">
        <v-row no-gutters class="py-7 px-4" :class="isExpanded ? '' : 'bottom-border'">
          <v-col cols="9">
            <v-row no-gutters>
              <v-col cols="auto">
                <v-icon :color="item.iconColor">{{ item.icon }}</v-icon>
              </v-col>
              <v-col cols="auto" class="table-text pl-3">
                <span v-html="item.problem"/>
                <b v-for="(word, index) in item.words" :key=index>
                  <span v-if="index === 0"> {{ word }}</span>
                  <span v-else>, {{ word }}</span>
                </b>
              </v-col>
            </v-row>
          </v-col>
          <v-col cols="3">
            <v-row no-gutters jusify="end">
              <v-col cols="12">
                <v-row no-gutters justify="end">
                  <v-col v-if="item.count" cols="auto">
                    <v-chip :color="item.iconColor" dark>{{ item.count }}</v-chip>
                  </v-col>
                  <v-col v-if="!isExpanded && item.expandLabel" :cols="item.count ? 4 : 8">
                    <v-row no-gutters :justify="item.count ? 'center' : 'end'">
                      <v-col :cols="item.count ? 4 : 6">
                        <span class="expand-label">{{ item.expandLabel.closed }}</span>
                      </v-col>
                      <v-col cols="2" class="pl-3 pr-5">
                        <v-icon class="expand-icon" color="blue">mdi-chevron-down</v-icon>
                      </v-col>
                    </v-row>
                  </v-col>
                  <v-col v-else-if="item.expandLabel" :cols="item.count ? 4 : 8">
                    <v-row no-gutters :justify="item.count ? 'center' : 'end'">
                      <v-col :cols="item.count ? 4 : 6">
                        <span class="expand-label">{{ item.expandLabel.open }}</span>
                      </v-col>
                      <v-col class="expand-icon pl-3 pr-5" cols="2">
                        <v-icon color="blue">mdi-chevron-up</v-icon>
                      </v-col>
                    </v-row>
                  </v-col>
                </v-row>
              </v-col>
            </v-row>
          </v-col>
        </v-row>
      </tr>
    </template>
    <template v-slot:expanded-item="{ item }">
      <v-row no-guttters class="bottom-border pt-1">
        <v-col class="pa-0">
          <v-row v-if="item.expandedList" no-gutters class="px-15">
            <QuickSearchNames :namesList="item.expandedList"/>
          </v-row>
          <v-row v-if="item.expandedInfo1" no-gutters class="px-15 pt-7 pb-7 name-check-info-text">
            <v-col cols="auto">
              <v-icon>mdi-information-outline</v-icon>
            </v-col>
            <v-col class="pl-2" style="max-width: 50rem">
              <v-row no-gutters>
                <span>{{ item.expandedInfo1 }}</span>
              </v-row>
            </v-col>
          </v-row>
        </v-col>
      </v-row>
    </template>
  </v-data-table>
</template>

<script lang="ts">
import { Component, Prop, Vue } from 'vue-property-decorator'
import QuickSearchNames from '@/components/new-request/quick-search-names.vue'

@Component({
  components: { QuickSearchNames }
})
export default class NameCheckConflicts extends Vue {
  config = {
    modules: {
      toolbar: false
    },
    placeholder: '',
    scrollingContainer: false
  }
  @Prop() items: Array<{
    count: number,
    expandedExtraInfo: string,
    expandedInfo1: string,
    expandedInfo2: string,
    expandedList: Array<{ name: string, type: string }>,
    expandLabel: {
      open: string,
      closed: string
    }
    icon: string,
    iconColor: string,
    problem: string,
    words: Array<string>
  }>
  private expanded = []
  private headers = [
    { text: 'Icon', value: 'icon' },
    { text: 'Problem', value: 'problem' },
    { text: 'Words', value: 'words' },
    { text: '', value: 'data-table-expand' }
  ]
}
</script>

<style lang="scss">
@import '@/assets/scss/theme.scss';
.bottom-border {
  border-bottom: thin solid rgba(0, 0, 0, 0.12) !important;
}
.expand-label {
  color: #2196F3;
}
.name-check-info-text {
  color: $gray7;
  font-size: 0.875rem;
  line-height: 1.375rem;
}
.table-text {
  color: $gray7;
  font-size: 1rem;
  line-height: 1.5rem;
}
</style>
