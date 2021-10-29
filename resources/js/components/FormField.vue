<template>
    <component
        :dusk="field.attribute"
        :is="field.fullWidth ? 'full-width-field' : 'default-field'"
        :field="field"
        :errors="errors"
        full-width-content>
        <template slot="field">
            <!--                style="display: grid;grid-template-columns: repeat(12, minmax(0, 1fr));gap: 0.25rem;"-->
            <div
                class="flex flex-row flex-wrap items-start justify-center bg-40 p-2 mb-2 rounded-lg"
                v-if="order.length > 0">
                <div :class="column.width + ' p-1'" v-for="column in columns">
                    <div class="w-full bg-white shadow rounded-lg p-2">
                        <form-nova-flexible-content-group
                            :dusk="field.attribute + '-' + column.index"
                            :key="column.group.key"
                            :field="{...field, confirmRemove: true, confirmRemoveNo: 'Annulla', confirmRemoveYes: 'Elimina', confirmRemoveTitle: 'Elimina colonna', confirmRemoveMessage: 'Proseguendo verrà eliminata la colonna e il suo intero contenuto. L\'operazione è irreversibile. Sei sicuro di voler procedere?'}"
                            :group="column.group"
                            :index="column.index"
                            :resource-name="resourceName"
                            :resource-id="resourceId"
                            :resource="resource"
                            :errors="errors"
                            :layouts="layouts"
                            @move-up="moveUp(column.group.key, column.subgroups.length)"
                            @move-down="moveDown(column.group.key, column.subgroups.length)"
                            @remove="remove(column.group.key, column.subgroups.length)"
                        />
                        <form-nova-flexible-content-group
                            v-for="(group, index) in column.subgroups"
                            :dusk="field.attribute + '-' + (column.index + index + 1)"
                            :key="group.key"
                            :field="{...field, confirmRemove: true, confirmRemoveNo: 'Annulla', confirmRemoveYes: 'Elimina', confirmRemoveTitle: 'Elimina elemento', confirmRemoveMessage: 'Proseguendo l\'elemento selezionato verrà eliminato. L\'operazione è irreversibile. Sei sicuro di voler procedere?'}"
                            :group="group"
                            :index="column.index + index + 1"
                            :resource-name="resourceName"
                            :resource-id="resourceId"
                            :resource="resource"
                            :errors="errors"
                            @move-up="moveUp(group.key)"
                            @move-down="moveDown(group.key)"
                            @remove="remove(group.key)"
                        />
                        <component
                            :layouts="layouts"
                            :is="field.menu.component"
                            :field="field"
                            :limit-counter="limitCounter"
                            :errors="errors"
                            :resource-name="resourceName"
                            :resource-id="resourceId"
                            :resource="resource"
                            :index="column.index + column.subgroups.length + 1"
                        />
                    </div>

                </div>

            </div>

            <component
                :layouts="layouts"
                :is="field.menu.component"
                :field="field"
                :limit-counter="limitCounter"
                :errors="errors"
                :resource-name="resourceName"
                :resource-id="resourceId"
                :resource="resource"
                :index="order.length"
            />

        </template>
    </component>
</template>

<script>

import FullWidthField from './FullWidthField';
import {FormField, HandlesValidationErrors} from 'laravel-nova';
import Group from '../group';
import { eventBus } from '../eventbus';

