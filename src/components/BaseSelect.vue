<template>
  <v-container fluid>
    <v-select
      :items="options"
      item-text="title"
      item-value="id"
      :label="title"
      @update:modelValue="selectItem"
      v-model="selectedItem"
      outlined
    ></v-select>
  </v-container>
</template>

<script>
export default {
  props: { options: Array, modelValue: [String, Number] },
  emits: ["update:modelValue"],
  data: () => ({
    selectedItem: null,
    title: "Биржи",
  }),

  methods: {
    selectItem(id) {
      this.value = this.modelValue;
      this.$emit("update:modelValue", id);
    },
  },
};
</script>
