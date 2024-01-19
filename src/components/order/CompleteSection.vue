<template>
  <v-card>
    <v-table>
      <thead>
        <tr>
          <th>Order No.</th>
          <th>Order Type</th>
          <th>Order Items</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="row in completeOrder" :key="row.orderNo">
          <td>{{ row.orderNo }}</td>
          <td>{{ row.orderType }}</td>
          <td>{{ row.orderItems }}</td>
        </tr>
      </tbody>
    </v-table>
  </v-card>
</template>

<script>
export default {
  props: ["completeOrder"],
};
</script>
