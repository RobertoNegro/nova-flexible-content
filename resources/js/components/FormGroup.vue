<template>
      <div :class="{'mx-1 mb-1 mt-1': !isColumn}"
           class="overflow-x-auto overflow-y-hidden relative flex"
           :id="group.key">
          <div style="top: 1px; left: 1px" class="absolute z-10 h-auto w-8">
              <div v-if="collapsed">
                <button
                    dusk="expand-group"
                    type="button"
                    class="group-control rounded-tl-lg rounded-bl-lg bg-white btn w-8 h-8 block border-40"
                    :class="collapsed ? '' : 'border-r'"
                    :title="__('Expand')"
                    @click.prevent="expand">
                    <icon class="align-top" type="plus-square" width="16" height="16" view-box="0 0 24 24"/>
                </button>
              </div>
              <div v-if="!collapsed">
                  <button
                      dusk="collapse-group"
                      type="button"
                      class="rounded-tl-lg group-control bg-white btn border-r border-40 w-8 h-8 block"
                      :title="__('Collapse')"
                      @click.prevent="collapse">
                      <icon class="align-top" type="minus-square" width="16" height="16" view-box="0 0 24 24"/>
                  </button>
                  <div v-if="!readonly">
                      <button
                          dusk="move-up-group"
                          type="button"
                          class="group-control bg-white btn border-t border-r border-40 w-8 h-8 block"
                          :title="__('Move up')"
                          @click.prevent="moveUp">
                          <icon type="arrow-up" view-box="0 0 8 4.8" width="10" height="10"/>
                      </button>
                      <button
                          dusk="move-down-group"
                          type="button"
                          class="group-control bg-white btn border-t border-r border-40 w-8 h-8 block"
                          :title="__('Move down')"
                          @click.prevent="moveDown">
                          <icon type="arrow-down" view-box="0 0 8 4.8" width="10" height="10"/>
                      </button>
                      <button
                          dusk="delete-group"
                          type="button"
                          class="rounded-br-lg group-control bg-white btn border-t border-r border-b border-40 w-8 h-8 block"
                          :title="__('Delete')"
                          @click.prevent="confirmRemove">
                          <icon type="delete" view-box="0 0 20 20" width="16" height="16"/>
                      </button>
                      <portal to="modals">
                          <delete-flexible-content-group-modal
                              v-if="removeMessage"
                              @confirm="remove"
                              @close="removeMessage=false"
                              :title="field.confirmRemoveTitle"
                              :message="field.confirmRemoveMessage"
                              :yes="field.confirmRemoveYes"
                              :no="field.confirmRemoveNo"
                          />
                      </portal>
                  </div>
              </div>
          </div>
          <div :style="(!isColumn && !collapsed) ? 'min-width: 400px' : ''" class="flex flex-col min-h-full w-full">
              <div :class="titleStyle" v-if="group.title">
                  <div class="leading-normal py-1 px-8">
                      <p class="text-80 flex flex-row items-center justify-center">
                          <span class="flex-1 font-bold">{{ group.title }}</span>
                          <span v-if="width && isColumn" class="ml-4 font-light text-70 text-sm">{{width}}%</span>
                          <span class="inline-block rounded-full px-2 ml-4 text-90 font-light text-sm" :style="'background-color: ' + color">
                            {{ index + 1 }}
                          </span>
                      </p>
                  </div>
              </div>
              <div class="overflow-y-auto" :class="containerStyle" style="min-height: 6.5rem">
                  <component
                      v-for="(item, index) in group.fields"
                      :key="index"
                      :is="'form-' + item.component"
                      :resource-name="resourceName"
                      :resource-id="resourceId"
                      :resource="resource"
                      :field="item"
                      :errors="errors"
                      :show-help-text="item.helpText != null"
                      :class="'full-width-fields ' + ((compact || index == group.fields.length - 1) ? 'remove-bottom-border ' : '') + (compact ? 'fields-no-pad ' : '')"
                  />
              </div>
          </div>
      </div>
</template>

<script>
import {BehavesAsPanel} from 'laravel-nova';

export default {
    mixins: [BehavesAsPanel],

    props: ['errors', 'group', 'index', 'field', 'isColumn', 'compact', 'width', 'color'],

    data() {
        return {
            removeMessage: false,
            collapsed: this.group.collapsed,
            readonly: this.group.readonly,
        };
    },

    computed: {
        titleStyle() {
            let classes = [ 'pl-2', 'border-60', 'rounded-tl-lg', 'rounded-tr-lg'];
            if(!this.isColumn) {
                classes.push('bg-white');
                classes.push('border-t');
                classes.push('border-r');
                classes.push('border-l');
                if (this.collapsed) {
                  classes.push('border-b');
                  classes.push('rounded-bl-lg');
                  classes.push('rounded-br-lg');
                }
            }
            return classes;
        },
        containerStyle() {
            let classes = ['pl-2', 'flex-grow', 'border-60', 'rounded-b-lg'];
            if(this.isColumn) {
              classes.push('pt-1');
              classes.push('pb-1');
            }
            if(!this.isColumn) {
                classes.push('bg-white');
                classes.push('border-b');
                classes.push('border-r');
                classes.push('border-l');
            }
            if (this.collapsed) {
                classes.push('hidden');
            }
            return classes;
        }
    },

    methods: {
        /**
         * Move this group up
         */
        moveUp() {
            this.$emit('move-up');
        },

        /**
         * Move this group down
         */
        moveDown() {
            this.$emit('move-down');
        },

        /**
         * Remove this group
         */
        remove() {
            this.$emit('remove');
        },

        /**
         * Add group
         */
        addGroup() {
            this.$emit('addGroup');
        },

        /**
         * Confirm remove message
         */
        confirmRemove() {
            if (this.field.confirmRemove) {
                this.removeMessage = true;
            } else {
                this.remove()
            }
        },

        /**
         * Expand fields
         */
        expand() {
            this.collapsed = false;
        },

        /**
         * Collapse fields
         */
        collapse() {
            this.collapsed = true;
        }
    },
}
</script>

<style>
.fields-no-pad div.py-6{
    padding-top: 0 !important;
    padding-bottom: 0.5rem !important;
}
.full-width-fields div.w-1\/2 {
    width: 100% !important;
}

.group-control:focus {
    outline: none;
}

.group-control path {
    fill: #B7CAD6;
    transition: fill 200ms ease-out;
}

.group-control:hover path {
    fill: var(--primary);
}

.confirm-message {
    position: absolute;
    overflow: visible;
    right: 38px;
    bottom: 0;
    width: auto;
    border-radius: 4px;
    padding: 6px 7px;
    border: 1px solid #B7CAD6;
    background-color: var(--20);
    white-space: nowrap;
}

[dir=rtl] .confirm-message {
    right: auto;
    left: 35px;
}

.confirm-message .text-danger {
    color: #ee3f22;
}

.closebtn {
    /*color: #B7CAD6;*/
}
</style>
