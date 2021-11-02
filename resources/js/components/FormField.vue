<template>
    <component
        :dusk="field.attribute"
        :is="field.fullWidth ? 'full-width-field' : 'default-field'"
        :field="field"
        :errors="errors"
        full-width-content>
        <template slot="field">
            <div
                class="flex flex-row flex-wrap items-start justify-center bg-40 mb-2 p-2 rounded-lg"
                v-if="drafts.length > 0">
                <div class="w-full">
                    <div v-if="columns.length > 0" class="w-full flex flex-row justify-between items-center">
                        <div class="text-90 text-lg font-bold">{{ __('Bozze') }}:</div>
                        <div class="text-80 text-sm font-light">
                            {{ __('Sposta i componenti dentro ad una colonna per inserirli nella pagina') }}
                        </div>
                    </div>
                    <div class="w-full p-1" v-for="(draft, index) in drafts">
                        <div :class="columns.length > 0 ? 'border-4 border-dotted' : 'border'"
                             class="w-full bg-white shadow rounded-lg border-50 p-2">
                            <form-nova-flexible-content-group
                                :dusk="field.attribute + '-' + draft.index"
                                :key="draft.group.key"
                                :field="{...field, confirmRemove: true, confirmRemoveNo: 'Annulla', confirmRemoveYes: 'Elimina', confirmRemoveTitle: 'Elimina colonna', confirmRemoveMessage: 'Proseguendo verrà eliminata la colonna e il suo intero contenuto. L\'operazione è irreversibile. Sei sicuro di voler procedere?'}"
                                :group="{...draft.group, collapsed: false}"
                                :index="draft.index"
                                :resource-name="resourceName"
                                :resource-id="resourceId"
                                :resource="resource"
                                :errors="errors"
                                :is-column="true"
                                :is-draft="true"
                                :compact="false"
                                :width="draft.width"
                                :color="draft.color"
                                @move-up="moveUp(draft.group.key)"
                                @move-down="moveDown(draft.group.key)"
                                @draft-group="draftGroup(draft.group.key)"
                                @remove="remove(draft.group.key)"
                            />
                        </div>
                    </div>
                </div>
            </div>

            <div
                :class="'flex flex-row flex-wrap items-start justify-center p-2 mb-2 rounded-lg ' + (rowIndex % 2 === 0 ? 'bg-60' : 'bg-40')"
                v-for="(row, rowIndex) in rows"
                v-if="rows.length > 0">
                <div :class="mapWidthToCssClass(column.width) + ' full-on-mobile p-1'" v-for="(column, columnIndex) in row.columns">
                    <div class="w-full bg-white shadow rounded-lg border border-50 p-2">
                        <form-nova-flexible-content-group
                            :dusk="field.attribute + '-' + column.index"
                            :key="column.group.key"
                            :field="{...field, confirmRemove: true, confirmRemoveNo: 'Annulla', confirmRemoveYes: 'Elimina', confirmRemoveTitle: 'Elimina colonna', confirmRemoveMessage: 'Proseguendo verrà eliminata la colonna e il suo intero contenuto. L\'operazione è irreversibile. Sei sicuro di voler procedere?'}"
                            :group="{...column.group}"
                            :index="column.index"
                            :resource-name="resourceName"
                            :resource-id="resourceId"
                            :resource="resource"
                            :errors="errors"
                            :is-column="true"
                            :is-draft="false"
                            :compact="true"
                            :width="column.width"
                            :color="column.color"
                            @move-up="moveUp(column.group.key, row.offset + columnIndex)"
                            @move-down="moveDown(column.group.key, row.offset + columnIndex)"
                            @draft-group="draftGroup(column.group.key)"
                            @remove="remove(column.group.key, row.offset + columnIndex)"
                        />
                        <form-nova-flexible-content-group
                            v-for="(subgroup, index) in column.subgroups"
                            :dusk="field.attribute + '-' + (column.index + index + 1)"
                            :key="subgroup.group.key"
                            :field="{...field, confirmRemove: true, confirmRemoveNo: 'Annulla', confirmRemoveYes: 'Elimina', confirmRemoveTitle: 'Elimina elemento', confirmRemoveMessage: 'Proseguendo l\'elemento selezionato verrà eliminato. L\'operazione è irreversibile. Sei sicuro di voler procedere?'}"
                            :group="subgroup.group"
                            :index="column.index + index + 1"
                            :resource-name="resourceName"
                            :resource-id="resourceId"
                            :resource="resource"
                            :errors="errors"
                            :is-column="false"
                            :width="column.width"
                            :color="subgroup.color"
                            @move-up="moveUp(subgroup.group.key)"
                            @move-down="moveDown(subgroup.group.key)"
                            @draft-group="draftGroup(subgroup.group.key)"
                            @remove="remove(subgroup.group.key)"
                        />
                        <div v-if="column.allowChildrens">
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
import {eventBus} from '../eventbus';
import randomColor from 'randomcolor';

const colors = [];

