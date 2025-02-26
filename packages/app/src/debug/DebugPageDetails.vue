<template>
  <div
    class="flex flex-col w-full px-24px justify-center items-center align-middle"
    :class="{'flex-grow': ['PASSED', 'OVERLIMIT'].includes(status)}"
  >
    <DebugCancelledAlert
      v-if="status === 'CANCELLED'"
      :total-specs="specs.length"
      :total-skipped-specs="totalSkippedSpecs"
      :cancellation="cancellation"
    />
    <DebugPassed v-else-if="status === 'PASSED'" />
    <DebugErrored
      v-else-if="status === 'ERRORED'"
      :errors="errors"
      :total-specs="specs.length"
      :total-skipped-specs="totalSkippedSpecs"
    />
    <DebugNoTests v-else-if="status === 'NOTESTS'" />
    <DebugTimedout
      v-else-if="status === 'TIMEDOUT'"
      :total-specs="specs.length"
      :total-skipped-specs="totalSkippedSpecs"
      :ci="ci"
    />
    <DebugOverLimit
      v-else-if="isHidden"
      :over-limit-reasons="reasonsRunIsHidden"
      :over-limit-action-type="overLimitActionType"
      :over-limit-action-url="overLimitActionUrl"
      :run-age-days="runAgeDays"
    />
  </div>
</template>
<script lang="ts" setup>
import { computed } from 'vue'
import { gql } from '@urql/vue'
import type { CloudRunStatus, OverLimitActionTypeEnum, DebugSpecListSpecFragment, DebugPageDetails_CloudCiBuildInfoFragment } from '../generated/graphql'
import DebugCancelledAlert from './DebugCancelledAlert.vue'
import DebugPassed from './DebugPassed.vue'
import DebugErrored from './DebugErrored.vue'
import DebugNoTests from './DebugNoTests.vue'
import DebugTimedout from './DebugTimedout.vue'
import DebugOverLimit, { CloudRunHidingReason } from './DebugOverLimit.vue'

gql`
fragment DebugPageDetails_cloudCiBuildInfo on CloudCiBuildInfo {
  id
  ciBuildNumberFormatted
  formattedProvider
  url
}
`

const props = defineProps<{
  status: CloudRunStatus
  cancellation: {
    cancelledAt?: string | null
    cancelledBy: {
      fullName?: string | null
      email?: string | null
    } | null
  }
  isHidden: boolean
  reasonsRunIsHidden: (CloudRunHidingReason | null)[]
  overLimitActionType: OverLimitActionTypeEnum
  overLimitActionUrl: string
  specs: readonly DebugSpecListSpecFragment[]
  ci?: DebugPageDetails_CloudCiBuildInfoFragment
  errors: readonly string[]
  runAgeDays: number
}>()

const totalSkippedSpecs = computed(() => {
  return props.specs.filter((spec) => spec.status === 'UNCLAIMED' || spec.status === 'RUNNING').length
})

</script>
