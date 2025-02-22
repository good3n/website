<template>
  <div>
    <form-wizard
      title="Banner Creator"
      subtitle="Create a Modrinth Resource Banner"
      shape="tab"
      color="#4299e1"
      error-color="#ec4e20"
      @on-loading="setLoading"
      @on-complete="handleComplete"
    >
      <tab-content :before-change="checkValidResource" title="Resource Details">
        <GeneratorPreCheck
          :loading="loading"
          :error-message="resource.error"
          loading-message="One sec...we're just checking that resource."
        >
          <ResourceGeneratorStepOne
            type="modrinth"
            @update="updateResourceDetails"
          />
        </GeneratorPreCheck>
      </tab-content>
      <tab-content title="Configure Banner">
        <b-card no-body>
          <b-tabs pills card vertical>
            <GeneratorPreview :banner-u-r-l="bannerURL" />
            <b-tab title="Background">
              <BannerSelectControlBox
                :default="template"
                :options="templateOptions"
                title="Background"
                hint="Choose the background for your banner."
                @update="(newTemplate) => (template = newTemplate)"
              />
            </b-tab>
            <b-tab title="Resource Logo">
              <ControlBox title="Resource Logo">
                <template #hint>
                  <p>
                    Configure how the resource logo will display in the
                    generated banner.
                  </p>
                </template>
                <template #controls>
                  <b-input-group prepend="Size" append="sq px">
                    <b-form-input v-model.number="logo.size" type="number" />
                  </b-input-group>
                  <b-input-group prepend="X Offset" append="px">
                    <b-form-input v-model.number="logo.x" type="number" />
                  </b-input-group>
                </template>
              </ControlBox>
            </b-tab>
            <b-tab title="Resource Name">
              <BannerTextFieldControlBox
                :target="resource_name"
                title="Resource Name"
                namespace="resource_name"
                @update="handleFieldUpdate"
              >
                <template #hint>
                  <p>
                    Configure how the resource name will display in the
                    generated banner.
                  </p>
                  <p>
                    <small
                    >* If your resource's name is too long for the image, set
                      a <strong>Text Override</strong>.</small
                    >
                  </p>
                </template>
                <template #ext_bot_controls>
                  <b-input-group prepend="Text Override">
                    <b-input
                      v-model="resource_name.display"
                      type="text"
                      placeholder="No Override Set"
                    />
                  </b-input-group>
                </template>
              </BannerTextFieldControlBox>
            </b-tab>
            <b-tab title="Author Name">
              <BannerTextFieldControlBox
                :target="author_name"
                title="Author Name"
                namespace="author_name"
                @update="handleFieldUpdate"
              >
                <template #hint>
                  <p>
                    Configure how the author name will display in the generated
                    banner.
                  </p>
                </template>
              </BannerTextFieldControlBox>
            </b-tab>
            <b-tab title="Updated Time">
              <BannerTextFieldControlBox
                :target="updated"
                title="Updated Time"
                namespace="updated"
                @update="handleFieldUpdate"
              >
                <template #hint>
                  <p>
                    Configure how the updated time will display in the generated
                    banner.
                  </p>
                </template>
              </BannerTextFieldControlBox>
            </b-tab>
            <b-tab title="Download Count">
              <BannerTextFieldControlBox
                :target="downloads"
                title="Download Count"
                namespace="downloads"
                @update="handleFieldUpdate"
              >
                <template #hint>
                  <p>
                    Configure how the download count will display in the
                    generated banner.
                  </p>
                </template>
              </BannerTextFieldControlBox>
            </b-tab>
          </b-tabs>
        </b-card>
      </tab-content>
    </form-wizard>

    <CopyURLModal :banner-u-r-l="savedBannerURL" />
  </div>
</template>

