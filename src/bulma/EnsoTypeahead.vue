<template>
    <typeahead v-bind="$attrs"
        :i18n="i18n"
        :source="source ? route(source) : null"
        :error-handler="errorHandler"
        v-on="$listeners"
        ref="typeahead">
        <template #:controls="props">
            <slot name="controls"
                v-bind="props"/>
        </template>
        <template #:option="props">
            <slot name="option"
                v-bind="props"/>
        </template>
    </typeahead>
</template>

<script>
import Typeahead from './Typeahead.vue';

export default {
    name: 'EnsoTypeahead',

    components: { Typeahead },

    inject: ['errorHandler', 'i18n', 'route'],

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
