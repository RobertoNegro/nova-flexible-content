<template>
    <div class="w-full" v-if="layouts">

        <div v-if="this.limitCounter != 0">
            <div v-if="layouts.length === 1">
                <button
                    dusk="toggle-layouts-dropdown-or-add-default"
                    type="button"
                    tabindex="0"
                    class="btn btn-default btn-primary inline-flex items-center relative float-left"
                    @click="toggleLayoutsDropdownOrAddDefault"
                >
                    <span>{{ field.button }}</span>
                </button>
            </div>
            <div v-if="layouts.length > 1">
                <div style="min-width: 100px;">
                    <div class="flexible-search-menu-multiselect">
                        <multiselect
                             v-model="selectedLayout" :options="levelLayouts"
                             :custom-label="renderLayoutName"
                             :placeholder="field.button"
                             :group-select="false"
                             @input="selectLayout"
                             group-values="fields"
                             group-label="label"
                             v-bind="attributes"
                             track-by="name"
                        ></multiselect>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>

    import Multiselect from 'vue-multiselect';
    import { eventBus } from '../eventbus';

    export default {
        components: {Multiselect},

        props: ['layouts', 'field', 'resourceName', 'resourceId', 'resource', 'errors', 'limitCounter', 'index'],

        data() {
            return {
                selectedLayout: null,
                isLayoutsDropdownOpen: false
            };
        },

        computed: {
            levelLayouts () {
              const res = [
                {
                  'label': '1° Livello',
                  'fields': []
                },
                {
                  'label': '2° Livello',
                  'fields': []
                }
              ];

              this.layouts.forEach((f) => {
                if(['x_column', 'x_banner'].includes(f.name)) {
                  res[0].fields.push(f);
                } else {
                  res[1].fields.push(f);
                }
              });

              return res;
            },

            attributes() {
                return {
                    selectLabel: this.field.menu.data.selectLabel || __('Press enter to select'),
                    label: this.field.menu.data.label || 'title',
                    openDirection: this.field.menu.data.openDirection || 'bottom',
                }
            }
        },

        methods: {
            selectLayout(value){
                this.addGroup(value);
            },
            renderLayoutName(layout){
               return layout.title;
            },
            /**
             * Display or hide the layouts choice dropdown if there are multiple layouts
             * or directly add the only available layout.
             */
            toggleLayoutsDropdownOrAddDefault(event) {
                if (this.layouts.length === 1) {
                    return this.addGroup(this.layouts[0]);
                }

                this.isLayoutsDropdownOpen = !this.isLayoutsDropdownOpen;
            },

            /**
             * Append the given layout to flexible content's list
             */
            addGroup(layout) {
                if (!layout) return;

                eventBus.$emit('add-group-'+this.field.attribute, layout, this.index);

                this.isLayoutsDropdownOpen = false;
                this.selectedLayout = null;
            },
        }
    }
</script>

<style>
.multiselect__content-wrapper {
  position: static;
}
</style>
