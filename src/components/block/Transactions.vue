<template>
  <div v-if="transactions.length > 0">
    <h2 class="text-2xl mb-5 md:mb-6 px-10 sm:hidden text-theme-text-primary">Transactions</h2>
    <section class="page-section py-8">
      <div class="hidden sm:block">
        <table-transactions :transactions="transactions"></table-transactions>
      </div>
      <div class="sm:hidden">
        <table-transactions-mobile :transactions="transactions"></table-transactions-mobile>
      </div>
    </section>
  </div>
</template>

<script type="text/ecmascript-6">
import TransactionService from '@/services/transaction'

export default {
  props: {
    block: {
      type: Object,
      required: true
    }
  },

  data: () => ({ transactions: [] }),

  watch: {
    block() {
      this.getTransactions()
    }
  },

  methods: {
    getTransactions () {
      if (!this.block.id) return

      TransactionService
        .findByBlock(this.block.id)
        .then(response => this.transactions = response)
    }
  }
}
</script>
