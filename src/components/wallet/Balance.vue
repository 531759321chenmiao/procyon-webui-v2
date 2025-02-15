<template>
  <div v-if='!hidden'>
    <h2>{{ $t('MSG_ACCOUNT_BALANCE') }}</h2>
    <div class='earnings-summary'>
      <div class='earnings-figure'>
        <span class='amount'>{{ (_totalEarningUSD - totalWithdrawedUSD + _totalPaymentBalanceUSD + commissionUSD).toFixed(4) }}</span>
        <span class='unit'>{{ PriceCoinName }}</span>
        <div class='hr' />
        <h4 class='description'>
          {{ $t('MSG_CURRENT_BALANCE') }} ({{ PriceCoinName }})
        </h4>
      </div>
      <div class='earnings-figure'>
        <span class='amount'>{{ (totalEarningJPY - totalWithdrawedJPY + totalPaymentBalanceJPY + commissionJPY).toFixed(4) }}</span>
        <span class='unit'>JPY</span>
        <div class='hr' />
        <h4 class='description'>
          {{ $t('MSG_CURRENT_BALANCE') }} (JPY)
        </h4>
      </div>
    </div>
  </div>
</template>

<script setup lang='ts'>
import {
  Currency,
  totalEarningUSD,
  useCurrencyStore,
  NotificationType,
  useCoinStore,
  useTransactionStore,
  totalWithdrawedEarningUSD,
  totalEarningCurrency,
  totalWithdrawedEarningCurrency,
  PriceCoinName,
  useGoodStore,
  useBenefitStore,
  useBillingStore,
  totalPaymentBalanceUSD,
  totalPaymentBalanceCurrency,
  useOrderStore,
  CoinType
} from 'npool-cli-v2'
import { onMounted, ref, defineProps, toRef, computed } from 'vue'
import { useI18n } from 'vue-i18n'

interface Props {
  hidden?: boolean
}

const props = defineProps<Props>()
const hidden = toRef(props, 'hidden')

// eslint-disable-next-line @typescript-eslint/unbound-method
const { t } = useI18n({ useScope: 'global' })

const _totalEarningUSD = ref(0)
const totalWithdrawedUSD = ref(0)
const _totalPaymentBalanceUSD = ref(0)

const totalEarningJPY = ref(0)
const totalWithdrawedJPY = ref(0)
const totalPaymentBalanceJPY = ref(0)

const currency = useCurrencyStore()
const coin = useCoinStore()
const transaction = useTransactionStore()
const good = useGoodStore()
const benefit = useBenefitStore()
const billing = useBillingStore()
const order = useOrderStore()

const commissionUSD = computed(() => benefit.Commission.Balance)
const commissionJPY = ref(0)

const getPaymentBalances = () => {
  billing.getPaymentBalances({
    Message: {
      Error: {
        Title: t('MSG_GET_PAYMENT_BALANCES'),
        Message: t('MSG_GET_PAYMENT_BALANCES_FAIL'),
        Popup: true,
        Type: NotificationType.Error
      }
    }
  }, () => {
    totalPaymentBalanceUSD((usdAmount: number) => {
      _totalPaymentBalanceUSD.value = usdAmount
    })
    totalPaymentBalanceCurrency(Currency.JPY, (amount: number) => {
      totalPaymentBalanceJPY.value = amount
    })
  })
}

const getEarning = () => {
  benefit.getBenefits({
    Message: {
      Error: {
        Title: t('MSG_GET_BENEFITS'),
        Message: t('MSG_GET_BENEFITS_FAIL'),
        Popup: true,
        Type: NotificationType.Error
      }
    }
  }, () => {
    totalEarningUSD((usdAmount: number) => {
      _totalEarningUSD.value = usdAmount
    })
    totalEarningCurrency(Currency.JPY, (usdAmount: number) => {
      totalEarningJPY.value = usdAmount
    })
  })
}

const getWithdrawed = () => {
  transaction.getTransactions({
    Message: {
      Error: {
        Title: t('MSG_GET_TRANSACTIONS'),
        Message: t('MSG_GET_TRANSACTIONS_FAIL'),
        Popup: true,
        Type: NotificationType.Error
      }
    }
  }, () => {
    transaction.getWithdraws({
      Message: {
        Error: {
          Title: t('MSG_GET_WITHDRAWS'),
          Message: t('MSG_GET_WITHDRAWS_FAIL'),
          Popup: true,
          Type: NotificationType.Error
        }
      }
    }, () => {
      totalWithdrawedEarningUSD((usdAmount: number) => {
        totalWithdrawedUSD.value = usdAmount
      })
      totalWithdrawedEarningCurrency(Currency.JPY, (usdAmount: number) => {
        totalWithdrawedJPY.value = usdAmount
      })
    })
  })
}

const getGoods = () => {
  good.getGoods({
    Message: {
      Error: {
        Title: t('MSG_GET_GOODS'),
        Message: t('MSG_GET_GOODS_FAIL'),
        Popup: true,
        Type: NotificationType.Error
      }
    }
  }, () => {
    getEarning()
    getWithdrawed()
  })
}

const getOrders = () => {
  order.getOrders({
    Message: {
      Error: {
        Title: t('MSG_GET_ORDERS'),
        Message: t('MSG_GET_ORDERS_FAIL'),
        Popup: true,
        Type: NotificationType.Error
      }
    }
  }, () => {
    getPaymentBalances()
  })
}

const getCurrencies = () => {
  currency.getAllCoinCurrencies({
    Currencies: [Currency.USD],
    Message: {
      Error: {
        Title: t('MSG_GET_CURRENCIES'),
        Message: t('MSG_GET_CURRENCIES_FAIL'),
        Popup: true,
        Type: NotificationType.Error
      }
    }
  }, () => {
    getGoods()
    getOrders()
  })
}

const getCoins = () => {
  coin.getCoins({
    Message: {
      Error: {
        Title: t('MSG_GET_COINS'),
        Message: t('MSG_GET_COINS_FAIL'),
        Popup: true,
        Type: NotificationType.Error
      }
    }
  }, () => {
    getCurrencies()
  })
}

onMounted(() => {
  benefit.getCommission({
    Message: {
      Error: {
        Title: t('MSG_GET_COMMISSION_FAIL'),
        Popup: true,
        Type: NotificationType.Error
      }
    }
  }, () => {
    currency.getCoinNameCurrency(CoinType.USDTERC20, Currency.JPY, (jpyCurrency: number) => {
      commissionJPY.value = jpyCurrency * benefit.Commission.Balance
    })
  })

  if (benefit.Benefits.length === 0) {
    getCoins()
    return
  }

  totalEarningUSD((usdAmount: number) => {
    _totalEarningUSD.value = usdAmount
  })
  totalEarningCurrency(Currency.JPY, (amount: number) => {
    totalEarningJPY.value = amount
  })
  totalWithdrawedEarningUSD((usdAmount: number) => {
    totalWithdrawedUSD.value = usdAmount
  })
  totalWithdrawedEarningCurrency(Currency.JPY, (amount: number) => {
    totalWithdrawedJPY.value = amount
  })
  totalPaymentBalanceUSD((usdAmount: number) => {
    _totalPaymentBalanceUSD.value = usdAmount
  })
  totalPaymentBalanceCurrency(Currency.JPY, (amount: number) => {
    totalPaymentBalanceJPY.value = amount
  })
})

</script>
