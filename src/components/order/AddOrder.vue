<template>
  <v-card class="py-3">
    <v-card-title> New Order</v-card-title>
    <v-divider></v-divider>
    <v-form class="px-3" ref="form">
      <v-text-field label="Order No" v-model="formData.orderNo" disabled></v-text-field>
      <v-select
        label="Order Type"
        :items="orderTypeList"
        v-model="formData.orderType"
        :rules="rules.type"
        disabled
      ></v-select>
      <v-select
        label="Order Item"
        :items="orderItemList"
        v-model="formData.orderItems"
        :rules="rules.item"
      ></v-select>
    </v-form>
    <v-card-actions>
      <v-btn color="primary" block @click="formAddOrder">Add Order</v-btn>
    </v-card-actions>
  </v-card>
</template>

<script>
import OrderType from '../../enum/OrderType.enum'
import OrderStatus from '../../enum/OrderStatus.enum'

export default {
  props: ['addOrder', 'selectType'],
  data() {
    return {
      orderTypeList: [OrderType.normal, OrderType.vip],
      orderItemList: ['Food1', 'Food2', 'Food3', 'Food4', 'Food5', 'Food6'],
      rules: {
        type: [(v) => !!v.trim() || 'Order type is required'],
        item: [(v) => !!v.trim() || 'Order item is required']
      },
      formData: {
        orderNo: null,
        orderType: '',
        orderItems: ''
      }
    }
  },
  methods: {
    // Generate order no. from current timestamp
    generateOrderNo() {
      const date = Date.now()
      return date.toString().slice(-8)
    },
    async formAddOrder() {
      const { valid } = await this.$refs.form.validate()
      if (!valid) return
      const addData = {
        ...this.formData,
        orderStatus: OrderStatus.pending
      }
      this.addOrder(addData)
    }
  },
  mounted() {
    this.formData.orderItems = ''
    this.formData.orderType = this.selectType
    this.formData.orderNo = this.generateOrderNo()
  }
}
</script>
