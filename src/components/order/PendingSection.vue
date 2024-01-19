<template>
  <v-card>
    <v-table>
      <thead>
        <tr>
          <th>Order No.</th>
          <th>Order Type</th>
          <th>Order Items</th>
          <th>Status</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="row in pendingOrder" :key="row.orderNo">
          <td>{{ row.orderNo }}</td>
          <td>{{ row.orderType }}</td>
          <td>{{ row.orderItems }}</td>
          <td>{{ row.orderStatus }}</td>
        </tr>
      </tbody>
    </v-table>
  </v-card>
</template>

<script>
export default {
  props: ["pendingOrder"],
};
</script>