<script>
import { mapState } from 'vuex'
import UtilityMethods from '~/mixins/utility_methods'
import GeneratorParamMixin from '~/mixins/generator/generator_param_mixin'
import SaveBannerMixin from '~/mixins/generator/save_banner_mixin'
import LoadingMixin from '~/mixins/loading_mixin'
import GeneratorPreCheck from '~/components/generator/GeneratorPreCheck'
import GeneratorPreview from '~/components/generator/GeneratorPreview'
import ControlBox from '~/components/generator/control/ControlBox'
import BannerSelectControlBox from '~/components/generator/control/BannerSelectControlBox'
import BannerTextFieldControlBox from '~/components/generator/control/BannerTextFieldControlBox'
import ResourceGeneratorStepOne from '~/components/generator/type/resource/steps/ResourceGeneratorStepOne'
import CopyURLModal from '~/components/flow/CopyURLModal'

export default {
  name: 'ModrinthResourceGenerator',
  components: {
    GeneratorPreCheck,
    GeneratorPreview,
    ControlBox,
    BannerSelectControlBox,
    BannerTextFieldControlBox,
    ResourceGeneratorStepOne,
    CopyURLModal,
  },
  mixins: [UtilityMethods, GeneratorParamMixin, SaveBannerMixin, LoadingMixin],
  data() {
    return {
      resource: {
        name: undefined,
        error: '',
      },
      template: 'MOONLIGHT_PURPLE',
      logo: {
        size: 80,
        x: 12,
      },
      resource_name: {
        x: 104,
        y: 25,
        font_size: 18,
        bold: true,
        text_align: 'LEFT',
        font_face: 'SOURCE_SANS_PRO',
        display: '',
      },
      author_name: {
        x: 104,
        y: 42,
        font_size: 14,
        bold: false,
        text_align: 'LEFT',
        font_face: 'SOURCE_SANS_PRO',
      },
      updated: {
        x: 104,
        y: 62,
        font_size: 14,
        bold: false,
        text_align: 'LEFT',
        font_face: 'SOURCE_SANS_PRO',
      },
      downloads: {
        x: 104,
        y: 83,
        font_size: 14,
        bold: false,
        text_align: 'LEFT',
        font_face: 'SOURCE_SANS_PRO',
      },
    }
  },
  computed: {
    ...mapState({
      templates: (state) => state.svc.templates,
      defaults: (state) => state.svc.defaults.resource,
    }),
    templateOptions() {
      return this.makeSelectable(this.templates)
    },
    baseURL() {
      return this.generateBannerUrl('resource-modrinth', {
        id: this.resource.name,
      })
    },
  },
  methods: {
    handleFieldUpdate(payload) {
      this[payload.namespace][payload.key] = payload.value
    },
    cleanupModifiedParams(copy) {
      delete copy.resource

      if (!copy.resource_name.display) {
        copy.resource_name.display = ''
      }

      return copy
    },
    updateResourceDetails(payload) {
      this.resource.name = payload.subject
    },
    async checkValidResource() {
      this.resource.error = ''
      const { name } = this.resource

      if (!name) {
        this.resource.error = 'Please enter a Modrinth Resource ID.'
        return false
      }

      const valid = await this.$store.dispatch('checkValidModrinthResource', name)

      if (valid.state) {
        return true
      } else {
        if (valid.resp.status === 202) {
          this.resource.error = valid.resp.message
        } else {
          this.resource.error =
            "That doesn't seem to a valid Modrinth Resource ID. Please double check it."
        }
        return false
      }
    },
    async handleComplete() {
      this.loading = true
      await this.saveModrinthResourceBanner()
      this.loading = false
      this.$bvModal.show('copy-url-modal')
    },
  },
}
</script>

<style lang="scss" scoped>
.card-columns {
  column-count: 1;

  @media (min-width: 768px) {
    column-count: 2;
  }
}

.input-group {
  margin-bottom: 2px;
}

.result_box {
  margin-bottom: 10px;

  .col-12 {
    margin-bottom: 5px;
  }
}
</style>

