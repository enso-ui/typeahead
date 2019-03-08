<template>
    <typeahead :i18n="i18n"
        v-bind="$attrs"
        v-on="$listeners"/>
</template>

<script>
import Typeahead from './Typeahead.vue';

export default {
    name: 'EnsoTypeahead',

    inject: ['i18n'],

    components: { Typeahead },
};
</script>
