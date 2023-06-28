<template>
    <button class="inventory-button" @click="toggleInventory">
      Data Inventory
    </button>
</template>

<script>
  export default {
    name: 'DataInventoryButton',
    methods: {
      toggleInventory() {
        this.$emit('toggle');
      },
    },
  };
</script>
  
<style scoped>
  .inventory-button {
    margin-right: 10px;
  }
</style>  