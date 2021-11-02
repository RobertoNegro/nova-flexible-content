<template>
    <div :class="{'-m-2 pb-2': isColumn}">
        <div :class="'overflow-x-auto relative flex mb-4 pb-2 ' + (isColumn ? 'border-b border-40' : '') "
             :id="group.key">
            <div :class="(isColumn ? 'border-40 border-b rounded-br-lg' : 'border-b border-t border-l border-60 rounded-l bg-white') + ' z-10 h-auto pin-l pin-t self-start w-8'">
                <button
                    dusk="expand-group"
                    type="button"
                    class="group-control btn border-r border-40 w-8 h-8 block"
                    :title="__('Expand')"
                    @click.prevent="expand"
                    v-if="collapsed">
                    <icon class="align-top" type="plus-square" width="16" height="16" view-box="0 0 24 24"/>
                </button>
                <div v-if="!collapsed">
                    <button
                        v-if="!isColumn"
                        dusk="collapse-group"
                        type="button"
                        class="group-control btn border-r border-40 w-8 h-8 block"
                        :title="__('Collapse')"
                        @click.prevent="collapse">
                        <icon class="align-top" type="minus-square" width="16" height="16" view-box="0 0 24 24"/>
                    </button>
                    <div v-if="!readonly">
                        <button
                            dusk="move-up-group"
                            type="button"
                            :class="(isColumn ? '' : 'border-t') +' group-control btn border-r border-40 w-8 h-8 block'"
                            :title="__('Move up')"
                            @click.prevent="moveUp">
                            <icon type="arrow-up" view-box="0 0 8 4.8" width="10" height="10"/>
                        </button>
                        <button
                            dusk="move-down-group"
                            type="button"
                            class="group-control btn border-t border-r border-40 w-8 h-8 block"
                            :title="__('Move down')"
                            @click.prevent="moveDown">
                            <icon type="arrow-down" view-box="0 0 8 4.8" width="10" height="10"/>
                        </button>
                        <button
                            v-if="!isColumn && !isDraft"
                            dusk="draft-group"
                            type="button"
                            class="group-control btn border-t border-r border-40 w-8 h-8 block"
                            :title="__('Draft')"
                            @click.prevent="draftGroup">
                          <icon type="edit" view-box="0 0 20 20" width="16" height="16"/>
                        </button>
                        <button
                            dusk="delete-group"
                            type="button"
                            :class="(isColumn ? 'rounded-br-lg' : '') + ' group-control btn border-t border-r border-40 w-8 h-8 block'"
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
            <div :style="(!isColumn && !collapsed) ? 'min-width: 400px; margin-right:1px' : ''" class="-mb-1 flex flex-col min-h-full w-full">
                <div :class="titleStyle" v-if="group.title">
                    <div class="leading-normal py-1 px-8"
                         :class="{'border-b border-40': !collapsed && !isColumn}">
                        <p class="text-80">
                            <span class="mr-4 font-semibold">#{{ index + 1 }}</span>
                            {{ group.title }}
                        </p>
                    </div>
                </div>
                <div :class="containerStyle">
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
    </div>
</template>

<script>
import {BehavesAsPanel} from 'laravel-nova';

export default {
    mixins: [BehavesAsPanel],

    props: ['errors', 'group', 'index', 'field', 'isColumn', 'isDraft', 'compact'],

    data() {
        return {
            removeMessage: false,
            collapsed: this.group.collapsed,
            readonly: this.group.readonly,
        };
    },

    computed: {
        titleStyle() {
            let classes = [ 'border-60', 'rounded-tr-lg'];
            if(!this.isColumn) {
                classes.push('bg-white');
                classes.push('border-t');
                classes.push('border-r');
            }
            if (this.collapsed) {
                if(!this.isColumn) {
                    classes.push('border-b');
                }
                classes.push('rounded-br-lg');
            }
            return classes;
        },
        containerStyle() {
            let classes = ['flex-grow', 'border-60', 'rounded-b-lg'];
            if(!this.isColumn) {
                classes.push('bg-white');
                classes.push('border-b');
                classes.push('border-r');
                classes.push('border-l');
            }
            if (!this.group.title) {
                if(!this.isColumn) {
                    classes.push('border-t');
                }
                classes.push('rounded-tr-lg');
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
         * Move to draft
         */
        draftGroup() {
            this.$emit('draft-group');
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