export default {
    mixins: [FormField, HandlesValidationErrors],

    props: ['resourceName', 'resourceId', 'resource', 'field'],

    components: {FullWidthField},

    computed: {
        layouts() {
            return this.field.layouts || false
        },
        orderedGroups() {
            console.log(this.order);
            return this.order.reduce((groups, key) => {
                groups.push(this.groups[key]);
                return groups;
            }, []);
        },
        groupsWithColumns() {
            let width = 'grid-column: span 12 / span 12;';
            return this.orderedGroups.map((g, k) => {
                const newWidth = this.getColumnWidth(g);
                width = newWidth ? newWidth : width;
                return {...g, width};
            });
        },
        columns() {
            let width = 'grid-column: span 12 / span 12;';
            const res = [];
            let col = null;
            this.orderedGroups.forEach((g, i) => {
                if (this.isColumn(g.name)) {
                    if (col) {
                        res.push(col);
                    }
                    col = {
                        group: g,
                        width: this.getColumnWidth(g),
                        subgroups: [],
                        index: i,
                    };
                } else {
                    if (col) {
                        col.subgroups.push(g);
                    }
                }
            })
            if (col) {
                res.push(col);
            }
            console.log(res);
            return res;
        }
    },

    data() {
        return {
            order: [],
            groups: {},
            files: {},
            limitCounter: this.field.limit
        };
    },

    methods: {
        /*
         * Set the initial, internal value for the field.
         */
        setInitialValue() {
            this.value = this.field.value || [];
            this.files = {};

            this.populateGroups();
        },

        /**
         * Fill the given FormData object with the field's internal value.
         */
        fill(formData) {
            let key, group;

            this.value = [];
            this.files = {};

            for (var i = 0; i < this.order.length; i++) {
                key = this.order[i];
                group = this.groups[key].serialize();

                // Only serialize the group's non-file attributes
                this.value.push({
                    layout: group.layout,
                    key: group.key,
                    attributes: group.attributes
                });

                // Attach the files for formData appending
                this.files = {...this.files, ...group.files};
            }

            this.appendFieldAttribute(formData, this.field.attribute);
            formData.append(this.field.attribute, this.value.length ? JSON.stringify(this.value) : '');

            // Append file uploads
            for (let file in this.files) {
                formData.append(file, this.files[file]);
            }
        },

        /**
         * Register given field attribute into the parsable flexible fields register
         */
        appendFieldAttribute(formData, attribute) {
            let registered = [];

            if (formData.has('___nova_flexible_content_fields')) {
                registered = JSON.parse(formData.get('___nova_flexible_content_fields'));
            }

            registered.push(attribute);

            formData.set('___nova_flexible_content_fields', JSON.stringify(registered));
        },

        /**
         * Update the field's internal value.
         */
        handleChange(value) {
            this.value = value || [];
            this.files = {};

            this.populateGroups();
        },

        /**
         * Set the displayed layouts from the field's current value
         */
        populateGroups() {
            this.order.splice(0, this.order.length);
            this.groups = {};

            for (var i = 0; i < this.value.length; i++) {
                this.addGroup(
                    this.getLayout(this.value[i].layout),
                    i,
                    this.value[i].attributes,
                    this.value[i].key,
                    this.field.collapsed
                );
            }
        },

        /**
         * Retrieve layout definition from its name
         */
        getLayout(name) {
            if (!this.layouts) return;
            return this.layouts.find(layout => layout.name == name);
        },

        /**
         * Append the given layout to flexible content's list
         */
        addGroup(layout, position, attributes, key, collapsed) {
            if (!layout) return;

            collapsed = collapsed || false;

            let fields = attributes || JSON.parse(JSON.stringify(layout.fields)),
                group = new Group(layout.name, layout.title, fields, this.field, key, collapsed);

            this.groups[group.key] = group;
            this.order.splice(position, 0, group.key);

            if (this.limitCounter > 0) {
                this.limitCounter--;
            }
        },

        isColumn(name) {
            return name === 'x_column';
        },

        getColumnWidth(group) {
            if (!this.isColumn(group.name)) {
                return false;
            }
            const widthField = group.fields.find(f => f.sortableUriKey === 'x_width');
            if (!widthField) {
                return false;
            }
            return widthField.value;
        },

        /**
         * Move a group up
         */
        moveUp(key, amount) {
            let index = this.order.indexOf(key);

            if (index <= 0) return;

            this.order.splice(index - 1, 0, ...this.order.splice(index, 1 + (amount != null ? amount : 0)));
        },

        /**
         * Move a group down
         */
        moveDown(key, amount) {
            let index = this.order.indexOf(key);

            if (index < 0 || index >= this.order.length - 1) return;

            this.order.splice(index + 1, 0, ...this.order.splice(index, 1 + (amount != null ? amount : 0)));
        },

        /**
         * Remove a group
         */
        remove(key, amount) {
            let index = this.order.indexOf(key);

            if (index < 0) return;

            this.order.splice(index, 1);
            delete this.groups[key];

            if (this.limitCounter >= 0) {
                this.limitCounter++;
            }

            if (amount) {
                for (let i = 0; i < amount; i++) {
                    this.order.splice(index, 1);
                    delete this.groups[key];

                    if (this.limitCounter >= 0) {
                        this.limitCounter++;
                    }
                }
            }
        }
    },
    mounted() {
        eventBus.$on('add-group-'+this.field.attribute, (layout, position) => {
            this.addGroup(layout, position);
        });
    }
}
</script>
