<template>
    <field-wrapper>
        <div class="pt-2 pb-2 px-2 w-full">
            <div class="ml-6 mb-4" v-if="fieldLabel">
                <form-label :for="field.attribute" :class="{
                      'mb-2': field.helpText && showHelpText
                  }">
                    {{ fieldLabel }}

                    <span v-if="field.required" class="text-danger text-sm">{{
                        __('*')
                    }}</span>
                </form-label>

                <help-text v-if="showHelpText">
                    {{ field.helpText }}
                </help-text>
            </div>

            <slot name="field"/>

            <help-text
                class="error-text mt-2 text-danger"
                v-if="showErrors && hasError"
            >
                {{ firstError }}
            </help-text>
        </div>
    </field-wrapper>
</template>

<script>
import { mapProps } from 'laravel-nova'
import { HandlesValidationErrors, Errors } from 'laravel-nova'

export default {
    mixins: [HandlesValidationErrors],

    props: {
        field: { type: Object, required: true },
        fieldName: { type: String },
        showErrors: { type: Boolean, default: true },
        ...mapProps(['showHelpText']),
    },

    computed: {
        fieldLabel() {
            // If the field name is purposefully empty, hide the label altogether
            if (this.fieldName === '') {
                return false;
            }

            return this.fieldName || this.field.singularLabel || this.field.name
        },
    },
};
</script>
