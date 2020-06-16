<template>
    <core-typeahead
        v-bind="$attrs"
        v-on="$listeners"
        ref="typeahead">
        <template v-slot:default="{
                clearBindings, disabled, hasError, highlight, i18n, inputBindings,
                inputEvents, itemEvents, items, label, loading, minQueryLength,
                modeSelector, modeBindings, modeEvents, query,
            }">
            <dropdown class="typeahead"
                :disabled="!query"
                :disable-controls="items.length === 0"
                manual
                v-on="$listeners">
                <template v-slot:trigger="{ show, hide }">
                    <div class="control has-icons-left has-icons-right"
                        :class="{ 'is-loading': loading }">
                        <input class="input is-fullwidth"
                            :class="[{ 'is-rounded': isRounded }, { 'is-danger': hasError }]"
                            type="text"
                            :disabled="disabled"
                            :placeholder="i18n(placeholder)"
                            v-bind="inputBindings"
                            @keyup="query.length >= minQueryLength ? show() : hide()"
                            v-on="inputEvents">
                        <span class="icon is-small is-left">
                            <fa icon="search"/>
                        </span>
                        <search-mode class="is-right is-small search-mode"
                            v-bind="modeBindings"
                            v-on="modeEvents"
                            v-if="modeSelector"/>
                        <span class="icon is-small is-right clear-button"
                            v-on="clearBindings"
                            v-if="query && !loading">
                            <a class="delete is-small"/>
                        </span>
                    </div>
                </template>
                <template v-slot:controls>
                    <slot name="controls"
                        :items="items"/>
                </template>
                <template v-slot:items
                    :item-events="itemEvents"
                    :highlight="highlight">
                    <dropdown-item v-for="(item, index) in items"
                        :key="index"
                        :selected="false"
                        v-on="itemEvents(index)">
                        <slot name="option"
                            :highlight="highlight"
                            :item="item"
                            :label="label"
                            :query="query">
                            <!-- eslint-disable-next-line vue/no-v-html -->
                            <span v-html="highlight(item[label])"/>
                        </slot>
                    </dropdown-item>
                    <template v-if="items.length === 0">
                        <dropdown-item v-if="loading">
                            {{ i18n(searching) }}
                        </dropdown-item>
                        <dropdown-item v-else-if="query">
                            {{ i18n(noResults) }}
                        </dropdown-item>
                    </template>
                </template>
            </dropdown>
        </template>
    </core-typeahead>
</template>

<script>
import { library } from '@fortawesome/fontawesome-svg-core';
import { faSearch } from '@fortawesome/free-solid-svg-icons';
import { Dropdown, DropdownItem } from '@enso-ui/dropdown/bulma';
import SearchMode from '@enso-ui/search-mode/bulma';
import CoreTypeahead from '../renderless/CoreTypeahead.vue';

library.add(faSearch);
export default {
    name: 'Typeahead',

    components: {
        CoreTypeahead, SearchMode, Dropdown, DropdownItem,
    },

    props: {
        isRounded: {
            type: Boolean,
            default: false,
        },
        noResults: {
            type: String,
            default: 'Nothing found...',
        },
        placeholder: {
            type: String,
            default: 'What are you searching for today?',
        },
        searching: {
            type: String,
            default: 'Searching...',
        },
    },

    methods: {
        clear() {
            this.$refs.typeahead.clear();
        },
    },
};
</script>

<style lang="scss">
    .dropdown.typeahead {
        width: 100%;

        .dropdown-trigger {
            width: 100%;

            .search-mode {
                pointer-events: all;
                right: 1.6em;
            }
        }

        .dropdown-menu {
            width: 100%;

            .dropdown-content {
                width: 100%;

                .options {
                    max-height: 20.5em;
                    width: 100%;
                    overflow: auto;

                    a.dropdown-item {
                        width: 100%;
                        text-overflow: ellipsis;
                        overflow-x: hidden;
                        padding-right: 1em;
                    }
                }

            }
        }
    }

    .control.has-icons-right .icon.clear-button {
        pointer-events: all;
    }
</style>
