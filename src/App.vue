<template>
  <div id="app">
      <b-table :items="items" :fields="fields">
          <template #cell(key_type)="data">
              <b-form-select v-model="data.value" :options="['Type-1', 'Type-2', 'Type-3', 'Type-4']"
                  class="form-control"></b-form-select>
          </template>
          <template #cell(description)="data">
              <b-form-input type="text" :value="data.value"></b-form-input>
          </template>
          <template #cell(value)="data">
              <b-form-input type="text" :value="data.value"></b-form-input>
          </template>
      </b-table>
  </div>
</template>
<script>
export default {
description: "App",
components: {},
data() {
  return {
      fields: [
          { key: "key_type", label: "Key Type" },
          { key: "description", label: "Description" },
          { key: "value", label: "Value" }
      ],
      items: [
          { key_type: 'Type-1', description: 'Sample Key', value: 'Sample Value' },
          { key_type: 'Type-1', description: 'Sample Key', value: 'Sample Value' },
          { key_type: 'Type-1', description: 'Sample Key', value: 'Sample Value' },
          { key_type: 'Type-1', description: 'Sample Key', value: 'Sample Value' }

      ],
  };
},
};
</script>
<style>
#scrubbing-app {
text-align: center;
margin-top: 60px;
}

thead,
tbody,
tfoot,
tr,
td,
th {
text-align: center;
}
</style>