export default {
    mixins: [FormField, HandlesValidationErrors],

    props: ['resourceName', 'resourceId', 'resource', 'field'],

    components: {FullWidthField},

    computed: {
        layouts() {
            return this.field.layouts || false
        },
        orderedGroups() {
            return this.order.reduce((groups, key) => {
                groups.push(this.groups[key]);
                return groups;
            }, []);
        },
        drafts() {
            const res = [];
            this.orderedGroups.every((g, i) => {
                if (this.isColumn(g.name)) {
                    return false;
                } else {
                    res.push({
                        group: g,
                        index: i,
                        color: this.getColor(g),
                        width: '100'
                    });
                    return true;
                }
            });

            let hasToCheck = false;
            this.orderedGroups.forEach((g, i) => {
                if (this.isSpecialChildless(g.name)) {
                    hasToCheck = true;
                } else {
                    if (hasToCheck) {
                        if (!this.isColumn(g.name)) {
                            res.push({
                                group: g,
                                index: i,
                                color: this.getColor(g),
                                width: '100'
                            });
                        } else {
                            hasToCheck = false;
                        }
                    }
                }
            });
            return res;
        },
        columns() {
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
                color: this.getColor(g),
                subgroups: [],
                index: i,
                allowChildrens: true
              };
              if (this.isSpecialChildless(g.name)) {
                res.push({...col, allowChildrens: false});
                col = null;
              }
            } else {
              if (col) {
                col.subgroups.push({
                  group: g,
                  color: this.getColor(g),
                });
              }
            }
          })
          if (col) {
            res.push(col);
          }
          return res;
        },

        rows() {
          const res = [];
          let row = {
            offset: 0,
            columns: []
          };
          let width = 0;
          this.columns.forEach((column, i) => {
            if(width + parseInt(column.width) > 100) {
              if(row.columns.length > 0) {
                res.push(row);
              }
              row = {
                offset: i,
                columns: [column]
              };
              width = parseInt(column.width) ;
            } else {
              row.columns.push(column);
              width += parseInt(column.width) ;
            }
          });
          if(row.columns.length > 0) {
            res.push(row);
          }
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
            return ['x_column'].includes(name) || this.isSpecialChildless(name);
        },

        isSpecialChildless(name) {
            return ['x_banner'].includes(name);
        },

        getColor(group) {
          if(!colors[group.key]) {
            colors[group.key] = randomColor({
              luminosity: 'light',
            })
          }
          return colors[group.key];
        },

        getColumnWidth(group) {
            if (!group || !this.isColumn(group.name)) {
                return false;
            }
            if (group.name === 'x_column') {
                const widthField = group.fields.find(f => f.sortableUriKey === 'x_width');
                if (!widthField) {
                    return false;
                }
                return widthField.value;
            } else {
                return '100';
            }
        },

        mapWidthToCssClass(width) {
              switch (width) {
                case '25': return 'w-1/4';
                case '33': return 'w-1/3';
                case '50': return 'w-1/2';
                case '66': return 'w-2/3';
                case '75': return 'w-3/4';
                default: return 'w-full';
              }
        },

        /**
         * Move a group up
         */
        moveUp(key, columnIndex) {
            let index = this.order.indexOf(key);

            if (index <= 0 || (columnIndex !== undefined && columnIndex <= 0)) return;

            const amount = columnIndex !== undefined ? this.columns[columnIndex].subgroups.length + 1 : 1;
            let shift = columnIndex !== undefined ? this.columns[columnIndex - 1].subgroups.length + 1 : 1;
            if(columnIndex === undefined) {
                let found = false;
                for(let i = index-2; i >= 0; i--) {
                    if(this.isSpecialChildless(this.groups[this.order[i]].name)) {
                        shift += 1;
                    } else {
                        found = true;
                        break;
                    }
                }
                if(!found) {
                    return;
                }
            }
            this.order.splice(index - shift, 0, ...this.order.splice(index, amount));
        },

        /**
         * Move a group down
         */
        moveDown(key, columnIndex) {
            let index = this.order.indexOf(key);

            if (index < 0 || index >= this.order.length - 1 || columnIndex >= this.columns.length - 1) return;

            const amount = columnIndex !== undefined ? this.columns[columnIndex].subgroups.length + 1 : 1;
            let shift = columnIndex !== undefined ? this.columns[columnIndex + 1].subgroups.length + 1 : 1;
            if(columnIndex === undefined) {
                let found = false;
                for(let i = index + 1; i < this.order.length; i++) {
                    if(this.isSpecialChildless(this.groups[this.order[i]].name)) {
                        shift += 1;
                    } else {
                        found = true;
                        break;
                    }
                }
                if(!found) {
                    return;
                }
            }
            this.order.splice(index + shift, 0, ...this.order.splice(index, amount));
        },

        /**
         * Remove a group
         */
        remove(key, columnIndex) {
            let index = this.order.indexOf(key);

            if (index < 0) return;

            const amount = columnIndex !== undefined ? this.columns[columnIndex].subgroups.length + 1 : 1;

            const keys = this.order.splice(index, amount);
            keys.forEach(k => {
                delete this.groups[k];
            })

            if (this.limitCounter >= 0) {
                this.limitCounter += amount;
            }
        },

        /**
         * Move group to drafts
         */
        draftGroup(key) {
          let index = this.order.indexOf(key);
          this.order.splice(0, 0, ...this.order.splice(index, 1));
        },
    },
    mounted() {
        eventBus.$on('add-group-' + this.field.attribute, (layout, position) => {
            this.addGroup(layout, position);
        });
    }
}
</script>

<style>
@media only screen and (max-width: 1400px) {
    .full-on-mobile {
        width: 100% !important;
    }
}
</style>
