<template>
    <typeahead :i18n="i18n"
        :source="source ? route(source) : null"
        v-bind="$attrs"
        v-on="$listeners"
        ref="typeahead">
        <template v-slot:controls="props">
            <slot name="controls"
                v-bind="props"/>
        </template>
        <template v-slot:option="props">
            <slot name="option"
                v-bind="props"/>
        </template>
    </typeahead>
</template>

<script>
import Typeahead from './Typeahead.vue';

export default {
    name: 'EnsoTypeahead',

    inject: ['i18n', 'route'],

    components: { Typeahead },

    props: {
        source: {
            type: String,
            default: null,
        },
    },

    methods: {
        clear() {
            this.$refs.typeahead.clear();
        },
        focus() {
            this.$refs.typeahead.focus();
        },
    },
};
</script>
