<template>
  <v-container class="mt-4 px-xl-8">
    <h1 class="text-center mb-4">McDonald Cooking Bots</h1>
    <v-row>
      <v-col class="mb-sm-3" cols="12" md="5" lg="4" xl="3">
        <v-row>
          <v-col cols="12">
            <v-card class="px-5 py-3">
              <v-card-title class="text-center"> Customers Menu </v-card-title>
              <v-divider></v-divider>
              <v-card-text>
                <v-row>
                  <v-col col="12">
                    <v-btn color="primary" block @click="openAddDialog('Normal')">
                      New Normal Order
                    </v-btn>
                  </v-col>

                  <v-col col="12">
                    <v-btn color="primary" block @click="openAddDialog('VIP')">
                      New VIP Order
                    </v-btn>
                  </v-col>
                </v-row>
              </v-card-text>
            </v-card>
          </v-col>

          <v-col cols="12">
            <v-card class="px-5 py-3">
              <v-card-title class="text-center"> Manager Menu </v-card-title>
              <v-divider></v-divider>
              <v-card-text>
                <v-row>
                  <v-col class="text-center">
                    <v-btn color="error" @click="removeBot"> - Bot</v-btn>
                  </v-col>
                  <v-col class="mt-2 text-center">
                    <span> Bot Count: {{ botCount }}</span>
                  </v-col>
                  <v-col class="text-center">
                    <v-btn color="info" @click="addBot"> + Bot</v-btn>
                  </v-col>
                </v-row>
              </v-card-text>
            </v-card>
          </v-col>
        </v-row>
      </v-col>
      <v-col class="mb-sm-3" cols="12" md="7" lg="8" xl="9">
        <v-card class="px-5 py-4" height="600px">
          <v-tabs v-model="tab" align-tabs="center">
            <v-tab value="pendingTab">Pending</v-tab>
            <v-tab value="completeTab">Complete</v-tab>
          </v-tabs>
          <v-window v-model="tab">
            <v-window-item value="pendingTab">
              <PendingSection :pendingOrder="pendingOrder" />
            </v-window-item>
            <v-window-item value="completeTab">
              <CompleteSection :completeOrder="completeOrder" />
            </v-window-item>
          </v-window>
        </v-card>
      </v-col>
    </v-row>

    <v-dialog v-model="showAddDialog" width="400px">
      <AddOrder :addOrder="addOrder" :selectType="selectType" />
    </v-dialog>
  </v-container>
</template>

<script>
import AddOrder from './components/order/AddOrder.vue'
import PendingSection from './components/order/PendingSection.vue'
import CompleteSection from './components/order/CompleteSection.vue'

import OrderStatus from './enum/OrderStatus.enum'
import OrderType from './enum/OrderType.enum'

export default {
  components: {
    AddOrder,
    PendingSection,
    CompleteSection
  },
  data() {
    return {
      tab: 'pendingTab',
      pendingOrder: [],
      completeOrder: [],
      processingBot: [],
      botCount: 0,
      maxBot: 5,
      showAddDialog: false,
      processTime: 10000,
      selectType: ''
    }
  },
  computed: {
    getPendingOrder() {
      return this.pendingOrder.filter((ele) => ele.orderStatus === OrderStatus.pending).length
    },
    checkAvailableBot() {
      return this.botCount > 0 && this.processingBot.length < this.botCount
    },
    checkMaxBot() {
      return this.botCount === this.maxBot
    },
    checkMinBot() {
      return this.botCount === 0
    }
  },
  methods: {
    openAddDialog(type) {
      this.selectType = type
      this.showAddDialog = true
    },
    addOrder(formData) {
      this.pendingOrder.push(formData)
      this.showAddDialog = false
      this.sortPendingOrder()
      if (this.checkAvailableBot) {
        this.botProcess()
      }
    },
    addBot() {
      if (this.checkMaxBot) return
      this.botCount++

      if (this.checkAvailableBot && this.getPendingOrder > 0) {
        this.botProcess()
      }
    },
    removeBot() {
      if (this.checkMinBot) return

      if (this.processingBot.length === this.botCount) {
        this.botRemoveProcess()
      }

      this.botCount--
    },
    // bot main process function
    botProcess() {
      // check if have pending vip
      const findPendingVIP = this.pendingOrder.find(
        (ele) => ele.orderType === OrderType.vip && ele.orderStatus === OrderStatus.pending
      )

      const toProcessType = findPendingVIP ? OrderType.vip : OrderType.normal

      const thisOrder = this.pendingOrder.find(
        (ele) => ele.orderStatus === OrderStatus.pending && ele.orderType === toProcessType
      )

      thisOrder.orderStatus = OrderStatus.processing

      // Set time out to move order to complete
      thisOrder.timer = setTimeout(() => {
        this.botDone(thisOrder.orderNo)
      }, this.processTime)

      // Add record to processing bot for checking
      this.processingBot.push(thisOrder)

      // Sort pending order list
      this.sortPendingOrder()
    },
    // Complete bot process
    botDone(orderNo) {
      // Get current order location
      const findIndex = this.pendingOrder.findIndex((ele) => ele.orderNo === orderNo)
      // Get and remove current order from pending list
      const order = this.pendingOrder[findIndex]
      this.pendingOrder.splice(findIndex, 1)
      // Update status to complete and move to complete order list
      order.status = OrderStatus.completed
      this.completeOrder.push(order)

      // Remove order remove from proccessing bot list
      this.processingBot = this.processingBot.filter((ele) => ele.orderNo !== orderNo)

      // Check pending list still have not process order
      if (this.pendingOrder.filter((ele) => ele.orderStatus === OrderStatus.pending).length > 0) {
        this.botProcess()
      }
    },
    // Remove bot process
    botRemoveProcess() {
      // Get current processing last record
      const lastProcessOrder = this.processingBot.slice(-1)[0]

      // Remove processing timer and remove last processing record
      clearTimeout(lastProcessOrder.timer)
      this.processingBot.pop()

      // Find current record in pending list and update back to pending status
      this.pendingOrder.find((ele) => {
        if (
          ele.orderStatus === OrderStatus.processing &&
          ele.orderNo === lastProcessOrder.orderNo
        ) {
          ele.orderStatus = OrderStatus.pending
        }
      })
    },
    sortPendingOrder() {
      // Sort pending list by processing status, order type is vip and order no in asc order
      this.pendingOrder.sort(
        (a, b) =>
          (b.orderStatus === OrderStatus.processing) - (a.orderStatus === OrderStatus.processing) ||
          (b.orderType === OrderType.vip) - (a.orderType === OrderType.vip) ||
          parseInt(a.orderNo) - parseInt(b.orderNo)
      )
    }
  }
}
</script>

<style scoped>
.v-card {
  overflow: auto;
}
</style>
