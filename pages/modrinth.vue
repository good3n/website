<template>
  <b-container>
    <b-form-group id="module-select">
      <b-input-group prepend="Module">
        <b-form-select v-model="module.selected" :options="module.options" />
      </b-input-group>
    </b-form-group>

    <div class="generator">
      <div v-if="module.selected == 0">
        <ModrinthResourceGenerator />
      </div>

      <div v-else>
        <ModrinthAuthorGenerator />
      </div>
    </div>
  </b-container>
</template>

<script>
import ModrinthResourceGenerator from "~/components/generator/type/modrinth/ModrinthResourceGenerator";
import ModrinthAuthorGenerator from "~/components/generator/type/modrinth/ModrinthAuthorGenerator";

export default {
  components: {ModrinthResourceGenerator, ModrinthAuthorGenerator },
  data() {
    return {
      module: {
        selected: 0,
        options: [
          { value: 0, text: 'Resource Banner' },
          { value: 1, text: 'Author Banner' },
        ],
      },
    }
  },
  head() {
    return this.$seo({
      name: 'MCBanners',
      title: 'Modrinth Banners',
      templateTitle: '%name% - %title%',
    })
  },
}
</script>

<style lang="scss" scoped>
.generator {
  margin-bottom: 45px;
}
</style>
