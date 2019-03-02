<template>
    <core-typeahead
        v-bind="$attrs"
        v-on="$listeners"
        ref="typeahead">
        <template v-slot:default="{
                query, label, items, loading, hasError, currentIndex, highlight,
                clearBindings, inputBindings, inputEvents, itemEvents,
            }">
            <dropdown class="typeahead"
                width="100%"
                :disabled="!query">
                <template v-slot:dropdown-trigger>
                    <div class="control has-icons-left has-icons-right"
                        :class="{ 'is-loading': loading }">
                        <input class="input is-fullwidth"
                            :class="[{ 'is-rounded': isRounded }, { 'is-danger': hasError }]"
                            type="text"
                            :disabled="disabled"
                            :placeholder="i18n(placeholder)"
                            v-bind="inputBindings"
                            v-on="inputEvents">
                        <span class="icon is-small is-left">
                            <fa icon="search"/>
                        </span>
                        <span class="icon is-small is-right clear-button"
                            v-on="clearBindings"
                            v-if="query && !loading">
                            <a class="delete is-small"/>
                        </span>
                    </div>
                    <slot name="controls"
                        :items="items"/>
                </template>
                <template v-slot:dropdown-content>
                    <a class="dropdown-item"
                        v-for="(item, index) in items"
                        :key="index"
                        :class="{ 'is-active': index === currentIndex }"
                        v-on="itemEvents(index)">
                        <slot name="option"
                            :highlight="highlight"
                            :item="item"
                            :label="label">
                            <span v-html="highlight(item[label])"/>
                        </slot>
                    </a>
                    <template v-if="!items.length">
                        <a class="dropdown-item"
                            v-if="loading">
                            <span v-if="loading">
                                {{ i18n(searching) }}
                            </span>
                        </a>
                        <a class="dropdown-item"
                            v-else-if="query">
                            <span>
                                {{ i18n(noResults) }}
                            </span>
                        </a>
                    </template>
                </template>
            </dropdown>
        </template>
    </core-typeahead>
</template>

<script>
import { library } from '@fortawesome/fontawesome-svg-core';
import { faSearch } from '@fortawesome/free-solid-svg-icons';
import Dropdown from '@enso-ui/dropdown/bulma';
import DropdownIndicator from '@enso-ui/dropdown-indicator';
import CoreTypeahead from '../renderless/Typeahead.vue';

library.add(faSearch);
export default {
    name: 'Typeahead',

    components: { CoreTypeahead, Dropdown, DropdownIndicator },

    model: {
        event: 'selected',
    },
    props: {
        disabled: {
            type: Boolean,
            default: false,
        },
        i18n: {
            type: Function,
            default: v => v,
        },
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

<style lang="scss" scoped>
    .wrapper {
        width: 100%;
        .dropdown.typeahead {
            width: calc(100% - 1.4em);
            position: absolute;
            .dropdown-menu {
                width: 100%;
                .dropdown-content {
                    max-height: 20em;
                    overflow-y: scroll;
                    a.dropdown-item {
                        text-overflow: ellipsis;
                        overflow-x: hidden;
                        padding-right: 1em;
                    }
                }
            }
        }
        .control.has-icons-right .icon.clear-button {
            pointer-events: all;
        }
    }
</style>
