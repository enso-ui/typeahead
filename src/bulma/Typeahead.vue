<template>
    <core-typeahead
        v-bind="$attrs"
        v-on="$listeners"
        ref="typeahead">
        <template v-slot:default="{
                query, label, items, loading, hasError, visibleDropdown, currentIndex,
                clear, updateIndex, highlight, inputBindings, inputEvents, itemEvents,
            }">
            <div class="wrapper">
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
                        v-if="query && !loading"
                        @click="clear">
                        <a class="delete is-small"/>
                    </span>
                </div>
                <slot name="controls"
                    :items="items"/>
                <fade>
                    <div class="dropdown typeahead is-active"
                        v-if="visibleDropdown">
                        <div class="dropdown-menu">
                            <div class="dropdown-content">
                                <a href="#" class="dropdown-item"
                                    v-for="(item, index) in items"
                                    :key="JSON.stringify(item)"
                                    :class="{ 'is-active': index === currentIndex }"
                                    v-on="itemEvents"
                                    @mouseover="updateIndex(index)">
                                    <slot name="option"
                                        :highlight="highlight"
                                        :item="item"
                                        :label="label">
                                        <span v-html="highlight(item[label])"/>
                                    </slot>
                                </a>
                                <a href="#"
                                    class="dropdown-item"
                                    v-if="!items.length">
                                    <span v-if="loading">
                                        {{ i18n(searching) }}
                                    </span>
                                    <span v-else>
                                        {{ i18n(noResults) }}
                                    </span>
                                </a>
                            </div>
                        </div>
                    </div>
                </fade>
            </div>
        </template>
    </core-typeahead>
</template>
<script>
import { library } from '@fortawesome/fontawesome-svg-core';
import { faSearch } from '@fortawesome/free-solid-svg-icons';
import { Fade } from '@enso-ui/transitions';
import CoreTypeahead from '../renderless/Typeahead.vue';

library.add(faSearch);
export default {
    name: 'Typeahead',

    components: { CoreTypeahead, Fade },

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
