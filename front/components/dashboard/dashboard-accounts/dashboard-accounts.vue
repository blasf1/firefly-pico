<template>
  <van-cell-group inset style="overflow: auto">
    <div class="flex-center-vertical justify-between">
      <!-- <div class="van-cell-group-title">Total balance:</div> -->
      <!-- <div class="flex-1" /> -->
      
      <van-button v-if="hasMultipleCurrencies" @click="onToggleTotalCurrency" size="small" class="mr-10">
        <template #icon>
          <app-icon :icon="TablerIconConstants.transaction" :size="20" />
          {{ dataStore.dashboardCurrency }}
        </template>
      </van-button>

      <div class="flex-center text-size-13 m-10 flex-wrap justify-center flex-grow-1">
        <div class="flex-center text-size-13 me-1">
          <icon-cash class="text-muted" :size="50" :stroke="2" />
          <span class="font-700 text-size-26">Total: </span>
        </div>

        <span v-for="(totalValue, totalCurrency) in dataStore.dashboardAccountsTotalByCurrency" class="font-700 text-size-26 ms-1 mx-1 app-select-option-tag">
          {{ getFormattedValue(totalValue) }} {{ totalCurrency }}
        </span>
      </div>

      <van-button @click="onToggleShowDashboardAccountValues" size="small" class="mr-10">
        <template #icon>
          <app-icon :icon="profileStore.dashboard.showAccountAmounts ? TablerIconConstants.eyeHidden : TablerIconConstants.eyeVisible" :size="20" />
        </template>
      </van-button>
    </div>
    
    <van-grid :column-num="4">
      <van-grid-item v-for="account in visibleDashboardAccounts" :key="account.id" @click="onGoToTransactions(account)" class="left-align-grid-item">
        <template #text>
          <div class="display-flex align-items-left">
            <app-icon :icon="Account.getIcon(account) ?? TablerIconConstants.account" :size="50" class="me-2" />
            <div>
              <div class="font-400 text-size-16 text-muted">{{ Account.getDisplayName(account) }}</div>
              <div :class="getAmountClass(extractNumericValue(getAccountAmount(account)))" class="font-700 text-size-16">{{ getAccountAmount(account) }}</div>
            </div>
          </div>
        </template>
      </van-grid-item>
    </van-grid>

    <div v-if="hasHiddenAccounts" class="flex-center">
      <div @click="toggleHiddenAccounts" class="p-5 m-5 button-link">
        {{ showHiddenAccounts ? 'View less...' : 'View more...' }}
      </div>
    </div>

    <div v-if="hasMultipleCurrencies" class="flex-center text-size-13 mb-3 gap-1">
      <span class="font-700">~{{ accountTotal }} {{ dataStore.dashboardCurrency }}</span>
    </div>
  </van-cell-group>
</template>

<script setup>
import TablerIconConstants from '~/constants/TablerIconConstants.js'
import Account from '~/models/Account.js'
import RouteConstants from '~/constants/RouteConstants.js'
import { IconCash } from '@tabler/icons-vue'
import { getFormattedValue } from '~/utils/MathUtils.js'

const profileStore = useProfileStore()
const dataStore = useDataStore()

const showHiddenAccounts = ref(false)
const toggleHiddenAccounts = () => {
  showHiddenAccounts.value = !showHiddenAccounts.value
}

const visibleDashboardAccounts = computed(() => {
  const sortedAccounts = dataStore.dashboardAccounts.sort((a, b) => Account.getIsVisibleOnDashboard(b) - Account.getIsVisibleOnDashboard(a))
  return showHiddenAccounts.value ? sortedAccounts : sortedAccounts.filter((account) => Account.getIsVisibleOnDashboard(account))
})

const hasHiddenAccounts = computed(() => dataStore.dashboardAccounts.some((account) => !Account.getIsVisibleOnDashboard(account)))

const accountTotal = computed(() => {
  return getFormattedValue(dataStore.dashboardAccountsEstimatedTotal)
})

const onToggleShowDashboardAccountValues = async () => {
  profileStore.dashboard.showAccountAmounts = !profileStore.dashboard.showAccountAmounts
}

const extractNumericValue = (value) => {
  return parseFloat(value.replace(/[^0-9.-]+/g, ''))
}

const getAccountAmount = (account) => {
  return `${getFormattedValue(Account.getBalance(account))} ${Account.getCurrency(account)}`
}

const getAmountClass = (amount) => {
      if (amount > 0) {
        return 'amount-positive';
      } else if (amount < 0) {
        return 'amount-negative';
      } else {
        return 'amount-zero';
      }
    }

const hasMultipleCurrencies = computed(() => dataStore.dashboardAccountsCurrencyList.length > 1)

const onToggleTotalCurrency = () => {
  if (dataStore.dashboardAccountsCurrencyList.length === 0) {
    return
  }
  let index = dataStore.dashboardAccountsCurrencyList.indexOf(dataStore.dashboardCurrency)
  let newIndex = (index + 1) % dataStore.dashboardAccountsCurrencyList.length
  dataStore.dashboardCurrency = dataStore.dashboardAccountsCurrencyList[newIndex]
}

const onGoToTransactions = async (account) => {
  if (!account) {
    return
  }
  await navigateTo(`${RouteConstants.ROUTE_TRANSACTION_LIST}?account_id=${account.id}`)
}
</script>
